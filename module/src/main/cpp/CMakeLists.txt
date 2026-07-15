cmake_minimum_required(VERSION 3.22.1)
project(sample)

if (CMAKE_BUILD_TYPE STREQUAL "Debug")
    add_definitions(-DDEBUG_BUILD)
    set(CXX_FLAGS "${CXX_FLAGS} -O0 -g -fno-exceptions -fno-rtti")
    set(LINKER_FLAGS "-g -rdynamic -ffixed-x18 -Wl,--hash-style=both -Wl,--export-dynamic")
else ()
    set(CXX_FLAGS "${CXX_FLAGS} -fno-exceptions -fno-rtti -fvisibility=hidden -fvisibility-inlines-hidden")
    set(LINKER_FLAGS "-ffixed-x18 -Wl,--hash-style=both")
endif ()

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${CXX_FLAGS}")

set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} ${LINKER_FLAGS}")
set(CMAKE_MODULE_LINKER_FLAGS "${CMAKE_MODULE_LINKER_FLAGS} ${LINKER_FLAGS}")

# the check is not needed for androidndk
if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
add_compile_options("SHELL:-mllvm --polly")
add_compile_options("SHELL:-mllvm --polly-run-inliner")
add_compile_options("SHELL:-mllvm --polly-invariant-load-hoisting")
add_compile_options("SHELL:-mllvm --polly-run-dce")
add_compile_options("SHELL:-mllvm --polly-vectorizer=stripmine")

add_compile_options("SHELL:-mllvm --enable-loopinterchange")
add_compile_options("SHELL:-mllvm --enable-partial-inlining")
add_compile_options("SHELL:-mllvm --enable-loop-flatten")
add_compile_options("SHELL:-mllvm --enable-jump-table-to-switch")
add_compile_options("SHELL:-mllvm --enable-dfa-jump-thread")
add_compile_options("SHELL:-mllvm --enable-knowledge-retention")
add_compile_options("SHELL:-mllvm --extra-vectorizer-passes=true")
add_compile_options("-funsafe-math-optimizations")
add_compile_options("SHELL:-mllvm --hot-cold-split")
endif()

include(CheckIPOSupported)
check_ipo_supported(RESULT CMAKE_IPO_AVAILABLE)
if (CMAKE_IPO_AVAILABLE)
    set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)
endif()

add_library(${MODULE_NAME} SHARED nohello.cpp)
target_link_libraries(${MODULE_NAME} cxx log)

add_subdirectory(external)
