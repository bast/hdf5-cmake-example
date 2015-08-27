cmake_minimum_required(VERSION 2.8 FATAL_ERROR)

project(hdf5-example)

enable_language(CXX)

# the option is here so that we can plug it into other
# codes where HDF5 may be optional
option(ENABLE_HDF5 "Enable HDF5 support" ON)
if(ENABLE_HDF5)
    find_package(HDF5)
    if(HDF5_FOUND)
        include_directories(${HDF5_INCLUDE_DIR})
        set(_hdf5_libs hdf5 hdf5_cpp)
    else()
        # we did not find it so we fetch it from the web, configure, compile, and link
        include(ExternalProject)
        set(ExternalProjectCMakeArgs
            -DHDF5_BUILD_CPP_LIB=ON
            )
        set(_hdf5_version "1.8.15-patch1")
        file(MAKE_DIRECTORY ${PROJECT_BINARY_DIR}/hdf5_local/src)
        ExternalProject_Add(hdf5_local
            DOWNLOAD_COMMAND curl http://www.hdfgroup.org/ftp/HDF5/current/src/hdf5-${_hdf5_version}.tar.gz | tar xvz -C ${PROJECT_BINARY_DIR}/hdf5_local/src
            CMAKE_ARGS ${ExternalProjectCMakeArgs}
            SOURCE_DIR ${PROJECT_BINARY_DIR}/hdf5_local/src/hdf5-${_hdf5_version}
            BINARY_DIR ${PROJECT_BINARY_DIR}/hdf5_local/build
            STAMP_DIR  ${PROJECT_BINARY_DIR}/hdf5_local/stamp
            TMP_DIR    ${PROJECT_BINARY_DIR}/hdf5_local/tmp
            INSTALL_COMMAND ""
            )
        include_directories(
            ${PROJECT_BINARY_DIR}/hdf5_local/src/hdf5-${_hdf5_version}/src
            ${PROJECT_BINARY_DIR}/hdf5_local/src/hdf5-${_hdf5_version}/c++/src
            ${PROJECT_BINARY_DIR}/hdf5_local/build
            )
        set(_hdf5_libs
            ${PROJECT_BINARY_DIR}/hdf5_local/build/bin/libhdf5_cpp.a
            ${PROJECT_BINARY_DIR}/hdf5_local/build/bin/libhdf5.a
            -ldl
            )
    endif()
endif()

if(ENABLE_HDF5)
    add_executable(
        writedata.x
        src/writedata.cpp
        )
    if(NOT HDF5_FOUND)
        add_dependencies(writedata.x hdf5_local)
    endif()
    target_link_libraries(
        writedata.x
        ${_hdf5_libs}
        )
endif()
