# Copyright 2024 The BoringSSL Authors
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

project(
  'boringssl',
  ['c', 'cpp'],
  default_options : [
    'c_std=c11',
    'cpp_std=c++17',
    'default_library=static',
  ],
)

# ── Compiler detection ──────────────────────────────────────────────────────
cc  = meson.get_compiler('c')
cxx = meson.get_compiler('cpp')
is_clang = cc.get_id() == 'clang'
is_gcc   = cc.get_id() == 'gcc'
is_msvc  = cc.get_id() == 'msvc'

host_os  = host_machine.system()   # 'linux', 'darwin', 'windows', …
host_cpu = host_machine.cpu_family()  # 'x86_64', 'aarch64', 'x86', …

# ── Common compiler flags ────────────────────────────────────────────────────
common_c_cpp_args = []
if is_gcc or is_clang
  common_c_cpp_args += [
    '-fno-strict-aliasing',
    '-fno-common',
    '-fvisibility=hidden',
  ]
  if host_os != 'emscripten'
    common_c_cpp_args += ['-ggdb']
  endif
endif

if host_os == 'linux'
  common_c_cpp_args += ['-D_XOPEN_SOURCE=700']
endif

if host_os == 'windows'
  common_c_cpp_args += [
    '-D_HAS_EXCEPTIONS=0',
    '-DWIN32_LEAN_AND_MEAN',
    '-DNOMINMAX',
    '-D_CRT_SECURE_NO_WARNINGS',
  ]
endif

# Disable C++ runtime features in the crypto libraries (matches CMake config)
no_cxx_runtime_flags = []
if not is_msvc
  no_cxx_runtime_flags = ['-fno-exceptions', '-fno-rtti']
endif

impl_define = ['-DBORINGSSL_IMPLEMENTATION']

# ── Include directories ──────────────────────────────────────────────────────
inc = include_directories('include')

# ── Thread dependency ────────────────────────────────────────────────────────
# On Generic/Android we skip the threads package; add only for real OSes.
thread_dep = []
if host_os not in ['none', 'android']
  thread_dep = [dependency('threads')]
endif

# ── Source lists (generated from gen/sources.cmake) ─────────────────────────

bcm_sources = files(
  'crypto/fipsmodule/bcm.cc',
)

bcm_sources_asm = files(
  'gen/bcm/aes-gcm-avx2-x86_64-apple.S',
  'gen/bcm/aes-gcm-avx2-x86_64-linux.S',
  'gen/bcm/aes-gcm-avx512-x86_64-apple.S',
  'gen/bcm/aes-gcm-avx512-x86_64-linux.S',
  'gen/bcm/aesni-gcm-x86_64-apple.S',
  'gen/bcm/aesni-gcm-x86_64-linux.S',
  'gen/bcm/aesni-x86-apple.S',
  'gen/bcm/aesni-x86-linux.S',
  'gen/bcm/aesni-x86_64-apple.S',
  'gen/bcm/aesni-x86_64-linux.S',
  'gen/bcm/aesv8-armv7-linux.S',
  'gen/bcm/aesv8-armv8-apple.S',
  'gen/bcm/aesv8-armv8-linux.S',
  'gen/bcm/aesv8-armv8-win.S',
  'gen/bcm/aesv8-gcm-armv8-apple.S',
  'gen/bcm/aesv8-gcm-armv8-linux.S',
  'gen/bcm/aesv8-gcm-armv8-win.S',
  'gen/bcm/armv4-mont-linux.S',
  'gen/bcm/armv8-mont-apple.S',
  'gen/bcm/armv8-mont-linux.S',
  'gen/bcm/armv8-mont-win.S',
  'gen/bcm/bn-586-apple.S',
  'gen/bcm/bn-586-linux.S',
  'gen/bcm/bn-armv8-apple.S',
  'gen/bcm/bn-armv8-linux.S',
  'gen/bcm/bn-armv8-win.S',
  'gen/bcm/bsaes-armv7-linux.S',
  'gen/bcm/co-586-apple.S',
  'gen/bcm/co-586-linux.S',
  'gen/bcm/ghash-armv4-linux.S',
  'gen/bcm/ghash-neon-armv8-apple.S',
  'gen/bcm/ghash-neon-armv8-linux.S',
  'gen/bcm/ghash-neon-armv8-win.S',
  'gen/bcm/ghash-ssse3-x86-apple.S',
  'gen/bcm/ghash-ssse3-x86-linux.S',
  'gen/bcm/ghash-ssse3-x86_64-apple.S',
  'gen/bcm/ghash-ssse3-x86_64-linux.S',
  'gen/bcm/ghash-x86-apple.S',
  'gen/bcm/ghash-x86-linux.S',
  'gen/bcm/ghash-x86_64-apple.S',
  'gen/bcm/ghash-x86_64-linux.S',
  'gen/bcm/ghashv8-armv7-linux.S',
  'gen/bcm/ghashv8-armv8-apple.S',
  'gen/bcm/ghashv8-armv8-linux.S',
  'gen/bcm/ghashv8-armv8-win.S',
  'gen/bcm/p256-armv8-asm-apple.S',
  'gen/bcm/p256-armv8-asm-linux.S',
  'gen/bcm/p256-armv8-asm-win.S',
  'gen/bcm/p256-x86_64-asm-apple.S',
  'gen/bcm/p256-x86_64-asm-linux.S',
  'gen/bcm/p256_beeu-armv8-asm-apple.S',
  'gen/bcm/p256_beeu-armv8-asm-linux.S',
  'gen/bcm/p256_beeu-armv8-asm-win.S',
  'gen/bcm/p256_beeu-x86_64-asm-apple.S',
  'gen/bcm/p256_beeu-x86_64-asm-linux.S',
  'gen/bcm/rdrand-x86_64-apple.S',
  'gen/bcm/rdrand-x86_64-linux.S',
  'gen/bcm/rsaz-avx2-apple.S',
  'gen/bcm/rsaz-avx2-linux.S',
  'gen/bcm/sha1-586-apple.S',
  'gen/bcm/sha1-586-linux.S',
  'gen/bcm/sha1-armv4-large-linux.S',
  'gen/bcm/sha1-armv8-apple.S',
  'gen/bcm/sha1-armv8-linux.S',
  'gen/bcm/sha1-armv8-win.S',
  'gen/bcm/sha1-x86_64-apple.S',
  'gen/bcm/sha1-x86_64-linux.S',
  'gen/bcm/sha256-586-apple.S',
  'gen/bcm/sha256-586-linux.S',
  'gen/bcm/sha256-armv4-linux.S',
  'gen/bcm/sha256-armv8-apple.S',
  'gen/bcm/sha256-armv8-linux.S',
  'gen/bcm/sha256-armv8-win.S',
  'gen/bcm/sha256-x86_64-apple.S',
  'gen/bcm/sha256-x86_64-linux.S',
  'gen/bcm/sha512-586-apple.S',
  'gen/bcm/sha512-586-linux.S',
  'gen/bcm/sha512-armv4-linux.S',
  'gen/bcm/sha512-armv8-apple.S',
  'gen/bcm/sha512-armv8-linux.S',
  'gen/bcm/sha512-armv8-win.S',
  'gen/bcm/sha512-x86_64-apple.S',
  'gen/bcm/sha512-x86_64-linux.S',
  'gen/bcm/vpaes-armv7-linux.S',
  'gen/bcm/vpaes-armv8-apple.S',
  'gen/bcm/vpaes-armv8-linux.S',
  'gen/bcm/vpaes-armv8-win.S',
  'gen/bcm/vpaes-x86-apple.S',
  'gen/bcm/vpaes-x86-linux.S',
  'gen/bcm/vpaes-x86_64-apple.S',
  'gen/bcm/vpaes-x86_64-linux.S',
  'gen/bcm/x86-mont-apple.S',
  'gen/bcm/x86-mont-linux.S',
  'gen/bcm/x86_64-mont-apple.S',
  'gen/bcm/x86_64-mont-linux.S',
  'gen/bcm/x86_64-mont5-apple.S',
  'gen/bcm/x86_64-mont5-linux.S',
  'third_party/fiat/asm/fiat_p256_adx_mul.S',
  'third_party/fiat/asm/fiat_p256_adx_sqr.S',
)

bcm_sources_nasm = files(
  'gen/bcm/aes-gcm-avx2-x86_64-win.asm',
  'gen/bcm/aes-gcm-avx512-x86_64-win.asm',
  'gen/bcm/aesni-gcm-x86_64-win.asm',
  'gen/bcm/aesni-x86-win.asm',
  'gen/bcm/aesni-x86_64-win.asm',
  'gen/bcm/bn-586-win.asm',
  'gen/bcm/co-586-win.asm',
  'gen/bcm/ghash-ssse3-x86-win.asm',
  'gen/bcm/ghash-ssse3-x86_64-win.asm',
  'gen/bcm/ghash-x86-win.asm',
  'gen/bcm/ghash-x86_64-win.asm',
  'gen/bcm/p256-x86_64-asm-win.asm',
  'gen/bcm/p256_beeu-x86_64-asm-win.asm',
  'gen/bcm/rdrand-x86_64-win.asm',
  'gen/bcm/rsaz-avx2-win.asm',
  'gen/bcm/sha1-586-win.asm',
  'gen/bcm/sha1-x86_64-win.asm',
  'gen/bcm/sha256-586-win.asm',
  'gen/bcm/sha256-x86_64-win.asm',
  'gen/bcm/sha512-586-win.asm',
  'gen/bcm/sha512-x86_64-win.asm',
  'gen/bcm/vpaes-x86-win.asm',
  'gen/bcm/vpaes-x86_64-win.asm',
  'gen/bcm/x86-mont-win.asm',
  'gen/bcm/x86_64-mont-win.asm',
  'gen/bcm/x86_64-mont5-win.asm',
)

crypto_sources = files(
  'crypto/aes/aes.cc',
  'crypto/asn1/a_bitstr.cc',
  'crypto/asn1/a_bool.cc',
  'crypto/asn1/a_d2i_fp.cc',
  'crypto/asn1/a_dup.cc',
  'crypto/asn1/a_gentm.cc',
  'crypto/asn1/a_i2d_fp.cc',
  'crypto/asn1/a_int.cc',
  'crypto/asn1/a_mbstr.cc',
  'crypto/asn1/a_object.cc',
  'crypto/asn1/a_octet.cc',
  'crypto/asn1/a_strex.cc',
  'crypto/asn1/a_strnid.cc',
  'crypto/asn1/a_time.cc',
  'crypto/asn1/a_type.cc',
  'crypto/asn1/a_utctm.cc',
  'crypto/asn1/asn1_lib.cc',
  'crypto/asn1/asn1_par.cc',
  'crypto/asn1/asn_pack.cc',
  'crypto/asn1/f_int.cc',
  'crypto/asn1/f_string.cc',
  'crypto/asn1/posix_time.cc',
  'crypto/asn1/tasn_dec.cc',
  'crypto/asn1/tasn_enc.cc',
  'crypto/asn1/tasn_fre.cc',
  'crypto/asn1/tasn_new.cc',
  'crypto/asn1/tasn_typ.cc',
  'crypto/asn1/tasn_utl.cc',
  'crypto/base64/base64.cc',
  'crypto/bio/bio.cc',
  'crypto/bio/bio_mem.cc',
  'crypto/bio/connect.cc',
  'crypto/bio/errno.cc',
  'crypto/bio/fd.cc',
  'crypto/bio/file.cc',
  'crypto/bio/hexdump.cc',
  'crypto/bio/pair.cc',
  'crypto/bio/printf.cc',
  'crypto/bio/socket.cc',
  'crypto/bio/socket_helper.cc',
  'crypto/blake2/blake2.cc',
  'crypto/bn/bn_asn1.cc',
  'crypto/bn/convert.cc',
  'crypto/bn/div.cc',
  'crypto/bn/exponentiation.cc',
  'crypto/bn/sqrt.cc',
  'crypto/buf/buf.cc',
  'crypto/bytestring/asn1_compat.cc',
  'crypto/bytestring/ber.cc',
  'crypto/bytestring/cbb.cc',
  'crypto/bytestring/cbs.cc',
  'crypto/bytestring/unicode.cc',
  'crypto/chacha/chacha.cc',
  'crypto/cipher/derive_key.cc',
  'crypto/cipher/e_aesctrhmac.cc',
  'crypto/cipher/e_aeseax.cc',
  'crypto/cipher/e_aesgcmsiv.cc',
  'crypto/cipher/e_chacha20poly1305.cc',
  'crypto/cipher/e_des.cc',
  'crypto/cipher/e_null.cc',
  'crypto/cipher/e_rc2.cc',
  'crypto/cipher/e_rc4.cc',
  'crypto/cipher/e_tls.cc',
  'crypto/cipher/get_cipher.cc',
  'crypto/cipher/tls_cbc.cc',
  'crypto/cms/cms.cc',
  'crypto/conf/conf.cc',
  'crypto/cpu_aarch64_apple.cc',
  'crypto/cpu_aarch64_fuchsia.cc',
  'crypto/cpu_aarch64_linux.cc',
  'crypto/cpu_aarch64_openbsd.cc',
  'crypto/cpu_aarch64_sysreg.cc',
  'crypto/cpu_aarch64_win.cc',
  'crypto/cpu_arm_freebsd.cc',
  'crypto/cpu_arm_linux.cc',
  'crypto/cpu_intel.cc',
  'crypto/crypto.cc',
  'crypto/curve25519/curve25519.cc',
  'crypto/curve25519/curve25519_64_adx.cc',
  'crypto/curve25519/spake25519.cc',
  'crypto/des/des.cc',
  'crypto/dh/dh_asn1.cc',
  'crypto/dh/params.cc',
  'crypto/digest/digest_extra.cc',
  'crypto/dsa/dsa.cc',
  'crypto/dsa/dsa_asn1.cc',
  'crypto/ec/ec_asn1.cc',
  'crypto/ec/ec_derive.cc',
  'crypto/ec/hash_to_curve.cc',
  'crypto/ecdh/ecdh.cc',
  'crypto/ecdsa/ecdsa_asn1.cc',
  'crypto/ecdsa/ecdsa_p1363.cc',
  'crypto/engine/engine.cc',
  'crypto/err/err.cc',
  'crypto/evp/evp.cc',
  'crypto/evp/evp_asn1.cc',
  'crypto/evp/evp_ctx.cc',
  'crypto/evp/evp_kem.cc',
  'crypto/evp/p_dh.cc',
  'crypto/evp/p_dsa.cc',
  'crypto/evp/p_ec.cc',
  'crypto/evp/p_ed25519.cc',
  'crypto/evp/p_hkdf.cc',
  'crypto/evp/p_mldsa.cc',
  'crypto/evp/p_mlkem.cc',
  'crypto/evp/p_rsa.cc',
  'crypto/evp/p_x25519.cc',
  'crypto/evp/pbkdf.cc',
  'crypto/evp/print.cc',
  'crypto/evp/scrypt.cc',
  'crypto/evp/sign.cc',
  'crypto/ex_data.cc',
  'crypto/fipsmodule/fips_shared_support.cc',
  'crypto/fuzzer_mode.cc',
  'crypto/hpke/hpke.cc',
  'crypto/hrss/hrss.cc',
  'crypto/kyber/kyber.cc',
  'crypto/lhash/lhash.cc',
  'crypto/md4/md4.cc',
  'crypto/md5/md5.cc',
  'crypto/mem.cc',
  'crypto/mldsa/mldsa.cc',
  'crypto/mlkem/mlkem.cc',
  'crypto/obj/obj.cc',
  'crypto/obj/obj_xref.cc',
  'crypto/pem/pem_all.cc',
  'crypto/pem/pem_info.cc',
  'crypto/pem/pem_lib.cc',
  'crypto/pem/pem_oth.cc',
  'crypto/pem/pem_pk8.cc',
  'crypto/pem/pem_pkey.cc',
  'crypto/pem/pem_x509.cc',
  'crypto/pem/pem_xaux.cc',
  'crypto/pkcs7/pkcs7.cc',
  'crypto/pkcs7/pkcs7_x509.cc',
  'crypto/pkcs8/p5_pbev2.cc',
  'crypto/pkcs8/pkcs8.cc',
  'crypto/pkcs8/pkcs8_x509.cc',
  'crypto/poly1305/poly1305.cc',
  'crypto/poly1305/poly1305_arm.cc',
  'crypto/poly1305/poly1305_vec.cc',
  'crypto/pool/pool.cc',
  'crypto/rand/deterministic.cc',
  'crypto/rand/fork_detect.cc',
  'crypto/rand/forkunsafe.cc',
  'crypto/rand/getentropy.cc',
  'crypto/rand/ios.cc',
  'crypto/rand/passive.cc',
  'crypto/rand/rand.cc',
  'crypto/rand/trusty.cc',
  'crypto/rand/urandom.cc',
  'crypto/rand/windows.cc',
  'crypto/rc4/rc4.cc',
  'crypto/refcount.cc',
  'crypto/rsa/rsa_asn1.cc',
  'crypto/rsa/rsa_crypt.cc',
  'crypto/rsa/rsa_extra.cc',
  'crypto/rsa/rsa_print.cc',
  'crypto/sha/sha1.cc',
  'crypto/sha/sha256.cc',
  'crypto/sha/sha512.cc',
  'crypto/siphash/siphash.cc',
  'crypto/slhdsa/slhdsa.cc',
  'crypto/spake2plus/spake2plus.cc',
  'crypto/stack/stack.cc',
  'crypto/thread.cc',
  'crypto/thread_none.cc',
  'crypto/thread_pthread.cc',
  'crypto/thread_win.cc',
  'crypto/trust_token/pmbtoken.cc',
  'crypto/trust_token/trust_token.cc',
  'crypto/trust_token/voprf.cc',
  'crypto/x509/a_digest.cc',
  'crypto/x509/a_sign.cc',
  'crypto/x509/a_verify.cc',
  'crypto/x509/algorithm.cc',
  'crypto/x509/asn1_gen.cc',
  'crypto/x509/by_dir.cc',
  'crypto/x509/by_file.cc',
  'crypto/x509/i2d_pr.cc',
  'crypto/x509/name_print.cc',
  'crypto/x509/policy.cc',
  'crypto/x509/rsa_pss.cc',
  'crypto/x509/t_crl.cc',
  'crypto/x509/t_req.cc',
  'crypto/x509/t_x509.cc',
  'crypto/x509/t_x509a.cc',
  'crypto/x509/v3_akey.cc',
  'crypto/x509/v3_akeya.cc',
  'crypto/x509/v3_alt.cc',
  'crypto/x509/v3_bcons.cc',
  'crypto/x509/v3_bitst.cc',
  'crypto/x509/v3_conf.cc',
  'crypto/x509/v3_cpols.cc',
  'crypto/x509/v3_crld.cc',
  'crypto/x509/v3_enum.cc',
  'crypto/x509/v3_extku.cc',
  'crypto/x509/v3_genn.cc',
  'crypto/x509/v3_ia5.cc',
  'crypto/x509/v3_info.cc',
  'crypto/x509/v3_int.cc',
  'crypto/x509/v3_lib.cc',
  'crypto/x509/v3_ncons.cc',
  'crypto/x509/v3_ocsp.cc',
  'crypto/x509/v3_pcons.cc',
  'crypto/x509/v3_pmaps.cc',
  'crypto/x509/v3_prn.cc',
  'crypto/x509/v3_purp.cc',
  'crypto/x509/v3_skey.cc',
  'crypto/x509/v3_utl.cc',
  'crypto/x509/x509.cc',
  'crypto/x509/x509_att.cc',
  'crypto/x509/x509_cmp.cc',
  'crypto/x509/x509_d2.cc',
  'crypto/x509/x509_def.cc',
  'crypto/x509/x509_ext.cc',
  'crypto/x509/x509_lu.cc',
  'crypto/x509/x509_obj.cc',
  'crypto/x509/x509_req.cc',
  'crypto/x509/x509_set.cc',
  'crypto/x509/x509_trs.cc',
  'crypto/x509/x509_txt.cc',
  'crypto/x509/x509_v3.cc',
  'crypto/x509/x509_vfy.cc',
  'crypto/x509/x509_vpm.cc',
  'crypto/x509/x509cset.cc',
  'crypto/x509/x509name.cc',
  'crypto/x509/x509rset.cc',
  'crypto/x509/x509spki.cc',
  'crypto/x509/x_algor.cc',
  'crypto/x509/x_all.cc',
  'crypto/x509/x_attrib.cc',
  'crypto/x509/x_crl.cc',
  'crypto/x509/x_exten.cc',
  'crypto/x509/x_name.cc',
  'crypto/x509/x_pubkey.cc',
  'crypto/x509/x_req.cc',
  'crypto/x509/x_sig.cc',
  'crypto/x509/x_spki.cc',
  'crypto/x509/x_x509.cc',
  'crypto/x509/x_x509a.cc',
  'crypto/xwing/xwing.cc',
  'gen/crypto/err_data.cc',
)

crypto_sources_asm = files(
  'crypto/curve25519/asm/x25519-asm-arm.S',
  'crypto/hrss/asm/poly_rq_mul.S',
  'crypto/poly1305/poly1305_arm_asm.S',
  'gen/crypto/aes128gcmsiv-x86_64-apple.S',
  'gen/crypto/aes128gcmsiv-x86_64-linux.S',
  'gen/crypto/chacha-armv4-linux.S',
  'gen/crypto/chacha-armv8-apple.S',
  'gen/crypto/chacha-armv8-linux.S',
  'gen/crypto/chacha-armv8-win.S',
  'gen/crypto/chacha-x86-apple.S',
  'gen/crypto/chacha-x86-linux.S',
  'gen/crypto/chacha-x86_64-apple.S',
  'gen/crypto/chacha-x86_64-linux.S',
  'gen/crypto/chacha20_poly1305_armv8-apple.S',
  'gen/crypto/chacha20_poly1305_armv8-linux.S',
  'gen/crypto/chacha20_poly1305_armv8-win.S',
  'gen/crypto/chacha20_poly1305_x86_64-apple.S',
  'gen/crypto/chacha20_poly1305_x86_64-linux.S',
  'gen/crypto/md5-586-apple.S',
  'gen/crypto/md5-586-linux.S',
  'gen/crypto/md5-x86_64-apple.S',
  'gen/crypto/md5-x86_64-linux.S',
  'third_party/fiat/asm/fiat_curve25519_adx_mul.S',
  'third_party/fiat/asm/fiat_curve25519_adx_square.S',
)

crypto_sources_nasm = files(
  'gen/crypto/aes128gcmsiv-x86_64-win.asm',
  'gen/crypto/chacha-x86-win.asm',
  'gen/crypto/chacha-x86_64-win.asm',
  'gen/crypto/chacha20_poly1305_x86_64-win.asm',
  'gen/crypto/md5-586-win.asm',
  'gen/crypto/md5-x86_64-win.asm',
)

ssl_sources = files(
  'ssl/bio_ssl.cc',
  'ssl/d1_both.cc',
  'ssl/d1_lib.cc',
  'ssl/d1_pkt.cc',
  'ssl/d1_srtp.cc',
  'ssl/dtls_method.cc',
  'ssl/dtls_record.cc',
  'ssl/encrypted_client_hello.cc',
  'ssl/extensions.cc',
  'ssl/handoff.cc',
  'ssl/handshake.cc',
  'ssl/handshake_client.cc',
  'ssl/handshake_server.cc',
  'ssl/s3_both.cc',
  'ssl/s3_lib.cc',
  'ssl/s3_pkt.cc',
  'ssl/ssl_aead_ctx.cc',
  'ssl/ssl_asn1.cc',
  'ssl/ssl_buffer.cc',
  'ssl/ssl_cert.cc',
  'ssl/ssl_cipher.cc',
  'ssl/ssl_credential.cc',
  'ssl/ssl_file.cc',
  'ssl/ssl_key_share.cc',
  'ssl/ssl_lib.cc',
  'ssl/ssl_privkey.cc',
  'ssl/ssl_session.cc',
  'ssl/ssl_stat.cc',
  'ssl/ssl_transcript.cc',
  'ssl/ssl_versions.cc',
  'ssl/ssl_x509.cc',
  'ssl/t1_enc.cc',
  'ssl/tls13_both.cc',
  'ssl/tls13_client.cc',
  'ssl/tls13_enc.cc',
  'ssl/tls13_server.cc',
  'ssl/tls_method.cc',
  'ssl/tls_record.cc',
)

decrepit_sources = files(
  'decrepit/bio/base64_bio.cc',
  'decrepit/blowfish/blowfish.cc',
  'decrepit/cast/cast.cc',
  'decrepit/cast/cast_tables.cc',
  'decrepit/cfb/cfb.cc',
  'decrepit/des/cfb64ede.cc',
  'decrepit/dh/dh_decrepit.cc',
  'decrepit/dsa/dsa_decrepit.cc',
  'decrepit/evp/dss1.cc',
  'decrepit/evp/evp_do_all.cc',
  'decrepit/obj/obj_decrepit.cc',
  'decrepit/rc4/rc4_decrepit.cc',
  'decrepit/ripemd/ripemd.cc',
  'decrepit/rsa/rsa_decrepit.cc',
  'decrepit/ssl/ssl_decrepit.cc',
  'decrepit/x509/x509_decrepit.cc',
  'decrepit/xts/xts.cc',
)

pki_sources = files(
  'pki/cert_error_id.cc',
  'pki/cert_error_params.cc',
  'pki/cert_errors.cc',
  'pki/cert_issuer_source_static.cc',
  'pki/certificate.cc',
  'pki/certificate_policies.cc',
  'pki/common_cert_errors.cc',
  'pki/crl.cc',
  'pki/encode_values.cc',
  'pki/extended_key_usage.cc',
  'pki/general_names.cc',
  'pki/input.cc',
  'pki/ip_util.cc',
  'pki/merkle_tree.cc',
  'pki/name_constraints.cc',
  'pki/ocsp.cc',
  'pki/parse_certificate.cc',
  'pki/parse_name.cc',
  'pki/parse_values.cc',
  'pki/parsed_certificate.cc',
  'pki/parser.cc',
  'pki/path_builder.cc',
  'pki/pem.cc',
  'pki/revocation_util.cc',
  'pki/signature_algorithm.cc',
  'pki/simple_path_builder_delegate.cc',
  'pki/string_util.cc',
  'pki/trust_store.cc',
  'pki/trust_store_collection.cc',
  'pki/trust_store_in_memory.cc',
  'pki/verify.cc',
  'pki/verify_certificate_chain.cc',
  'pki/verify_error.cc',
  'pki/verify_name_match.cc',
  'pki/verify_signed_data.cc',
)

bssl_sources = files(
  'tool/args.cc',
  'tool/ciphers.cc',
  'tool/client.cc',
  'tool/const.cc',
  'tool/digest.cc',
  'tool/fd.cc',
  'tool/file.cc',
  'tool/generate_ech.cc',
  'tool/generate_ed25519.cc',
  'tool/genrsa.cc',
  'tool/pkcs12.cc',
  'tool/rand.cc',
  'tool/server.cc',
  'tool/sign.cc',
  'tool/tool.cc',
  'tool/transport_common.cc',
)

test_support_sources = files(
  'crypto/test/abi_test.cc',
  'crypto/test/der_trailing_data.cc',
  'crypto/test/file_test.cc',
  'crypto/test/file_test_gtest.cc',
  'crypto/test/file_util.cc',
  'crypto/test/test_data.cc',
  'crypto/test/test_util.cc',
  'crypto/test/wycheproof_util.cc',
)

test_support_sources_asm = files(
  'gen/test_support/trampoline-armv4-linux.S',
  'gen/test_support/trampoline-armv8-apple.S',
  'gen/test_support/trampoline-armv8-linux.S',
  'gen/test_support/trampoline-armv8-win.S',
  'gen/test_support/trampoline-x86-apple.S',
  'gen/test_support/trampoline-x86-linux.S',
  'gen/test_support/trampoline-x86_64-apple.S',
  'gen/test_support/trampoline-x86_64-linux.S',
)

test_support_sources_nasm = files(
  'gen/test_support/trampoline-x86-win.asm',
  'gen/test_support/trampoline-x86_64-win.asm',
)

crypto_test_sources = files(
  'crypto/abi_self_test.cc',
  'crypto/asn1/asn1_test.cc',
  'crypto/base64/base64_test.cc',
  'crypto/bio/bio_test.cc',
  'crypto/blake2/blake2_test.cc',
  'crypto/buf/buf_test.cc',
  'crypto/bytestring/bytestring_test.cc',
  'crypto/chacha/chacha_test.cc',
  'crypto/cipher/aead_test.cc',
  'crypto/cipher/cipher_test.cc',
  'crypto/cms/cms_test.cc',
  'crypto/compiler_test.cc',
  'crypto/conf/conf_test.cc',
  'crypto/constant_time_test.cc',
  'crypto/cpu_arm_linux_test.cc',
  'crypto/crypto_test.cc',
  'crypto/curve25519/ed25519_test.cc',
  'crypto/curve25519/spake25519_test.cc',
  'crypto/curve25519/x25519_test.cc',
  'crypto/dh/dh_test.cc',
  'crypto/digest/digest_test.cc',
  'crypto/dsa/dsa_test.cc',
  'crypto/ecdh/ecdh_test.cc',
  'crypto/ecdsa/ecdsa_p1363_test.cc',
  'crypto/err/err_test.cc',
  'crypto/evp/evp_extra_test.cc',
  'crypto/evp/evp_test.cc',
  'crypto/evp/pbkdf_test.cc',
  'crypto/evp/scrypt_test.cc',
  'crypto/fipsmodule/aes/aes_test.cc',
  'crypto/fipsmodule/aes/gcm_test.cc',
  'crypto/fipsmodule/bn/bn_test.cc',
  'crypto/fipsmodule/cmac/cmac_test.cc',
  'crypto/fipsmodule/ec/ec_test.cc',
  'crypto/fipsmodule/ec/p256-nistz_test.cc',
  'crypto/fipsmodule/ec/p256_test.cc',
  'crypto/fipsmodule/ecdsa/ecdsa_test.cc',
  'crypto/fipsmodule/entropy/jitter_test.cc',
  'crypto/fipsmodule/hkdf/hkdf_test.cc',
  'crypto/fipsmodule/keccak/keccak_test.cc',
  'crypto/fipsmodule/rand/ctrdrbg_test.cc',
  'crypto/fipsmodule/service_indicator/service_indicator_test.cc',
  'crypto/fipsmodule/sha/sha_test.cc',
  'crypto/hmac/hmac_test.cc',
  'crypto/hpke/hpke_test.cc',
  'crypto/hrss/hrss_test.cc',
  'crypto/impl_dispatch_test.cc',
  'crypto/kyber/kyber_test.cc',
  'crypto/lhash/lhash_test.cc',
  'crypto/md5/md5_test.cc',
  'crypto/mem_test.cc',
  'crypto/mldsa/mldsa_test.cc',
  'crypto/mlkem/mlkem_test.cc',
  'crypto/obj/obj_test.cc',
  'crypto/pem/pem_test.cc',
  'crypto/pkcs7/pkcs7_test.cc',
  'crypto/pkcs8/pkcs12_test.cc',
  'crypto/pkcs8/pkcs8_test.cc',
  'crypto/poly1305/poly1305_test.cc',
  'crypto/pool/pool_test.cc',
  'crypto/rand/fork_detect_test.cc',
  'crypto/rand/getentropy_test.cc',
  'crypto/rand/rand_test.cc',
  'crypto/refcount_test.cc',
  'crypto/rsa/rsa_test.cc',
  'crypto/self_test.cc',
  'crypto/siphash/siphash_test.cc',
  'crypto/slhdsa/slhdsa_test.cc',
  'crypto/spake2plus/spake2plus_test.cc',
  'crypto/stack/stack_test.cc',
  'crypto/test/gtest_main.cc',
  'crypto/thread_test.cc',
  'crypto/trust_token/trust_token_test.cc',
  'crypto/x509/x509_test.cc',
  'crypto/x509/x509_time_test.cc',
  'crypto/xwing/xwing_test.cc',
  'third_party/fiat/bedrock_platform_test.cc',
)

ssl_test_sources = files(
  'crypto/test/gtest_main.cc',
  'ssl/span_test.cc',
  'ssl/ssl_c_test.c',
  'ssl/ssl_internal_test.cc',
  'ssl/ssl_test.cc',
)

decrepit_test_sources = files(
  'crypto/test/gtest_main.cc',
  'decrepit/blowfish/blowfish_test.cc',
  'decrepit/cast/cast_test.cc',
  'decrepit/cfb/cfb_test.cc',
  'decrepit/des/des_test.cc',
  'decrepit/evp/evp_test.cc',
  'decrepit/ripemd/ripemd_test.cc',
  'decrepit/xts/xts_test.cc',
)

pki_test_sources = files(
  'crypto/test/gtest_main.cc',
  'pki/cert_issuer_source_static_unittest.cc',
  'pki/certificate_policies_unittest.cc',
  'pki/certificate_unittest.cc',
  'pki/crl_unittest.cc',
  'pki/encode_values_unittest.cc',
  'pki/extended_key_usage_unittest.cc',
  'pki/general_names_unittest.cc',
  'pki/input_unittest.cc',
  'pki/ip_util_unittest.cc',
  'pki/merkle_tree_unittest.cc',
  'pki/mock_signature_verify_cache.cc',
  'pki/name_constraints_unittest.cc',
  'pki/nist_pkits_unittest.cc',
  'pki/ocsp_unittest.cc',
  'pki/parse_certificate_unittest.cc',
  'pki/parse_name_unittest.cc',
  'pki/parse_values_unittest.cc',
  'pki/parsed_certificate_unittest.cc',
  'pki/parser_unittest.cc',
  'pki/path_builder_pkits_unittest.cc',
  'pki/path_builder_unittest.cc',
  'pki/path_builder_verify_certificate_chain_unittest.cc',
  'pki/pem_unittest.cc',
  'pki/signature_algorithm_unittest.cc',
  'pki/simple_path_builder_delegate_unittest.cc',
  'pki/string_util_unittest.cc',
  'pki/test_helpers.cc',
  'pki/trust_store_collection_unittest.cc',
  'pki/trust_store_in_memory_unittest.cc',
  'pki/verify_certificate_chain_pkits_unittest.cc',
  'pki/verify_certificate_chain_unittest.cc',
  'pki/verify_name_match_unittest.cc',
  'pki/verify_signed_data_unittest.cc',
  'pki/verify_unittest.cc',
)

urandom_test_sources = files(
  'crypto/rand/urandom_test.cc',
)

bench_sources = files(
  'bench/aead.cc',
  'bench/aes.cc',
  'bench/base64.cc',
  'bench/curve25519.cc',
  'bench/ecdh.cc',
  'bench/ecdsa.cc',
  'bench/hash.cc',
  'bench/hash_to_curve.cc',
  'bench/hrss.cc',
  'bench/main.cc',
  'bench/mldsa.cc',
  'bench/mlkem.cc',
  'bench/rand.cc',
  'bench/rsa.cc',
  'bench/rsakeygen.cc',
  'bench/scrypt.cc',
  'bench/selftest.cc',
  'bench/siphash.cc',
  'bench/slhdsa.cc',
  'bench/spake2.cc',
  'bench/trust_token.cc',
  'bench/x509.cc',
)

# ── Assembly file selection ──────────────────────────────────────────────────
# All .S files are pre-processed by the C preprocessor and contain
# #if guards (e.g. `#if defined(OPENSSL_X86_64) && defined(__ELF__)`)
# so irrelevant files assemble to empty translation units on wrong platforms.
# On Windows we use NASM-style .asm files instead.
if host_os == 'windows'
  bcm_sources_asm_used = bcm_sources_nasm
  crypto_sources_asm_used = crypto_sources_nasm
  test_support_sources_asm_used = test_support_sources_nasm
else
  bcm_sources_asm_used = bcm_sources_asm
  crypto_sources_asm_used = crypto_sources_asm
  test_support_sources_asm_used = test_support_sources_asm
endif

# ── Third-party: GoogleTest ──────────────────────────────────────────────────
boringssl_gtest = static_library(
  'boringssl_gtest',
  files(
    'third_party/googletest/googlemock/src/gmock-all.cc',
    'third_party/googletest/googletest/src/gtest-all.cc',
  ),
  include_directories : [
    include_directories(
      'third_party/googletest/googlemock/include',
      'third_party/googletest/googletest/include',
      'third_party/googletest/googlemock',
      'third_party/googletest/googletest',
    ),
  ],
  cpp_args : common_c_cpp_args,
)

boringssl_gtest_dep = declare_dependency(
  link_with : boringssl_gtest,
  include_directories : [
    include_directories(
      'third_party/googletest/googlemock/include',
      'third_party/googletest/googletest/include',
    ),
  ],
)

# ── libcrypto ────────────────────────────────────────────────────────────────
# BCM (fipsmodule) sources are compiled directly into libcrypto.
libcrypto = library(
  'crypto',
  bcm_sources + bcm_sources_asm_used + crypto_sources + crypto_sources_asm_used,
  include_directories : inc,
  c_args   : common_c_cpp_args + impl_define,
  cpp_args : common_c_cpp_args + no_cxx_runtime_flags + impl_define,
  dependencies : thread_dep,
)

libcrypto_dep = declare_dependency(
  link_with         : libcrypto,
  include_directories : inc,
)

# ── libssl ───────────────────────────────────────────────────────────────────
libssl = library(
  'ssl',
  ssl_sources,
  include_directories : inc,
  cpp_args : common_c_cpp_args + impl_define,
  c_args   : common_c_cpp_args + impl_define,
  dependencies : [libcrypto_dep],
)

libssl_dep = declare_dependency(
  link_with         : libssl,
  include_directories : inc,
  dependencies      : [libcrypto_dep],
)

# ── libdecrepit ──────────────────────────────────────────────────────────────
libdecrepit = library(
  'decrepit',
  decrepit_sources,
  include_directories : inc,
  cpp_args : common_c_cpp_args + impl_define,
  c_args   : common_c_cpp_args + impl_define,
  dependencies : [libssl_dep],
)

# ── libpki ───────────────────────────────────────────────────────────────────
pki_extra_args = []
if host_os == 'darwin'
  pki_extra_args = ['-fno-aligned-new']
endif

libpki = library(
  'pki',
  pki_sources,
  include_directories : inc,
  cpp_args : common_c_cpp_args + impl_define + pki_extra_args,
  c_args   : common_c_cpp_args + impl_define,
  dependencies : [libcrypto_dep],
)

libpki_dep = declare_dependency(
  link_with         : libpki,
  include_directories : inc,
  dependencies      : [libcrypto_dep],
)

# ── bssl tool ────────────────────────────────────────────────────────────────
executable(
  'bssl',
  bssl_sources,
  include_directories : inc,
  cpp_args : common_c_cpp_args,
  c_args   : common_c_cpp_args,
  dependencies : [libssl_dep],
)

# ── test_support_lib ─────────────────────────────────────────────────────────
test_support_lib = static_library(
  'test_support_lib',
  test_support_sources + test_support_sources_asm_used,
  include_directories : inc,
  cpp_args : common_c_cpp_args,
  c_args   : common_c_cpp_args,
  dependencies : [libcrypto_dep, boringssl_gtest_dep],
)

test_support_dep = declare_dependency(
  link_with : [test_support_lib],
  dependencies : [boringssl_gtest_dep, libcrypto_dep],
)

# ── Unit tests ───────────────────────────────────────────────────────────────
crypto_test_exe = executable(
  'crypto_test',
  crypto_test_sources,
  include_directories : inc,
  cpp_args : common_c_cpp_args,
  c_args   : common_c_cpp_args,
  dependencies : [libcrypto_dep, test_support_dep],
)
test('crypto_test', crypto_test_exe)

ssl_test_exe = executable(
  'ssl_test',
  ssl_test_sources,
  include_directories : inc,
  cpp_args : common_c_cpp_args,
  c_args   : common_c_cpp_args,
  dependencies : [libssl_dep, test_support_dep],
)
test('ssl_test', ssl_test_exe)

decrepit_test_exe = executable(
  'decrepit_test',
  decrepit_test_sources,
  include_directories : inc,
  cpp_args : common_c_cpp_args,
  c_args   : common_c_cpp_args,
  link_with : libdecrepit,
  dependencies : [libssl_dep, test_support_dep],
)
test('decrepit_test', decrepit_test_exe)

pki_test_exe = executable(
  'pki_test',
  pki_test_sources,
  include_directories : inc,
  cpp_args : common_c_cpp_args + pki_extra_args,
  c_args   : common_c_cpp_args,
  dependencies : [libpki_dep, test_support_dep],
)
test('pki_test', pki_test_exe)

urandom_test_exe = executable(
  'urandom_test',
  urandom_test_sources,
  include_directories : inc,
  cpp_args : common_c_cpp_args,
  c_args   : common_c_cpp_args,
  dependencies : [libcrypto_dep, test_support_dep],
)
test('urandom_test', urandom_test_exe)

# ── Benchmark ────────────────────────────────────────────────────────────────
# google_benchmark is an optional dependency; only build bssl_bench when found.
benchmark_dep = dependency('benchmark', required : false)
if benchmark_dep.found()
  executable(
    'bssl_bench',
    bench_sources,
    include_directories : inc,
    cpp_args : common_c_cpp_args,
    c_args   : common_c_cpp_args,
    dependencies : [libssl_dep, libpki_dep, benchmark_dep],
  )
endif
