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
set(EXTRA_LLVM_FLAGS
"-mllvm --polly"
"-mllvm --polly-run-inliner"
"-mllvm --polly-invariant-load-hoisting"
"-mllvm --polly-run-dce"
"-mllvm --polly-vectorizer=stripmine"
"-mllvm --enable-loopinterchange"
"-mllvm --enable-partial-inlining"
"-mllvm --disable-mr-partial-inlining"
"-mllvm --enable-loop-flatten"
"-mllvm --enable-jump-table-to-switch"
"-mllvm --enable-dfa-jump-thread"
"-mllvm --enable-knowledge-retention"
"-mllvm --extra-vectorizer-passes"
"-mllvm --hot-cold-split"
"-mllvm --regalloc-enable-advisor=release"
"-mllvm --enable-local-reassign=true"
"-mllvm --regalloc-eviction-max-interference-cutoff=20"
)

foreach(FLAG IN LISTS EXTRA_LLVM_FLAGS)
    add_compile_options("SHELL:${FLAG}")
endforeach()
foreach(FLAG IN LISTS EXTRA_LLVM_FLAGS)
    string(REPLACE " " "," LINKER_FLAG "${FLAG}")
    add_link_options("-Wl,${LINKER_FLAG}")
endforeach()

add_compile_options("-funsafe-math-optimizations")
add_link_options("-funsafe-math-optimizations")

add_link_options("-Wl,-O3,--relax,--lto-O3,-S")
endif()

include(CheckIPOSupported)
check_ipo_supported(RESULT CMAKE_IPO_AVAILABLE)
if (CMAKE_IPO_AVAILABLE)
    set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)
endif()

add_library(${MODULE_NAME} SHARED nohello.cpp)
target_link_libraries(${MODULE_NAME} cxx log)

add_subdirectory(external)
