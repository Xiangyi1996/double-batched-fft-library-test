# Copyright (C) 2022 Intel Corporation
# SPDX-License-Identifier: BSD-3-Clause

include(GenerateExportHeader)
include(CommonOptions)
include(GitVersion)
include(InstallLib)

set(SOURCES_PRIVATE_TEST
    math.cpp
    prime_factorization.cpp
)

set(SOURCES
    aot_cache.cpp
    bad_configuration.cpp
    compiler_options.cpp
    configuration.cpp
    device_info.cpp
    generator.cpp
    jit_cache.cpp
    jit_cache_all.cpp
    mixed_radix_fft.cpp
    parser.cpp
    root_of_unity.cpp
    user_module.cpp
    generator/f2fft_gen.cpp
    generator/factor2_slm_fft.cpp
    generator/sbfft_gen.cpp
    generator/small_batch_fft.cpp
    generator/snippet.cpp
    generator/tensor_accessor.cpp
    generator/utility.cpp
)
set(PUBLIC_HEADERS
    aot_cache.hpp
    bad_configuration.hpp
    device_info.hpp
    configuration.hpp
    jit_cache.hpp
    jit_cache_all.hpp
    generator.hpp
    module_format.hpp
    parser.hpp
    plan.hpp
    tensor_indexer.hpp
    shared_handle.hpp
    user_module.hpp
    detail/cast.hpp
    detail/compiler_options.hpp
    detail/generator_impl.hpp
    detail/plan_impl.hpp
)
list(TRANSFORM PUBLIC_HEADERS PREPEND "${PROJECT_SOURCE_DIR}/include/bbfft/")

add_library(bbfft-private-test OBJECT ${SOURCES_PRIVATE_TEST})
target_include_directories(bbfft-private-test PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>"
)
set_property(TARGET bbfft-private-test PROPERTY POSITION_INDEPENDENT_CODE ON)

add_library(bbfft-base ${SOURCES} $<TARGET_OBJECTS:bbfft-private-test>)
add_library(bbfft::bbfft-base ALIAS bbfft-base)
set_common_options(bbfft-base)
target_link_libraries(bbfft-base PRIVATE clir::clir)
set(bbfft_export_header "${PROJECT_BINARY_DIR}/include/bbfft/export.hpp")
generate_export_header(bbfft-base BASE_NAME BBFFT
    EXPORT_FILE_NAME "${bbfft_export_header}")
target_compile_definitions(bbfft-base PUBLIC
    "$<$<NOT:$<BOOL:${BUILD_SHARED_LIBS}>>:BBFFT_STATIC_DEFINE>")
target_include_directories(bbfft-base PUBLIC
    "$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>"
    "$<BUILD_INTERFACE:${PROJECT_BINARY_DIR}/include>"
)
target_include_directories(bbfft-base PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}"
                                              "${CMAKE_CURRENT_SOURCE_DIR}/../common")


# Generate version header
set(bbfft_version_header_in "${PROJECT_SOURCE_DIR}/include/bbfft/version.hpp.in")
set(bbfft_version_header "${PROJECT_BINARY_DIR}/include/bbfft/version.hpp")
git_version()
configure_file(${bbfft_version_header_in} ${bbfft_version_header})

target_sources(bbfft-base PUBLIC FILE_SET HEADERS
    BASE_DIRS ${PROJECT_SOURCE_DIR}/include
              ${PROJECT_BINARY_DIR}/include
    FILES ${PUBLIC_HEADERS}
          "${bbfft_export_header}"
          "${bbfft_version_header}")
install_lib(bbfft-base bbfft)
