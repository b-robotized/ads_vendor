cmake_minimum_required(VERSION 3.13)

project(ads_vendor)

find_package(ament_cmake REQUIRED)

include(ExternalProject)


set(ADS_EXTERNAL_PROJECT_NAME ads_external_project)
set(ADS_GIT_REPOSITORY "https://github.com/Beckhoff/ADS.git")
set(ADS_GIT_TAG "e85ef696447a77039a4c3710d7573c8df32c0d58")

set(EXTERNAL_PROJECT_PREFIX ${CMAKE_CURRENT_BINARY_DIR}/ads_external_build)
set(EXTERNAL_PROJECT_SOURCE_DIR ${EXTERNAL_PROJECT_PREFIX}/src/Ads)
set(EXTERNAL_PROJECT_BINARY_DIR ${EXTERNAL_PROJECT_PREFIX}/build/Ads)

# ADS library should install its files (libraries, headers) into ads_vendor package install space.
set(EXTERNAL_PROJECT_INSTALL_DIR ${CMAKE_INSTALL_PREFIX})

if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message(STATUS "ads_vendor: No CMAKE_BUILD_TYPE specified, defaulting to 'RelWithDebInfo'.")
  set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING "Choose the type of build." FORCE)
endif()
message(STATUS "ads_vendor: CMAKE_BUILD_TYPE for ADS library will be: ${CMAKE_BUILD_TYPE}")

ExternalProject_Add(${ADS_EXTERNAL_PROJECT_NAME}
    # download step
    GIT_REPOSITORY    ${ADS_GIT_REPOSITORY} 
    GIT_TAG           ${ADS_GIT_TAG}        

    # directiories config
    PREFIX            ${EXTERNAL_PROJECT_PREFIX}
    SOURCE_DIR        ${EXTERNAL_PROJECT_SOURCE_DIR} # passed as <SOURCE_DIR> to commands
    BINARY_DIR        ${EXTERNAL_PROJECT_BINARY_DIR} # passed as <BINARY_DIR> to commands
    INSTALL_DIR       ${EXTERNAL_PROJECT_INSTALL_DIR} # passed as <INSTALL_DIR> to commands

    # after initial download, don't re-fetch source on subsequent CMake runs
    UPDATE_COMMAND    ""

    # configure step
    CONFIGURE_COMMAND ${CMAKE_COMMAND} -S <SOURCE_DIR> -B <BINARY_DIR>
                      -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>
                      -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
                      -DBUILD_SHARED_LIBS=ON
                      # here we can pass in additional command configs

    # build step
    BUILD_COMMAND     ${CMAKE_COMMAND} --build <BINARY_DIR> --config ${CMAKE_BUILD_TYPE}
                      -j${CMAKE_BUILD_PARALLEL_LEVEL}

    # install step
    INSTALL_COMMAND   ${CMAKE_COMMAND} --install <BINARY_DIR> --config ${CMAKE_BUILD_TYPE}

    # list files that we require to be installed, among others
    BUILD_BYPRODUCTS  ${EXTERNAL_PROJECT_INSTALL_DIR}/lib/libads.so
                      ${EXTERNAL_PROJECT_INSTALL_DIR}/include/ads/AdsLib.h 
                      ${EXTERNAL_PROJECT_INSTALL_DIR}/include/ads/AdsDef.h  
                      # You can list other critical files

    # logging
    LOG_CONFIGURE     YES
    LOG_BUILD         YES
    LOG_INSTALL       YES
)

# export hedaers in ads_vendor/include
ament_export_include_directories(include)

# if a package depends on ads_vendor, it also has ads library available
ament_export_libraries(ads)

ament_package()