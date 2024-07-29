# cmake build file for squish
# by Stefan Roettger (snroettg@gmail.com)
# updated by Simon Brown (si@sjbrown.co.uk)

# features:
#   uses -fopenmp when available
#    use BUILD_SQUISH_WITH_OPENMP to override
#   Xcode: builds universal binaries, uses SSE2 on i386 and Altivec on ppc
#   Unix and VS: SSE2 support is enabled by default
#    use BUILD_SQUISH_WITH_SSE2 and BUILD_SQUISH_WITH_ALTIVEC to override

cmake_minimum_required(VERSION 3.28, FATAL_ERROR)

project(squish LANGUAGES CXX)

option(BUILD_SQUISH_WITH_OPENMP "Build with OpenMP." OFF)

option(BUILD_SQUISH_WITH_SSE2 "Build with SSE2." OFF)
option(BUILD_SQUISH_WITH_ALTIVEC "Build with Altivec." OFF)

option(SQUISH_STATIC "Build static library." ON)

option(BUILD_SQUISH_EXTRA "Build extra source code." OFF)

if(SQUISH_STATIC)
    set(SQUISH_BUILD_TYPE STATIC)
    set(SQUISH_LINK_TYPE ARCHIVE)
else()
    set(SQUISH_BUILD_TYPE SHARED)
    set(SQUISH_LINK_TYPE LIBRARY)
endif()

set(SQUISH_HDRS
    squish.h
    )

set(SQUISH_SRCS
    alpha.cpp
    alpha.h
    clusterfit.cpp
    clusterfit.h
    colourblock.cpp
    colourblock.h
    colourfit.cpp
    colourfit.h
    colourset.cpp
    colourset.h
    maths.cpp
    maths.h
    rangefit.cpp
    rangefit.h
    simd.h
    simd_float.h
    simd_sse.h
    simd_ve.h
    singlecolourfit.cpp
    singlecolourfit.h
    singlecolourlookup.inl
    squish.cpp
    )

add_library(squish ${SQUISH_BUILD_TYPE} ${SQUISH_SRCS} ${SQUISH_HDRS})

target_include_directories(squish PUBLIC ${PROJECT_SOURCE_DIR})

target_link_libraries(squish PRIVATE -lstdc++ -lm)

if(BUILD_SQUISH_WITH_OPENMP)
    find_package(Threads REQUIRED)
    find_package(OpenMP REQUIRED)
    target_link_libraries(squish PRIVATE Threads::Threads)
    target_link_libraries(squish PRIVATE OpenMP::OpenMP_CXX)
    target_compile_definitions(squish PUBLIC "SQUISH_USE_OPENMP")
endif()

if(BUILD_SQUISH_WITH_SSE2)
    target_compile_definitions(squish PUBLIC "SQUISH_USE_SSE=2")
    if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang|AppleClang")
        target_compile_options(squish PRIVATE -msse2)
    endif()
endif()

if(BUILD_SQUISH_WITH_ALTIVEC)
    target_compile_definitions(squish PUBLIC "SQUISH_USE_ALTIVEC=1")
    if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang|AppleClang")
        target_compile_options(squish PRIVATE -maltivec)
    endif()
endif()

set_target_properties(
    squish PROPERTIES
    PUBLIC_HEADER "${SQUISH_HDRS}"
    VERSION 0.0
    SOVERSION 0.0
    DEBUG_POSTFIX "d"
    )

if(BUILD_SQUISH_EXTRA)
    set(SQUISHTEST_SRCS extra/squishtest.cpp)

    add_executable(squishtest ${SQUISHTEST_SRCS})
    set_target_properties(squishtest PROPERTIES DEBUG_POSTFIX "d")
    target_link_libraries(squishtest squish)

    set(SQUISHPNG_SRCS extra/squishpng.cpp)

    find_package(PNG)

    if(PNG_FOUND)
        set(CMAKE_PLATFORM_IMPLICIT_INCLUDE_DIRECTORIES)
        add_executable(squishpng ${SQUISHPNG_SRCS})
        target_include_directories(squishpng PRIVATE ${PNG_INCLUDE_DIR})
        set_target_properties(squishpng PROPERTIES DEBUG_POSTFIX "d")
        target_link_libraries(squishpng PRIVATE squish PNG::PNG)
    endif()
endif(BUILD_SQUISH_EXTRA)

install(
    TARGETS squish
    ${SQUISH_LINK_TYPE} DESTINATION lib
    RUNTIME DESTINATION bin
    PUBLIC_HEADER DESTINATION include
    )
