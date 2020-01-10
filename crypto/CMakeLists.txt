include(CMakeParseArguments)

string(TIMESTAMP BUILDINF_DATE "%Y-%m-%d %H:%M:%S UTC" UTC)
configure_file(buildinf.h.cmake buildinf.h)

include(CheckIncludeFile)
check_include_file(fcntl.h CHK_DLFCN)
if(CHK_DLFCN)
	set(HAVE_DLFCN_H ON)
endif()

if(APPLE)
	set(DSO_DLFCN ON)
	set(DSO_EXTENSION ".dylib")
elseif(WIN32 AND NOT CYGWIN)
	set(DSO_WIN32 ON)
	set(DSO_EXTENSION ".dll")
elseif(CYGWIN)
	set(DSO_DLFCN ON)
	set(DSO_EXTENSION ".dll")
else()
	if(CHK_DLFCN)
		set(DSO_DLFCN ON)
	else()
		set(DSO_NONE ON)
	endif()
	set(DSO_EXTENSION ".so")
endif()

file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/include/internal DESTINATION include)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/bn_conf.h.cmake
		${CMAKE_CURRENT_BINARY_DIR}/include/internal/bn_conf.h
		)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/dso_conf.h.cmake
		${CMAKE_CURRENT_BINARY_DIR}/include/internal/dso_conf.h
		)

set(LIBSRC cpt_err.c cryptlib.c ctype.c cversion.c ebcdic.c ex_data.c init.c 
		mem.c mem_clr.c mem_dbg.c mem_sec.c o_dir.c o_fips.c
		o_fopen.c o_init.c o_str.c o_time.c uid.c getenv.c
		)

macro(add_submodule dir)
	set(options)
	set(oneValueArgs)
	set(multiValueArgs EXHEADERS)
	cmake_parse_arguments(add_submodule "" "" "${multiValueArgs}" ${ARGN})

	#message(STATUS "{dir} ${dir}")
	#message(STATUS "{add_submodule_UNPARSED_ARGUMENTS} ${add_submodule_UNPARSED_ARGUMENTS}")
	foreach(name ${add_submodule_UNPARSED_ARGUMENTS})
		set(LIBSRC ${LIBSRC} ${dir}/${name})
	endforeach()

	#message(STATUS "{add_submodule_EXHEADERS} ${add_submodule_EXHEADERS}")
	foreach(name ${add_submodule_EXHEADERS})
		set(EXHEADERS ${EXHEADERS} ${name})
	endforeach()
endmacro()

add_submodule(aes aes_misc.c aes_ecb.c aes_cfb.c aes_ofb.c
		aes_ige.c aes_wrap.c aes_cbc.c aes_core.c
		)

add_submodule(aria aria.c)

add_submodule(asn1 a_object.c a_bitstr.c a_utctm.c a_gentm.c a_time.c a_int.c a_octet.c
		a_print.c a_type.c a_dup.c a_d2i_fp.c a_i2d_fp.c
		a_utf8.c a_sign.c a_digest.c a_verify.c a_mbstr.c a_strex.c
		x_algor.c x_val.c x_sig.c x_bignum.c
		x_long.c x_int64.c x_info.c x_spki.c nsseq.c
		d2i_pu.c d2i_pr.c i2d_pu.c i2d_pr.c
		t_pkey.c t_spki.c t_bitst.c
		tasn_new.c tasn_fre.c tasn_enc.c tasn_dec.c tasn_utl.c tasn_typ.c
		tasn_prn.c tasn_scn.c ameth_lib.c
		f_int.c f_string.c n_pkey.c
		x_pkey.c bio_asn1.c bio_ndef.c asn_mime.c
		asn1_gen.c asn1_par.c asn1_lib.c asn1_err.c a_strnid.c
		evp_asn1.c asn_pack.c p5_pbe.c p5_pbev2.c p5_scrypt.c p8_pkey.c
		asn_moid.c asn_mstbl.c asn1_item_list.c
		)

add_submodule(async async.c async_wait.c async_err.c arch/async_posix.c arch/async_win.c
		arch/async_null.c
		)

add_submodule(bf bf_skey.c bf_ecb.c bf_cfb64.c bf_ofb64.c bf_enc.c)

add_submodule(bio bio_lib.c bio_cb.c bio_err.c
		bss_mem.c bss_null.c bss_fd.c
		bss_file.c bss_sock.c bss_conn.c
		bf_null.c bf_buff.c b_print.c b_dump.c b_addr.c
		b_sock.c b_sock2.c bss_acpt.c bf_nbio.c bss_log.c bss_bio.c
		bss_dgram.c bio_meth.c #bf_lbuf.c
		)

add_submodule(blake2 blake2b.c blake2s.c m_blake2b.c m_blake2s.c)

add_submodule(bn 
		bn_add.c bn_div.c bn_exp.c bn_lib.c bn_ctx.c bn_mul.c bn_mod.c
		bn_print.c bn_rand.c bn_shift.c bn_word.c bn_blind.c
		bn_kron.c bn_sqrt.c bn_gcd.c bn_prime.c bn_err.c bn_sqr.c
		bn_asm.c rsaz_exp.c
		bn_recp.c bn_mont.c bn_mpi.c bn_exp2.c bn_gf2m.c bn_nist.c
		bn_depr.c bn_const.c bn_x931p.c bn_intern.c bn_dh.c bn_srp.c
		)

add_submodule(buffer buffer.c buf_err.c)

add_submodule(camellia cmll_ecb.c cmll_ofb.c cmll_cfb.c cmll_ctr.c
		camellia.c cmll_cbc.c cmll_misc.c
		)

add_submodule(cast c_skey.c c_ecb.c c_enc.c c_cfb64.c c_ofb64.c)

add_submodule(chacha chacha_enc.c)

add_submodule(cmac cmac.c cm_ameth.c cm_pmeth.c)

add_submodule(cms cms_lib.c cms_asn1.c cms_att.c cms_io.c cms_smime.c cms_err.c
		cms_sd.c cms_dd.c cms_cd.c cms_env.c cms_enc.c cms_ess.c
		cms_pwri.c cms_kari.c
		)

add_submodule(comp comp_lib.c comp_err.c
		c_zlib.c
		)

add_submodule(conf conf_err.c conf_lib.c conf_api.c conf_def.c conf_mod.c
		conf_mall.c conf_sap.c conf_ssl.c
		)

add_submodule(ct ct_b64.c ct_err.c ct_log.c ct_oct.c ct_policy.c
		ct_prn.c ct_sct.c ct_sct_ctx.c ct_vfy.c ct_x509v3.c
		)

add_submodule(des set_key.c ecb_enc.c cbc_enc.c
		ecb3_enc.c cfb64enc.c cfb64ede.c cfb_enc.c
		des_enc.c ofb64ede.c ofb64enc.c ofb_enc.c
		str2key.c pcbc_enc.c qud_cksm.c rand_key.c
		fcrypt.c fcrypt_b.c xcbc_enc.c cbc_cksm.c
		)

add_submodule(dh dh_asn1.c dh_gen.c dh_key.c dh_lib.c dh_check.c dh_err.c dh_depr.c
		dh_ameth.c dh_pmeth.c dh_prn.c dh_rfc5114.c dh_kdf.c dh_meth.c
		dh_rfc7919.c
		)

add_submodule(dsa dsa_gen.c dsa_key.c dsa_lib.c dsa_asn1.c dsa_vrf.c dsa_sign.c
		dsa_err.c dsa_ossl.c dsa_depr.c dsa_ameth.c dsa_pmeth.c dsa_prn.c
		dsa_meth.c
		)

add_submodule(dso dso_dl.c dso_dlfcn.c dso_err.c dso_lib.c
		dso_openssl.c dso_win32.c dso_vms.c
		)

add_submodule(ec ec_lib.c ecp_smpl.c ecp_mont.c ecp_nist.c ec_cvt.c ec_mult.c
		ec_err.c ec_curve.c ec_check.c ec_print.c ec_asn1.c ec_key.c
		ec2_smpl.c ec_ameth.c ec_pmeth.c eck_prn.c
		ecp_nistp224.c ecp_nistp256.c ecp_nistp521.c ecp_nistputil.c
		ecp_oct.c ec2_oct.c ec_oct.c ec_kmeth.c ecdh_ossl.c ecdh_kdf.c
		ecdsa_ossl.c ecdsa_sign.c ecdsa_vrf.c curve25519.c ecx_meth.c
		curve448/arch_32/f_impl.c curve448/f_generic.c curve448/scalar.c
		curve448/curve448_tables.c curve448/eddsa.c curve448/curve448.c
		)

add_submodule(engine eng_err.c eng_lib.c eng_list.c eng_init.c eng_ctrl.c
		eng_table.c eng_pkey.c eng_fat.c eng_all.c 
		tb_rsa.c tb_dsa.c tb_dh.c tb_rand.c
		tb_cipher.c tb_digest.c tb_pkmeth.c tb_asnmth.c tb_eckey.c
		eng_openssl.c eng_cnf.c eng_dyn.c
		eng_rdrand.c
		)

add_submodule(err err.c err_all.c err_prn.c)

add_submodule(evp encode.c digest.c evp_enc.c evp_key.c evp_cnf.c
		e_des.c e_bf.c e_idea.c e_des3.c e_camellia.c
		e_rc4.c e_aes.c names.c e_seed.c e_aria.c e_sm4.c
		e_xcbc_d.c e_rc2.c e_cast.c e_rc5.c
		m_null.c m_md2.c m_md4.c m_md5.c m_sha1.c m_wp.c
		m_md5_sha1.c m_mdc2.c m_ripemd.c m_sha3.c
		p_open.c p_seal.c p_sign.c p_verify.c p_lib.c p_enc.c p_dec.c
		bio_md.c bio_b64.c bio_enc.c evp_err.c e_null.c
		c_allc.c c_alld.c evp_lib.c bio_ok.c
		evp_pkey.c evp_pbe.c p5_crpt.c p5_crpt2.c pbe_scrypt.c
		e_old.c pmeth_lib.c pmeth_fn.c pmeth_gn.c m_sigver.c
		e_aes_cbc_hmac_sha1.c e_aes_cbc_hmac_sha256.c e_rc4_hmac_md5.c
		e_chacha20_poly1305.c cmeth_lib.c
		)

add_submodule(hmac hmac.c hm_ameth.c hm_pmeth.c)

add_submodule(idea i_cbc.c i_cfb64.c i_ofb64.c i_ecb.c i_skey.c)

add_submodule(kdf tls1_prf.c kdf_err.c hkdf.c scrypt.c)

add_submodule(lhash lhash.c lh_stats.c)

#add_submodule(md2 md2_dgst.c md2_one.c)

add_submodule(md4 md4_dgst.c md4_one.c)

add_submodule(md5 md5_dgst.c md5_one.c)

add_submodule(mdc2 mdc2dgst.c mdc2_one.c)

add_submodule(modes cbc128.c ctr128.c cts128.c cfb128.c ofb128.c gcm128.c
		ccm128.c xts128.c wrap128.c ocb128.c
		)

add_submodule(objects o_names.c obj_dat.c obj_lib.c obj_err.c obj_xref.c)

add_submodule(ocsp ocsp_asn.c ocsp_ext.c ocsp_ht.c ocsp_lib.c ocsp_cl.c
		ocsp_srv.c ocsp_prn.c ocsp_vfy.c ocsp_err.c v3_ocsp.c
		)

add_submodule(pem pem_sign.c pem_info.c pem_lib.c pem_all.c pem_err.c
		pem_x509.c pem_xaux.c pem_oth.c pem_pk8.c pem_pkey.c pvkfmt.c
		)

add_submodule(pkcs12 p12_add.c p12_asn.c p12_attr.c p12_crpt.c p12_crt.c p12_decr.c
		p12_init.c p12_key.c p12_kiss.c p12_mutl.c p12_sbag.c
		p12_utl.c p12_npas.c pk12err.c p12_p8d.c p12_p8e.c
		)

add_submodule(pkcs7 pk7_asn1.c pk7_lib.c pkcs7err.c pk7_doit.c pk7_smime.c pk7_attr.c
		pk7_mime.c bio_pk7.c
		)

add_submodule(poly1305 poly1305_pmeth.c poly1305_ameth.c poly1305.c)

add_submodule(rand randfile.c rand_lib.c rand_err.c rand_egd.c
        rand_win.c rand_unix.c rand_vms.c drbg_lib.c drbg_ctr.c
		)

add_submodule(rc2 rc2_ecb.c rc2_skey.c rc2_cbc.c rc2cfb64.c rc2ofb64.c)

add_submodule(rc4 rc4_enc.c rc4_skey.c)

#add_submodule(rc5 rc5_ecb.c rc5_enc.c rc5_skey.c rc5cfb64.c rc5ofb64.c)

add_submodule(ripemd rmd_dgst.c rmd_one.c)

add_submodule(rsa rsa_ossl.c rsa_gen.c rsa_lib.c rsa_sign.c rsa_saos.c rsa_err.c
		rsa_pk1.c rsa_ssl.c rsa_none.c rsa_oaep.c rsa_chk.c
		rsa_pss.c rsa_x931.c rsa_asn1.c rsa_depr.c rsa_ameth.c rsa_prn.c
		rsa_pmeth.c rsa_crpt.c rsa_x931g.c rsa_meth.c rsa_mp.c
		)

add_submodule(seed seed.c seed_ecb.c seed_cbc.c seed_cfb.c seed_ofb.c)

add_submodule(sha keccak1600.c sha1dgst.c sha1_one.c sha256.c sha512.c)
 
add_submodule(siphash siphash.c
		siphash_pmeth.c
		siphash_ameth.c
		)

add_submodule(sm2 sm2_sign.c sm2_crypt.c sm2_err.c sm2_pmeth.c)

add_submodule(sm3 sm3.c m_sm3.c)

add_submodule(sm4 sm4.c)

add_submodule(srp srp_lib.c srp_vfy.c)

add_submodule(stack stack.c)

add_submodule(store store_err.c store_init.c store_lib.c store_register.c store_strings.c
		loader_file.c)

add_submodule(ts ts_err.c ts_req_utils.c ts_req_print.c ts_rsp_utils.c ts_rsp_print.c
		ts_rsp_sign.c ts_rsp_verify.c ts_verify_ctx.c ts_lib.c ts_conf.c
		ts_asn1.c
		)

add_submodule(txt_db txt_db.c)

add_submodule(ui ui_err.c ui_lib.c ui_openssl.c ui_null.c ui_util.c)

add_submodule(whrlpool wp_block.c wp_dgst.c)

add_submodule(x509 x509_def.c x509_d2.c x509_r2x.c x509_cmp.c
		x509_obj.c x509_req.c x509spki.c x509_vfy.c
		x509_set.c x509cset.c x509rset.c x509_err.c
		x509name.c x509_v3.c x509_ext.c x509_att.c
		x509type.c x509_meth.c x509_lu.c x_all.c x509_txt.c
		x509_trs.c by_file.c by_dir.c x509_vpm.c
		x_crl.c t_crl.c x_req.c t_req.c x_x509.c t_x509.c
		x_pubkey.c x_x509a.c x_attrib.c x_exten.c x_name.c
		)

add_submodule(x509v3 v3_bcons.c v3_bitst.c v3_conf.c v3_extku.c v3_ia5.c v3_lib.c
		v3_prn.c v3_utl.c v3err.c v3_genn.c v3_alt.c v3_skey.c v3_akey.c v3_pku.c
		v3_int.c v3_enum.c v3_sxnet.c v3_cpols.c v3_crld.c v3_purp.c v3_info.c
		v3_akeya.c v3_pmaps.c v3_pcons.c v3_ncons.c v3_pcia.c v3_pci.c
		pcy_cache.c pcy_node.c pcy_data.c pcy_map.c pcy_tree.c pcy_lib.c
		v3_asid.c v3_addr.c v3_tlsf.c v3_admis.c
		)

set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
find_package(Threads)
if(NOT Threads_FOUND)
	add_submodule(. threads_none.c)
elseif(WIN32)
	add_submodule(. threads_win.c)
else()
	add_submodule(. threads_pthread.c)
endif()

if(BUILD_SHARED_OPENSSL)
	add_library(crypto SHARED ${LIBSRC} ${OBJECTS_SRC})
	set_target_properties(crypto PROPERTIES WINDOWS_EXPORT_ALL_SYMBOLS TRUE)
else()
	add_library(crypto STATIC ${LIBSRC} ${OBJECTS_SRC})
endif()

target_compile_definitions(crypto PUBLIC 
		OPENSSL_NO_ASM OPENSSL_NO_STATIC_ENGINE
		$<$<PLATFORM_ID:Windows>:WIN32_LEAN_AND_MEAN _CRT_SECURE_NO_WARNINGS>
		$<$<AND:$<PLATFORM_ID:Windows>,$<BOOL:${BUILD_SHARED_OPENSSL}>>:D_WINDLL>
		$<$<BOOL:${BUILD_SHARED_OPENSSL}>:OPENSSL_BUILD_SHLIBCRYPTO>
		)
if(WIN32)
	target_compile_definitions(crypto PUBLIC OPENSSL_SYSNAME_WIN32)
	target_compile_definitions(crypto PUBLIC
			"-DOPENSSLDIR=\"C:/ssl\"" 
			"-DENGINESDIR=\"C:/engines-1.1\""
			)
else()
	if(APPLE)
		target_compile_definitions(crypto PUBLIC OPENSSL_SYSNAME_MACOSX)
	endif()
	target_compile_definitions(crypto PUBLIC
			"-DOPENSSLDIR=\"/usr/local/ssl\""
			"-DENGINESDIR=\"/usr/local/engines-1.1\""
			)
endif()

target_include_directories(crypto PUBLIC BEFORE SYSTEM
		modes ec/curve448 ec/curve448/arch_32
		${CMAKE_CURRENT_BINARY_DIR}/include # internal
		${CMAKE_CURRENT_BINARY_DIR}/ # buildconf.h
		${PROJECT_SOURCE_DIR}/ # e_os.h
		${PROJECT_BINARY_DIR}/include # public
		)
target_include_directories(crypto PUBLIC ${PROJECT_BINARY_DIR}/include)

if(WIN32)
	target_link_libraries(crypto PRIVATE ws2_32 crypt32)
else()
	if(Threads_FOUND)
		target_link_libraries(crypto PRIVATE ${CMAKE_THREAD_LIBS_INIT})
	endif()
	if(DSO_DLFCN AND HAVE_DLFCN_H)
		target_link_libraries(crypto PRIVATE dl)
	endif()
endif()

set_target_properties(crypto PROPERTIES
		VERSION "${LIB_VERSION}" SOVERSION "${LIB_SOVERSION}"
		)
if(ANDROID)
	set_target_properties(crypto PROPERTIES OUTPUT_NAME "crypto_1_1")
elseif(MSVC)
	if(CMAKE_SIZEOF_VOID_P EQUAL 8)
		set_target_properties(crypto PROPERTIES OUTPUT_NAME "crypto-1_1-x64")
	elseif(CMAKE_SIZEOF_VOID_P EQUAL 4)
		set_target_properties(crypto PROPERTIES OUTPUT_NAME "crypto-1_1")
	endif()
endif()

install(TARGETS crypto
		RUNTIME DESTINATION bin
		LIBRARY DESTINATION lib
		ARCHIVE DESTINATION lib
		)
