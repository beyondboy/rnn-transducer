IF(APPLE)
    cmake_minimum_required(VERSION 3.4)
ELSE()
    cmake_minimum_required(VERSION 2.8)
ENDIF()


project(transducer_release)


IF (NOT APPLE)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -O2")
ENDIF()
IF (APPLE)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -O2")
    add_definitions(-DAPPLE)
ENDIF()
include_directories(include)

option(WITH_OMP "compile transducer with openmp. " OFF)

IF(NOT WITH_OMP)
    add_definitions(-DTRANSDUCER_DISABLE_OMP)
endif()
IF (APPLE)
    EXEC_PROGRAM(uname ARGS -v  OUTPUT_VARIABLE DARWIN_VERSION)
    STRING(REGEX MATCH "[0-9]+" DARWIN_VERSION ${DARWIN_VERSION})
    MESSAGE(STATUS "DARWIN_VERSION=${DARWIN_VERSION}")

    #for el capitain have to use rpath

    IF (DARWIN_VERSION LESS 15)
        set(CMAKE_SKIP_RPATH TRUE)
    ENDIF ()

ELSE()
    #always skip for linux
    set(CMAKE_SKIP_RPATH TRUE)
ENDIF()

MESSAGE(STATUS "building shared library with CPU")
ADD_LIBRARY(transducer SHARED src/transducer_entrypoint.cpp)
add_executable(test_cpu tests/test_cpu.cpp)
TARGET_LINK_LIBRARIES(test_cpu transducer)
SET_TARGET_PROPERTIES(test_cpu PROPERTIES COMPILE_FLAGS "${CMAKE_CXX_FLAGS} --std=c++11")

INSTALL(TARGETS transducer transducer
        RUNTIME DESTINATION "bin"
        LIBRARY DESTINATION "lib"
        ARCHIVE DESTINATION "lib"
       )
INSTALL(FILES include/transducer.h DESTINATION "include")
