# This CMake config file for secp256k1 project from https://github.com/bitcoin-core/secp256k1
#
# The secp256k1 project has been configured following official docs with following options:
#
# ./configure --disable-shared --disable-tests --disable-coverage --disable-openssl-tests --disable-exhaustive-tests --disable-jni --with-bignum=no --with-field=64bit --with-scalar=64bit --with-asm=no
#
# Build static context:
# make src/ecmult_static_context.h
#
# Copy src/ecmult_static_context.h and src/libsecp256k1-config.h
#
# Copy CFLAGS from Makefile to COMPILE_OPTIONS.

cmake_minimum_required(VERSION 3.4)
project(secp256k1 LANGUAGES C)

set(COMMON_COMPILE_FLAGS 
	ENABLE_MODULE_RECOVERY 
	ENABLE_MODULE_ECDH 
	USE_ECMULT_STATIC_PRECOMPUTATION 
	USE_FIELD_INV_BUILTIN 
	USE_NUM_NONE 
	USE_SCALAR_INV_BUILTIN)
if (MSVC)
	set(COMPILE_FLAGS 
		USE_FIELD_10X26 
		USE_SCALAR_8X32)
	set(COMPILE_OPTIONS "")
else()
	set(COMPILE_FLAGS 
		USE_FIELD_5X52 USE_SCALAR_4X64 
		HAVE_BUILTIN_EXPECT HAVE___INT128)
	set(COMPILE_OPTIONS -O3 -W -std=c89 -pedantic -Wall -Wextra -Wcast-align
	       	-Wnested-externs -Wshadow -Wstrict-prototypes 
		-Wno-unused-function -Wno-long-long -Wno-overlength-strings 
		-fvisibility=hidden)
endif()

add_executable(gen_context src/gen_context.c)
target_include_directories(gen_context PRIVATE ${CMAKE_SOURCE_DIR})

add_custom_target(ecmult_static_context gen_context WORKING_DIRECTORY ${CMAKE_SOURCE_DIR})

add_library(secp256k1 STATIC src/secp256k1.c)
target_compile_definitions(secp256k1 PRIVATE ${COMMON_COMPILE_FLAGS} ${COMPILE_FLAGS})
target_include_directories(secp256k1 PRIVATE ${CMAKE_SOURCE_DIR} ${CMAKE_SOURCE_DIR}/src)
target_compile_options(secp256k1 PRIVATE ${COMPILE_OPTIONS})
add_dependencies(secp256k1 ecmult_static_context)

install(TARGETS secp256k1 ARCHIVE DESTINATION lib)
install(DIRECTORY include/ DESTINATION include)
#cmake_minimum_required(VERSION 3.2)
#
#project(secp256k1 LANGUAGES C)
#
#
#add_library(secp256k1 SHARED ${SOURCES} src/secp256k1.c)
#target_include_directories(secp256k1 PUBLIC . ./src PRIVATE ${GMP_INCLUDE_DIR})
#target_compile_definitions(secp256k1
#	PUBLIC -DUSE_NUM_NONE=1 -DUSE_FIELD_5X52=1 -DUSE_FIELD_INV_BUILTIN=1 -DUSE_SCALAR_4X64=1 -DUSE_SCALAR_INV_BUILTIN=1 -DHAVE___INT128=1 -DVERIFY=1
#	PRIVATE -DENABLE_MODULE_ECDH -DENABLE_MODULE_RECOVERY -DENABLE_MODULE_SCHNORR
#)
#
#add_executable(tests src/tests.c)
#target_link_libraries(tests secp256k1)
#
#add_executable(bench_ecdh src/bench_ecdh.c)
#target_link_libraries(bench_ecdh secp256k1)
#
#add_executable(bench_internal src/bench_internal.c)
#target_link_libraries(bench_internal secp256k1)
#
#add_executable(bench_recover src/bench_recover.c)
#target_link_libraries(bench_recover secp256k1)
#
#add_executable(bench_schnorr_verify src/bench_schnorr_verify.c)
#target_link_libraries(bench_schnorr_verify secp256k1)
#
#add_executable(bench_sign src/bench_sign.c)
#target_link_libraries(bench_sign secp256k1)
#
#add_executable(bench_verify src/bench_verify.c)
#target_link_libraries(bench_verify secp256k1)
