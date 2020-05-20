cmake_minimum_required(VERSION 3.5)
project(pmc VERSION 1.0.0)

set(PMC_SOURCE_FILES
        ${CMAKE_CURRENT_SOURCE_DIR}/pmc_heu.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/pmc_maxclique.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/pmcx_maxclique.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/pmcx_maxclique_basic.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/pmc_cores.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/pmc_utils.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/pmc_graph.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/pmc_clique_utils.cpp
        )

add_library(pmc SHARED ${PMC_SOURCE_FILES})

target_include_directories(pmc PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/include>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
        )

add_executable(pmc_main pmc_driver.cpp)
target_link_libraries(pmc_main pmc)


# NOTE(tonioteran) Hack to make OpenMP work for our old version of CMake.
# https://cliutils.gitlab.io/modern-cmake/chapters/packages/OpenMP.html
find_package(OpenMP REQUIRED)
# For CMake < 3.9, we need to make the target ourselves.
if(NOT TARGET OpenMP::OpenMP_CXX)
  find_package(Threads REQUIRED)
  add_library(OpenMP::OpenMP_CXX IMPORTED INTERFACE)
  set_property(TARGET OpenMP::OpenMP_CXX
    PROPERTY INTERFACE_COMPILE_OPTIONS ${OpenMP_CXX_FLAGS})
  # Only works if the same flag is passed to the linker; use CMake 3.9+ otherwise (Intel, AppleClang)
  set_property(TARGET OpenMP::OpenMP_CXX
    PROPERTY INTERFACE_LINK_LIBRARIES ${OpenMP_CXX_FLAGS} Threads::Threads)
endif()
target_link_libraries(pmc OpenMP::OpenMP_CXX)
target_link_libraries(pmc_main OpenMP::OpenMP_CXX)

# Installation
include(GNUInstallDirs)

install(TARGETS pmc
        EXPORT pmc-export
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        )

install(EXPORT pmc-export
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/pmc
        NAMESPACE pmc::
        FILE pmcTargets.cmake
        )

install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

install(FILES cmake/pmcConfig.cmake
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/pmc
        )
