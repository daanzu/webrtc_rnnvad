CMAKE_MINIMUM_REQUIRED(VERSION 3.9)
PROJECT(webrtc_rnnvad C CXX)

SET(CMAKE_CXX_STANDARD 14)
SET(CMAKE_C_STANDARD 11)

set(SRC
  main.cc

  #include/api
  #include/common_audio
  #include/rtc_base
  #include/modules
  common_audio/resampler/push_sinc_resampler.h
  common_audio/resampler/push_sinc_resampler.cc
  common_audio/resampler/sinc_resampler.h
  common_audio/resampler/sinc_resampler.cc
  common_audio/wav_file.h
  common_audio/wav_file.cc
  rtc_base/checks.h
  rtc_base/checks.cc
  rtc_base/logging.h
  rtc_base/logging.cc
  rtc_base/string_utils.h
  rtc_base/string_utils.cc
  rtc_base/time_utils.h
  rtc_base/time_utils.cc
  rtc_base/atomic_ops.h
  rtc_base/critical_section.h
  rtc_base/critical_section.cc
  rtc_base/sanitizer.h
  rtc_base/string_encode.h
  rtc_base/string_encode.cc
  rtc_base/thread_annotations.h
  rtc_base/platform_thread_types.h
  rtc_base/platform_thread_types.cc
  rtc_base/numerics/safe_minmax.h
  rtc_base/system/file_wrapper.h
  rtc_base/system/file_wrapper.cc
  rtc_base/system/unused.h
  rtc_base/numerics/safe_conversions.h
  rtc_base/numerics/safe_conversions_impl.h
  rtc_base/strings/string_builder.h
  rtc_base/strings/string_builder.cc
  common_audio/wav_header.h
  common_audio/wav_header.cc
  common_audio/include/audio_util.h
  common_audio/audio_util.cc
  modules/audio_processing/agc2/rnn_vad/features_extraction.h
  modules/audio_processing/agc2/rnn_vad/features_extraction.cc
  modules/audio_processing/agc2/rnn_vad/common.h
  modules/audio_processing/agc2/rnn_vad/common.cc
  modules/audio_processing/agc2/rnn_vad/rnn.h
  modules/audio_processing/agc2/rnn_vad/rnn.cc
  modules/audio_processing/agc2/biquad_filter.cc
  modules/audio_processing/agc2/biquad_filter.h
  third_party/rnnoise/src/rnn_activations.h
  third_party/rnnoise/src/rnn_vad_weights.h
  third_party/rnnoise/src/rnn_vad_weights.cc
  rtc_base/memory/aligned_malloc.h
  rtc_base/memory/aligned_malloc.cc
  modules/audio_processing/agc2/rnn_vad/pitch_info.h
  modules/audio_processing/agc2/rnn_vad/pitch_search.h
  modules/audio_processing/agc2/rnn_vad/pitch_search.cc
  modules/audio_processing/agc2/rnn_vad/sequence_buffer.h
  modules/audio_processing/agc2/rnn_vad/spectral_features.h
  modules/audio_processing/agc2/rnn_vad/spectral_features.cc
  modules/audio_processing/agc2/rnn_vad/lp_residual.h
  modules/audio_processing/agc2/rnn_vad/lp_residual.cc
  modules/audio_processing/agc2/rnn_vad/auto_correlation.h
  modules/audio_processing/agc2/rnn_vad/auto_correlation.cc
  modules/audio_processing/agc2/rnn_vad/pitch_search_internal.h
  modules/audio_processing/agc2/rnn_vad/pitch_search_internal.cc
  modules/audio_processing/agc2/rnn_vad/spectral_features.h
  modules/audio_processing/agc2/rnn_vad/spectral_features.cc
  modules/audio_processing/agc2/rnn_vad/spectral_features_internal.h
  modules/audio_processing/agc2/rnn_vad/spectral_features_internal.cc
  modules/audio_processing/agc2/rnn_vad/ring_buffer.h
  modules/audio_processing/agc2/rnn_vad/symmetric_matrix_buffer.h
  modules/audio_processing/utility/pffft_wrapper.h
  modules/audio_processing/utility/pffft_wrapper.cc
  third_party/pffft/src/pffft.h
  third_party/pffft/src/pffft.c
  third_party/pffft/src/fftpack.h
  third_party/pffft/src/fftpack.c
)

# Compiler-specific settings
if(MSVC)
  add_definitions(-D_WINSOCKAPI_ -DWEBRTC_WIN -D_USE_MATH_DEFINES -DNOMINMAX)
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /W4 /MP")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4 /MP")
  set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} /O2 /Ob2 /Oi /Ot /GL")
  set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /O2 /Ob2 /Oi /Ot /GL")
  set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} /Od /Zi /RTC1")
  set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /Od /Zi /RTC1")
  set(CMAKE_EXE_LINKER_FLAGS_RELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} /LTCG")
else()
  add_definitions(-DWEBRTC_POSIX -DWEBRTC_LINUX -D_USE_MATH_DEFINES -DNOMINMAX)
  # set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wextra -Wpedantic")
  # set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Wpedantic")
  set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -O3 -march=native -flto")
  set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O3 -march=native -flto")
  set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -O0 -g")
  set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -O0 -g")
  set(CMAKE_EXE_LINKER_FLAGS_RELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} -flto")
endif()

# Enable Link Time Optimization for Release builds
set(CMAKE_INTERPROCEDURAL_OPTIMIZATION_RELEASE TRUE)

INCLUDE_DIRECTORIES(
    ${PROJECT_SOURCE_DIR}
)

set(CMAKE_INSTALL_PREFIX ${CMAKE_BINARY_DIR}/install)

# Abseil
add_subdirectory(abseil-cpp)

# Main executable
add_executable(${CMAKE_PROJECT_NAME} ${SRC})

target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE
  absl::algorithm
  absl::base
  absl::debugging
  absl::flat_hash_map
  absl::flags
  absl::flags_parse
  absl::memory
  absl::meta
  absl::numeric
  absl::random_random
  absl::strings
  absl::synchronization
  absl::time
  absl::utility
)

# # Disable specific warnings for C files
# if(NOT MSVC)
#   set_source_files_properties(
#     ${CMAKE_CURRENT_SOURCE_DIR}/third_party/pffft/src/pffft.c
#     ${CMAKE_CURRENT_SOURCE_DIR}/third_party/pffft/src/fftpack.c
#     PROPERTIES
#     COMPILE_FLAGS "-Wno-unused-parameter -Wno-sign-compare"
#   )
# endif()

# Enable cache for faster rebuilds
set_property(GLOBAL PROPERTY USE_FOLDERS ON)
