project(nheqminer)
cmake_minimum_required(VERSION 3.5)
set(BUILD_ARCH "x86-64")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -fPIC ") # -Wall
## Enable solvers here
#### older slower
option(USE_CPU_TROMP "USE CPU_TROMP" OFF)
option(USE_CUDA_TROMP "USE CUDA_TROMP" OFF)
#### faster
option(USE_CPU_XENONCAT "USE CPU_XENONCAT" OFF)
option(USE_CUDA_DJEZO "USE CUDA_DJEZO" OFF)
option(USE_CPU_VERUSHASH "USE CPU_VERUSHASH" ON)

## Add solvers here
if (USE_CPU_TROMP)
    add_definitions(-DUSE_CPU_TROMP)
    message("-- USE_CPU_TROMP DEFINED")
endif()
if (USE_CPU_VERUSHASH)
    add_definitions(-DUSE_CPU_VERUSHASH)
    message("-- USE_CPU_VERUSHASH DEFINED")
endif()
if (USE_CPU_XENONCAT)
    add_definitions(-DUSE_CPU_XENONCAT)
    message("-- USE_CPU_XENONCAT DEFINED")
endif()
if (USE_CUDA_TROMP)
    add_definitions(-DUSE_CUDA_TROMP)
    message("-- USE_CUDA_TROMP DEFINED")
endif()
if (USE_CUDA_DJEZO)
    add_definitions(-DUSE_CUDA_DJEZO)
    message("-- USE_CUDA_DJEZO DEFINED")
endif()

# optimizations
add_definitions(-O3)

########
# use native cpu features

set_source_files_properties(${nheqminer_SOURCE_DIR}/nheqminer/crypto/verus_clhash.cpp PROPERTIES COMPILE_FLAGS " -m64 -mavx -mpclmul -msse2 -msse3 -mssse3 -msse4 -msse4.1 -msse4.2 -maes  -funroll-loops -fomit-frame-pointer")
set_source_files_properties(${nheqminer_SOURCE_DIR}/nheqminer/crypto/verus_hash.cpp PROPERTIES COMPILE_FLAGS "   -m64 -mavx -mpclmul -msse2 -msse3 -mssse3 -msse4 -msse4.1 -msse4.2 -maes  -funroll-loops -fomit-frame-pointer")
set_source_files_properties(${nheqminer_SOURCE_DIR}/nheqminer/crypto/haraka.c PROPERTIES COMPILE_FLAGS "         -m64 -mavx -mpclmul -msse2 -msse3 -mssse3 -msse4 -msse4.1 -msse4.2 -maes  -funroll-loops -fomit-frame-pointer")

# Common
include_directories(${nheqminer_SOURCE_DIR}/nheqminer)
# MACOS
if(APPLE)
    include_directories(${nheqminer_SOURCE_DIR}/nheqminer/compat)
endif(APPLE)
# BOOST
#find_package(Threads REQUIRED COMPONENTS)
# compile boost staticaly
set(Boost_USE_STATIC_LIBS ON)
set(CMAKE_FIND_LIBRARY_SUFFIXES ".a")
#set(BUILD_SHARED_LIBRARIES OFF)
#set(CMAKE_EXE_LINKER_FLAGS "-static-libgcc -static-libstdc++ -static")
find_package(Boost REQUIRED COMPONENTS system log_setup log date_time filesystem thread)

if (Boost_FOUND)
  # From the official documentation:
  # Add include directories to the build. [...] If the SYSTEM option is given,
  # the compiler will be told the directories are meant as system include
  # directories on some platforms (signalling this setting might achieve effects
  # such as the compiler skipping warnings [...])."
  include_directories (SYSTEM ${Boost_INCLUDE_DIR})

  # From the offical documentation:
  # "Specify directories in which the linker will look for libraries. [...] Note
  # that this command is rarely necessary. Library locations returned by
  # find_package() and find_library() are absolute paths. Pass these absolute
  # library file paths directly to the target_link_libraries() command. CMake
  # will ensure the linker finds them."
  link_directories (${Boost_LIBRARY_DIRS})
else()
    message("Boost_FOUND NOT FOUND")
endif ()

include_directories(${CMAKE_CURRENT_BINARY_DIR}/../)

set(SOURCE_FILES
    # sources
    nheqminer/amount.cpp
    nheqminer/api.cpp
    nheqminer/arith_uint256.cpp
    nheqminer/crypto/sha256.cpp
    nheqminer/crypto/haraka.c
    nheqminer/crypto/haraka_portable.c
    nheqminer/crypto/verus_hash.cpp
    nheqminer/crypto/verus_clhash.cpp
    nheqminer/crypto/verus_clhash_portable.cpp
    nheqminer/json/json_spirit_reader.cpp
    nheqminer/json/json_spirit_value.cpp
    nheqminer/json/json_spirit_writer.cpp
    nheqminer/libstratum/ZcashStratum.cpp
    nheqminer/main.cpp
    nheqminer/primitives/block.cpp
    nheqminer/primitives/nonce.cpp
    nheqminer/speed.cpp
    nheqminer/uint256.cpp
    nheqminer/utilstrencodings.cpp
    # headers
    nheqminer/amount.h
    nheqminer/api.hpp
    nheqminer/arith_uint256.h
    nheqminer/crypto/verus_clhash.h
    nheqminer/crypto/haraka.h
    nheqminer/crypto/haraka_portable.h
    nheqminer/crypto/sha256.h
    nheqminer/crypto/verus_hash.h
    nheqminer/crypto/verus_clhash.h
    nheqminer/hash.h
    nheqminer/json/json_spirit.h
    nheqminer/json/json_spirit_error_position.h
    nheqminer/json/json_spirit_reader.h
    nheqminer/json/json_spirit_reader_template.h
    nheqminer/json/json_spirit_stream_reader.h
    nheqminer/json/json_spirit_utils.h
    nheqminer/json/json_spirit_value.h
    nheqminer/json/json_spirit_writer.h
    nheqminer/json/json_spirit_writer_template.h
    nheqminer/libstratum/StratumClient.cpp
    nheqminer/libstratum/StratumClient.h
    nheqminer/libstratum/ZcashStratum.cpp
    nheqminer/libstratum/ZcashStratum.h
    nheqminer/primitives/block.h
    nheqminer/primitives/nonce.cpp
    nheqminer/primitives/transaction.h
    nheqminer/script/script.h
    nheqminer/serialize.h
    nheqminer/speed.hpp
    nheqminer/streams.h
    nheqminer/support/allocators/zeroafterfree.h
    nheqminer/tinyformat.h
    nheqminer/uint252.h
    nheqminer/uint256.h
    nheqminer/utilstrencodings.h
    nheqminer/version.h
    nheqminer/zcash/JoinSplit.hpp
    nheqminer/zcash/NoteEncryption.hpp
    nheqminer/zcash/Proof.hpp
    nheqminer/zcash/Zcash.h
    nheqminer/SolverStub.h # just a stub

    nheqminer/AvailableSolvers.h
    nheqminer/ISolver.h
    nheqminer/Solver.h
    nheqminer/MinerFactory.h
    nheqminer/MinerFactory.cpp

    # make same path on windows
    #blake shared
    # src
    blake2/blake2bx.cpp
    # headers
    blake2/blake2.h
    blake2/blake2b-load-sse2.h
    blake2/blake2b-load-sse41.h
    blake2/blake2b-round.h
    blake2/blake2-config.h
    blake2/blake2-impl.h
    blake2/blake2-round.h
    )

#set(LIBS ${LIBS} ${Threads_LIBRARIES} ${Boost_LIBRARIES})
set(LIBS ${LIBS} ${Boost_LIBRARIES})

message("-- CXXFLAGS: ${CMAKE_CXX_FLAGS}")
message("-- LIBS: ${LIBS}")

if (USE_CPU_TROMP)
    add_subdirectory(cpu_tromp)
endif()
if (USE_CPU_VERUSHASH)
    add_subdirectory(cpu_verushash)
endif()
if (USE_CPU_XENONCAT)
    add_subdirectory(cpu_xenoncat)
endif()
if (USE_CUDA_TROMP)
    add_subdirectory(cuda_tromp)
endif()
if (USE_CUDA_DJEZO)
    add_subdirectory(cuda_djezo)
endif()

#add_subdirectory(cpu_xenoncat)

ADD_EXECUTABLE(${PROJECT_NAME} ${SOURCE_FILES})

#target_link_libraries(${PROJECT_NAME} ${LIBS} ${CUDA_LIBRARIES} )
target_link_libraries(${PROJECT_NAME} ${CMAKE_THREAD_LIBS_INIT} ${LIBS} )

# link libs
if (USE_CPU_TROMP)
    target_link_libraries(${PROJECT_NAME} cpu_tromp)
endif()
if (USE_CPU_VERUSHASH)
    target_link_libraries(${PROJECT_NAME} cpu_verushash)
endif()
if (USE_CPU_XENONCAT)
    add_library ( xenoncat_avx1 SHARED IMPORTED GLOBAL )
    set_target_properties ( xenoncat_avx1 PROPERTIES IMPORTED_LOCATION "../nheqminer/cpu_xenoncat/asm_linux/equihash_avx1.o" )
    add_library ( xenoncat_avx2 SHARED IMPORTED GLOBAL )
    set_target_properties ( xenoncat_avx2 PROPERTIES IMPORTED_LOCATION "../nheqminer/cpu_xenoncat/asm_linux/equihash_avx2.o" )
    target_link_libraries(${PROJECT_NAME} cpu_xenoncat xenoncat_avx1 xenoncat_avx2)
endif()
if (USE_CUDA_TROMP)
    target_link_libraries(${PROJECT_NAME} cuda_tromp)
endif()
if (USE_CUDA_DJEZO)
    target_link_libraries(${PROJECT_NAME} cuda_djezo)
endif()

