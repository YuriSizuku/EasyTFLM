cmake_minimum_required(VERSION 3.5)
project(tflm)

# tested tflite-micro version, 240120
# https://github.com/tensorflow/tflite-micro/tree/324ae1eaba98c8aeb440072371efea481c42ace3

# fatal error: 'tensorflow/lite/array.h' file not found, if not static
option(USE_STATIC_MEMORY "" ON)
message(USE_STATIC_MEMORY=${USE_STATIC_MEMORY})
message(CMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE})
set(tflm_src_dir ${CMAKE_CURRENT_SOURCE_DIR}/src)
set(tflm_tflite_dir "${tflm_src_dir}/tensorflow/lite")
set(tflm_signal_dir "${tflm_src_dir}/signal")
set(tflm_micro_dir "${tflm_tflite_dir}/micro")
set(tflm_frontend_dir "${tflm_tflite_dir}/experimental/microfrontend/lib")
set(tflm_kernels_dir "${tflm_micro_dir}/kernels")

file(GLOB tflm_micro_src
    "${tflm_micro_dir}/*.cc"
    "${tflm_micro_dir}/*.c"
)

file(GLOB tflm_frontend_src
    "${tflm_frontend_dir}/*.c"
    "${tflm_frontend_dir}/*.cc"
)

file(GLOB tflm_bridge_src
    "${tflm_micro_dir}/tflite_bridge/*.c"
    "${tflm_micro_dir}/tflite_bridge/*.cc"
)

file(GLOB tflm_kernels_src
    "${tflm_kernels_dir}/*.c"
    "${tflm_kernels_dir}/*.cc"
)

file(GLOB tflm_signalkernels_src
    "${tflm_signal_dir}/micro/kernels/*.c"
    "${tflm_signal_dir}/micro/kernels/*.cc"
)

file(GLOB tflm_signal_src
    "${tflm_signal_dir}/src/*.c"
    "${tflm_signal_dir}/src/*.cc"
)

set(tflm_signalall_src
    "${tflm_signalkernels_src}"
    "${tflm_signal_src}"
    "${tflm_signal_dir}/src/kiss_fft_wrappers/kiss_fft_float.cc"
    "${tflm_signal_dir}/src/kiss_fft_wrappers/kiss_fft_int16.cc"
    "${tflm_signal_dir}/src/kiss_fft_wrappers/kiss_fft_int32.cc"
)

set(tflm_all_src
    "${tflm_micro_src}"
    "${tflm_kernels_src}"
    "${tflm_bridge_src}"
    "${esp_nn_kernels}"
    "${tflm_frontend_src}"
    "${tflm_signalall_src}"
    "${tflm_tflite_dir}/kernels/kernel_util.cc"
    "${tflm_tflite_dir}/micro/memory_planner/greedy_memory_planner.cc"
    "${tflm_tflite_dir}/micro/memory_planner/linear_memory_planner.cc"
    "${tflm_tflite_dir}/micro/arena_allocator/non_persistent_arena_buffer_allocator.cc"
    "${tflm_tflite_dir}/micro/arena_allocator/persistent_arena_buffer_allocator.cc"
    "${tflm_tflite_dir}/micro/arena_allocator/recording_single_arena_buffer_allocator.cc"
    "${tflm_tflite_dir}/micro/arena_allocator/single_arena_buffer_allocator.cc"
    "${tflm_tflite_dir}/core/c/common.cc"
    "${tflm_tflite_dir}/core/api/error_reporter.cc"
    "${tflm_tflite_dir}/core/api/flatbuffer_conversions.cc"
    "${tflm_tflite_dir}/core/api/tensor_utils.cc"
    "${tflm_tflite_dir}/kernels/internal/common.cc"
    "${tflm_tflite_dir}/kernels/internal/quantization_util.cc"
    "${tflm_tflite_dir}/kernels/internal/portable_tensor_utils.cc"
    "${tflm_tflite_dir}/kernels/internal/tensor_utils.cc"
    "${tflm_tflite_dir}/kernels/internal/tensor_ctypes.cc"
    "${tflm_tflite_dir}/kernels/internal/reference/portable_tensor_utils.cc"
    "${tflm_tflite_dir}/kernels/internal/reference/comparisons.cc"
    "${tflm_tflite_dir}/schema/schema_utils.cc"
)
set(tflm_all_include     
    ${tflm_src_dir}
    ${tflm_src_dir}/third_party/flatbuffers/include
    ${tflm_src_dir}/third_party/gemmlowp
    ${tflm_src_dir}/third_party/kissfft
    ${tflm_src_dir}/third_party/ruy
    CACHE INTERNAL ""
)

if(MSVC)
    list(REMOVE_ITEM tflm_all_src ${tflm_tflite_dir}/micro/test_helpers.cc)
else()
    set(tflm_c_flags ${tflm_c_flags} -DTF_LITE_DISABLE_X86_NEON
        -Wstrict-aliasing -Wno-unused-parameter -Wno-null-dereference
        -Wno-unused-but-set-variable -Wno-unused-function -Wno-shadow
        -Wsign-compare -Wdouble-promotion -Wswitch 
        -Wmissing-field-initializers -ffunction-sections -fdata-sections
        -fno-unwind-tables -fmessage-length=0
    )
    set(tflm_cxx_flags ${tflm_c_flags} 
        -fno-rtti -fno-exceptions -fno-threadsafe-statics
        -Werror -Wno-return-type -Wno-strict-aliasing
    )
endif()

if (USE_STATIC_MEMORY)
    set(tflm_c_flags ${tflm_c_flags} -DTF_LITE_STATIC_MEMORY)
    set(CMAKE_CXX_STANDARD 11)
else()
    list(APPEND tflm_all_src "${tflm_tflite_dir}/array.cc")
    set(CMAKE_CXX_STANDARD 17)
endif()

if(CMAKE_BUILD_TYPE MATCHES "Debug")
    set(tflm_c_flags -g ${tflm_c_flags})
else()
    set(tflm_c_flags -O3 ${tflm_c_flags})
endif()

if (WIN32)
    set(CMAKE_CXX_STANDARD 17)
endif()

add_library(${PROJECT_NAME} STATIC ${tflm_all_src})
target_include_directories(${PROJECT_NAME} PRIVATE ${tflm_all_include})   
target_compile_options(${PROJECT_NAME} PRIVATE 
    ${tflm_c_flags} 
)
target_compile_options(${PROJECT_NAME} PRIVATE 
    $<$<COMPILE_LANGUAGE:CXX>: ${tflm_cxx_flags}>
)