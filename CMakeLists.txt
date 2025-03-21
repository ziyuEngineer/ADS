cmake_minimum_required(VERSION 3.13.0)

project(ads VERSION 0.0.20)

find_package(Threads)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)


if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  set(COMPILE_OPTIONS
    -pedantic
    -Wall
    -Wextra
  )
  add_definitions(-D_GNU_SOURCE)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
  # Compiler flags and definitions for Visual Studio come here
  add_compile_definitions(NOMINMAX)
  if (BUILD_SHARED_LIBS)
     message(FATAL_ERROR "Building shared library is not supported for MSVC")
  endif ()
endif()

add_definitions(-DCONFIG_DEFAULT_LOGLEVEL=1)

add_subdirectory(AdsLib)
add_subdirectory(AdsTool)
add_subdirectory(AdsLibTest)
add_subdirectory(example)


# Install library
include(GenerateExportHeader)

set_property(TARGET ads PROPERTY VERSION ${PROJECT_VERSION})
set_property(TARGET ads PROPERTY SOVERSION 3)

set(additional_export_targets "")
if (TARGET ads_twincat)
  set(additional_export_targets "ads_twincat")
  set_property(TARGET ads_twincat PROPERTY VERSION ${PROJECT_VERSION})
endif ()

# export library (either static or shared depending on BUILD_SHARED_LIBS)
install(TARGETS ads AdsTool ${additional_export_targets}
  EXPORT adsTargets
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
  )

if (win32 AND CMAKE_BUILD_TYPE STREQUAL "Debug")
  install(FILES $<TARGET_PDB_FILE: ads>
          DESTINATION "${CMAKE_INSTALL_BINDIR}"
          OPTIONAL)
  if (additional_export_targets)
    install(FILES $<TARGET_PDB_FILE: ${additional_export_targets}>
            DESTINATION "${CMAKE_INSTALL_BINDIR}"
            OPTIONAL)
  endif()
endif()

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/AdsLib/ DESTINATION include/ads/
  FILES_MATCHING PATTERN "*.hpp" PATTERN "*.h")


set(cmake_configfile_install lib/cmake/ads)
set(target_install_dest_name "${cmake_configfile_install}/adsTargets.cmake")

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/adsConfigVersion.cmake"
  VERSION ${ADSLIB_VERSION}
  COMPATIBILITY AnyNewerVersion
)

configure_package_config_file("${CMAKE_CURRENT_SOURCE_DIR}/cmake/Config.cmake.in"
                "${CMAKE_CURRENT_BINARY_DIR}/cmake/adsConfig.cmake"
                INSTALL_DESTINATION "${cmake_configfile_install}"
                PATH_VARS target_install_dest_name)

install(EXPORT adsTargets
    FILE adsTargets.cmake
    NAMESPACE ads::
    DESTINATION "${cmake_configfile_install}"
    EXPORT_LINK_INTERFACE_LIBRARIES)

install(FILES "${CMAKE_CURRENT_BINARY_DIR}/cmake/adsConfig.cmake"
      "${CMAKE_CURRENT_BINARY_DIR}/adsConfigVersion.cmake"
    DESTINATION "${cmake_configfile_install}")
if (USE_TWINCAT_ROUTER) 
  install(FILES cmake/FindTwinCAT.cmake DESTINATION "${cmake_configfile_install}")
endif()

include(cmake/Package.cmake)
#install dpkg-dev to build debian package
#install nsis to build windows package
configure_packaging()
