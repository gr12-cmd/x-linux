cmake_minimum_required(VERSION 3.15)
project(xfawac CXX)

include(CTest)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# LLVM/LLD 查找：让 CMake 自己找，或通过命令行指定 LLVM_DIR
find_package(LLVM REQUIRED CONFIG)
find_package(LLD REQUIRED CONFIG)

message(STATUS "Found LLVM ${LLVM_PACKAGE_VERSION}")
message(STATUS "Using LLVMConfig.cmake in: ${LLVM_DIR}")

include_directories(
    ${CMAKE_SOURCE_DIR}/src/include
    ${LLVM_INCLUDE_DIRS}
)

separate_arguments(LLVM_DEFINITIONS_LIST NATIVE_COMMAND ${LLVM_DEFINITIONS})
add_definitions(${LLVM_DEFINITIONS_LIST})

llvm_map_components_to_libnames(llvm_libs support core X86Info X86AsmParser X86CodeGen Analysis Passes TransformUtils ScalarOpts)

set(SOURCES
    src/main.cpp
    src/lexer/lexer.cpp
    src/parser/parser.cpp
    src/llvm/xfawa_llvm_codegen.cpp
    src/config/config.cpp
    src/xfawa_types.cpp
    src/include/xfawa_transpiler.cpp
)

add_library(xfawa_runtime STATIC
    src/runtime/bsod_overlay.cpp
)
target_include_directories(xfawa_runtime PRIVATE
    ${CMAKE_SOURCE_DIR}/src/include
    ${CMAKE_SOURCE_DIR}/src/runtime
)

set_target_properties(xfawa_runtime PROPERTIES
    OUTPUT_NAME "xfawa_runtime"
)

add_executable(xfawac ${SOURCES})

target_link_libraries(xfawac
    ${llvm_libs}
    xfawa_runtime
    lldCOFF
    pthread
    dl
    m
)