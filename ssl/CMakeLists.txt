if(BUILD_SHARED_OPENSSL)
	add_definitions(-DOPENSSL_BUILD_SHLIBSSL)
endif()

set(LIBSRC
		bio_ssl.c d1_lib.c d1_msg.c d1_srtp.c methods.c packet.c pqueue.c s3_cbc.c s3_enc.c s3_lib.c
		s3_msg.c ssl_asn1.c ssl_cert.c ssl_ciph.c ssl_conf.c ssl_err.c ssl_init.c ssl_lib.c
		ssl_mcnf.c ssl_rsa.c ssl_sess.c ssl_stat.c ssl_txt.c ssl_utst.c t1_enc.c t1_lib.c
		t1_trce.c tls13_enc.c tls_srp.c
		record/dtls1_bitmap.c record/rec_layer_d1.c record/rec_layer_s3.c record/ssl3_buffer.c
		record/ssl3_record.c record/ssl3_record_tls13.c
		statem/extensions.c statem/extensions_clnt.c statem/extensions_cust.c
		statem/extensions_srvr.c statem/statem.c statem/statem_clnt.c statem/statem_dtls.c
		statem/statem_lib.c statem/statem_srvr.c
		)

include_directories(BEFORE SYSTEM
	${PROJECT_SOURCE_DIR}/ # e_os.h
)

if(BUILD_SHARED_OPENSSL)
	add_library(ssl SHARED ${LIBSRC})
	set_target_properties(ssl PROPERTIES
			WINDOWS_EXPORT_ALL_SYMBOLS TRUE
			)
else()
	add_library(ssl STATIC ${LIBSRC})
endif()

target_include_directories(ssl PUBLIC ${PROJECT_BINARY_DIR}/include)

target_link_libraries(ssl crypto)

set_target_properties(ssl PROPERTIES
		VERSION "${LIB_VERSION}" SOVERSION "${LIB_SOVERSION}"
		)

if(ANDROID)
	set_target_properties(ssl PROPERTIES OUTPUT_NAME "ssl_1_1")
endif()

if(MSVC)
	if(CMAKE_SIZEOF_VOID_P EQUAL 8)
		set_target_properties(ssl PROPERTIES OUTPUT_NAME "ssl-1_1-x64")
	elseif(CMAKE_SIZEOF_VOID_P EQUAL 4)
		set_target_properties(ssl PROPERTIES OUTPUT_NAME "ssl-1_1")
	endif()
endif()

install(TARGETS ssl
		RUNTIME DESTINATION bin
		LIBRARY DESTINATION lib
		ARCHIVE DESTINATION lib
		)
