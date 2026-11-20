cmake_minimum_required(VERSION 3.10)
project(ssc VERSION 1.0.0 LANGUAGES C)
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

option(ssc_BUILD_EXAMPLES "Build the command-line tools" OFF)

set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE Release)
endif()

if(MSVC)
  add_compile_options(/W3)
  add_compile_definitions(_CRT_SECURE_NO_WARNINGS)
else()
  add_compile_options(-Wall -Wextra)
endif()

find_library(MATH_LIBRARY m)

add_library(ssc
  src/ssc_encoder.c
  src/ssc_enc_analysis.c
  src/ssc_enc_alloc.c
  src/ssc_enc_bands.c
  src/ssc_enc_ec.c
  src/ssc_enc_energy.c
  src/ssc_enc_mdct.c
  src/ssc_enc_pitch.c
  src/ssc_enc_tables.c
  src/ssc_alloc.c
  src/ssc_bands.c
  src/ssc_cwrs.c
  src/ssc_decoder.c
  src/ssc_ec.c
  src/ssc_energy.c
  src/ssc_synth.c
  src/ssc_tables.c
  src/ssc_twiddle.c)
target_include_directories(ssc PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)
if(MATH_LIBRARY)
  target_link_libraries(ssc PUBLIC ${MATH_LIBRARY})
endif()

if(ssc_BUILD_EXAMPLES)
  add_executable(ssc_enc_tool examples/ssc_enc_tool.c)
  target_link_libraries(ssc_enc_tool ssc)
  add_executable(ssc_roundtrip examples/ssc_roundtrip.c)
  target_link_libraries(ssc_roundtrip ssc)
endif()


install(TARGETS ssc
  EXPORT sscTargets
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(DIRECTORY include/
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(EXPORT sscTargets
  FILE sscTargets.cmake
  NAMESPACE ssc::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/ssc
)

configure_package_config_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/cmake/sscConfig.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/sscConfig.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/ssc
)

write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/sscConfigVersion.cmake
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY SameMajorVersion
)

install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/sscConfig.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/sscConfigVersion.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/ssc
)

configure_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/libssc.pc.in
  ${CMAKE_CURRENT_BINARY_DIR}/libssc.pc
  @ONLY
)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/libssc.pc
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig
)
