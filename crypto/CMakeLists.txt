cmake_minimum_required(VERSION 3.10)

option(TREZOR_CRYPTO_TESTS "Perform Trezor Tests" ON)
#option(TREZOR_CRYPTO_TESTS "Perform Trezor Tests" OFF)

set(CRYPTO_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR})
set(AES_SOURCE_DIR ${CRYPTO_SOURCE_DIR}/aes)
set(CHACHA20_SOURCE_DIR ${CRYPTO_SOURCE_DIR}/chacha20poly1305)
set(ED25519_SOURCE_DIR ${CRYPTO_SOURCE_DIR}/ed25519-donna)
set(TESTS_SOURCE_DIR ${CRYPTO_SOURCE_DIR}/tests)
set(TOOLS_SOURCE_DIR ${CRYPTO_SOURCE_DIR}/tools)

include_directories(${CRYPTO_SOURCE_DIR})
include_directories(${AES_SOURCE_DIR})
include_directories(${CHACHA20_SOURCE_DIR})
include_directories(${ED25519_SOURCE_DIR})
include_directories(${TESTS_SOURCE_DIR})
include_directories(${TOOLS_SOURCE_DIR})

set(TREZOR_SOURCES
  address.c
	base32.c
	base58.c
	bignum.c
	bip32.c
	bip39.c
	blake256.c
	blake2b.c
	blake2s.c
	cash_addr.c
	curves.c
	ecdsa.c
	groestl.c
	hasher.c
	hmac.c
	hmac_drbg.c
	memzero.c
	nem.c
	nist256p1.c
	pbkdf2.c
	rand.c
	rc4.c
	rfc6979.c
	ripemd160.c
	script.c
	secp256k1.c
	segwit_addr.c
	sha2.c
	sha3.c
	shamir.c
	slip39.c
)

set(AES_SOURCES
	${AES_SOURCE_DIR}/aescrypt.c
	${AES_SOURCE_DIR}/aeskey.c
	${AES_SOURCE_DIR}/aes_modes.c
	${AES_SOURCE_DIR}/aestab.c
	${AES_SOURCE_DIR}/aestst.c
)

set(CHACHA20_SOURCES
	${CHACHA20_SOURCE_DIR}/chacha20poly1305.c
	${CHACHA20_SOURCE_DIR}/chacha_merged.c
	${CHACHA20_SOURCE_DIR}/poly1305-donna.c
	${CHACHA20_SOURCE_DIR}/rfc7539.c
)

set(ED25519_SOURCES
	${ED25519_SOURCE_DIR}/curve25519-donna-32bit.c
	${ED25519_SOURCE_DIR}/curve25519-donna-helpers.c
	${ED25519_SOURCE_DIR}/curve25519-donna-scalarmult-base.c
	${ED25519_SOURCE_DIR}/ed25519.c
	${ED25519_SOURCE_DIR}/ed25519-donna-32bit-tables.c
	${ED25519_SOURCE_DIR}/ed25519-donna-basepoint-table.c
	${ED25519_SOURCE_DIR}/ed25519-donna-impl-base.c
	${ED25519_SOURCE_DIR}/ed25519-keccak.c
	${ED25519_SOURCE_DIR}/ed25519-sha3.c
	${ED25519_SOURCE_DIR}/modm-donna-32bit.c
)

# disable sequence point warnings where they are expected
set_source_files_properties(${AES_SOURCES_DIR}/aeskey.c PROPERTIES
                            COMPILE_FLAGS -Wno-sequence-point)

add_library(trezor ${TREZOR_SOURCES})
add_library(aes ${AES_SOURCES})
add_library(chacha20 ${CHACHA20_SOURCES})
add_library(ed25519 ${ED25519_SOURCES})

target_link_libraries(ed25519 trezor)

install(TARGETS trezor DESTINATION lib)
install(TARGETS aes DESTINATION lib)
install(TARGETS chacha20 DESTINATION lib)
install(TARGETS ed25519 DESTINATION lib)

install(FILES ${TREZOR_SOURCES}/*.h DESTINATION include/trezor)
install(FILES ${AES_SOURCES}/*.h DESTINATION include/aes)
install(FILES ${CHACHA20_SOURCES}/*.h DESTINATION include/chacha20)
install(FILES ${ED25519_SOURCES}/*.h DESTINATION include/ed25519)

# Build tests (requires native OpenSSL)
if (TREZOR_CRYPTO_TESTS)

	# test_check (ToDo: find check.h)
	# add_test(NAME test_check COMMAND test_check)
	# add_executable(test_check ${TESTS_SOURCE_DIR}/test_check.c)
	# target_include_directories(test_check PUBLIC ${CRYPTO_SOURCE_DIR})
	# target_link_libraries(test_check trezor rt pthread m crypto)

	# test_openssl
	add_test(NAME test_openssl COMMAND test_openssl 100)
	add_executable(test_openssl ${TESTS_SOURCE_DIR}/test_openssl.c)
	target_link_libraries(test_openssl trezor rt pthread m crypto)

	# test_speed
	add_test(NAME test_speed COMMAND test_speed)
	add_executable(test_speed ${TESTS_SOURCE_DIR}/test_speed.c)
	target_link_libraries(test_speed trezor ed25519 rt pthread m crypto)

endif()
