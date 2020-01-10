cmake_minimum_required(VERSION 3.1.0)
project(openssl LANGUAGES C)

option(BUILD_SHARED_OPENSSL "Build shared openssl lib" ON)

include(GNUInstallDirs)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_BINDIR})

# OpenSSL version detection imported from FindOpenSSL.cmake
file(STRINGS "${PROJECT_SOURCE_DIR}/include/openssl/opensslv.h" openssl_version_str
		REGEX "^#[\t ]*define[\t ]+OPENSSL_VERSION_NUMBER[\t ]+0x([0-9a-fA-F])+.*"
		)

function(from_hex HEX DEC)
	string(TOUPPER "${HEX}" HEX)
	set(_res 0)
	string(LENGTH "${HEX}" _strlen)

	while(_strlen GREATER 0)
		math(EXPR _res "${_res} * 16")
		string(SUBSTRING "${HEX}" 0 1 NIBBLE)
		string(SUBSTRING "${HEX}" 1 -1 HEX)
		if(NIBBLE STREQUAL "A")
			math(EXPR _res "${_res} + 10")
		elseif(NIBBLE STREQUAL "B")
			math(EXPR _res "${_res} + 11")
		elseif(NIBBLE STREQUAL "C")
			math(EXPR _res "${_res} + 12")
		elseif(NIBBLE STREQUAL "D")
			math(EXPR _res "${_res} + 13")
		elseif(NIBBLE STREQUAL "E")
			math(EXPR _res "${_res} + 14")
		elseif(NIBBLE STREQUAL "F")
			math(EXPR _res "${_res} + 15")
		else()
			math(EXPR _res "${_res} + ${NIBBLE}")
		endif()

		string(LENGTH "${HEX}" _strlen)
	endwhile()

	set(${DEC} ${_res} PARENT_SCOPE)
endfunction()

string(REGEX REPLACE "^.*OPENSSL_VERSION_NUMBER[\t ]+0x([0-9a-fA-F])([0-9a-fA-F][0-9a-fA-F])([0-9a-fA-F][0-9a-fA-F])([0-9a-fA-F][0-9a-fA-F])([0-9a-fA-F]).*$"
		"\\1;\\2;\\3;\\4;\\5" OPENSSL_VERSION_LIST "${openssl_version_str}"
		)
list(GET OPENSSL_VERSION_LIST 0 OPENSSL_VERSION_MAJOR)
list(GET OPENSSL_VERSION_LIST 1 OPENSSL_VERSION_MINOR)
from_hex("${OPENSSL_VERSION_MINOR}" OPENSSL_VERSION_MINOR)
list(GET OPENSSL_VERSION_LIST 2 OPENSSL_VERSION_FIX)
from_hex("${OPENSSL_VERSION_FIX}" OPENSSL_VERSION_FIX)
list(GET OPENSSL_VERSION_LIST 3 OPENSSL_VERSION_PATCH)

if(NOT OPENSSL_VERSION_PATCH STREQUAL "00")
	from_hex("${OPENSSL_VERSION_PATCH}" _tmp)
	# 96 is the ASCII code of 'a' minus 1
	math(EXPR OPENSSL_VERSION_PATCH_ASCII "${_tmp} + 96")
	unset(_tmp)
	string(ASCII "${OPENSSL_VERSION_PATCH_ASCII}" OPENSSL_VERSION_PATCH_STRING)
endif()

set(OPENSSL_VERSION "${OPENSSL_VERSION_MAJOR}.${OPENSSL_VERSION_MINOR}.${OPENSSL_VERSION_FIX}${OPENSSL_VERSION_PATCH_STRING}")

set(VERSION_MAJOR ${OPENSSL_VERSION_MAJOR})
set(VERSION_MINOR ${OPENSSL_VERSION_MINOR})
set(VERSION_PATCH ${OPENSSL_VERSION_FIX})

set(VERSION_STRING ${OPENSSL_VERSION})
set(LIB_VERSION ${VERSION_MAJOR}.${VERSION_MINOR})
set(LIB_SOVERSION ${VERSION_MAJOR}.${VERSION_MINOR})

include(CheckTypeSize)
check_type_size("long" LONG_INT)
check_type_size("long long" LONG_LONG_INT)
check_type_size("int" INT)
if(HAVE_LONG_INT AND (${LONG_INT} EQUAL 8))
	set(SIXTY_FOUR_BIT_LONG ON)
elseif(HAVE_LONG_LONG_INT AND (${LONG_LONG_INT} EQUAL 8))
	set(SIXTY_FOUR_BIT ON)
else()
	set(THIRTY_TWO_BIT ON)
endif()

if(MSVC)
	set(OPENSSL_EXPORT_VAR_AS_FUNCTION 1)
endif()

# Begin configure public headers
file(COPY ${PROJECT_SOURCE_DIR}/include/internal DESTINATION include)
file(COPY ${PROJECT_SOURCE_DIR}/include/openssl DESTINATION include)

file(READ ${PROJECT_SOURCE_DIR}/opensslconf.h.cmake CONF)
set(CONF "
#define OPENSSL_NO_MD2
#define OPENSSL_NO_RC5
#define OPENSSL_NO_RFC3779
#define OPENSSL_NO_EC_NISTP_64_GCC_128
${CONF}")
file(WRITE ${PROJECT_BINARY_DIR}/opensslconf.h.cmake "${CONF}")

configure_file(${PROJECT_BINARY_DIR}/opensslconf.h.cmake
		${PROJECT_BINARY_DIR}/include/openssl/opensslconf.h
		)
# End configure public headers

add_subdirectory(crypto)
add_subdirectory(ssl)

file(READ ${PROJECT_SOURCE_DIR}/c_rehash.cmake C_REHASH)
string(REPLACE "@OPENSSLDIR@" "${OPENSSLDIR}" C_REHASH "${C_REHASH}")
string(REPLACE "@CMAKE_INSTALL_PREFIX@" "${CMAKE_INSTALL_PREFIX}" C_REHASH "${C_REHASH}")
file(WRITE ${PROJECT_BINARY_DIR}/c_rehash "${C_REHASH}")

file(GLOB PUBLIC_HEADERS "${PROJECT_BINARY_DIR}/include/openssl/*.h")
install(FILES ${PUBLIC_HEADERS} DESTINATION include/openssl)

install(FILES
		${PROJECT_BINARY_DIR}/c_rehash
		FAQ LICENSE README README.ENGINE
		DESTINATION share/openssl
		)

# Generate the package target
set(CPACK_GENERATOR ZIP TGZ)
set(CPACK_PACKAGE_NAME "${CMAKE_PROJECT_NAME}")
set(CPACK_PACKAGE_VERSION_MAJOR ${VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${VERSION_PATCH})
set(CPACK_PACKAGE_FILE_NAME "${CMAKE_PROJECT_NAME}-${VERSION_STRING}")

include(CPack)
