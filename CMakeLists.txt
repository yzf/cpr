cmake_minimum_required(VERSION 2.8.7)

if(POLICY CMP0048)
    # cmake warns if loaded from a min-3.0-required parent dir, so silence the warning:
    cmake_policy(SET CMP0048 NEW)
endif()

# Allow use of project folders for IDEs like Visual Studio, so we
# could organize projects into relevant folders: "cpr", "tests" & "external (libraries)".
set_property(GLOBAL PROPERTY USE_FOLDERS ON)
set_property(GLOBAL PROPERTY PREDEFINED_TARGETS_FOLDER "CMake")

project(cpr CXX)

if(NOT ${CMAKE_VERSION} LESS 3.2)
    set(CMAKE_CXX_STANDARD 11)
    set(CMAKE_CXX_STANDARD_REQUIRED ON)
else()
    message(STATUS "Checking compiler flags for C++11 support.")
    # Set C++11 support flags for various compilers
    include(CheckCXXCompilerFlag)
    check_cxx_compiler_flag("-std=c++11" COMPILER_SUPPORTS_CXX11)
    check_cxx_compiler_flag("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
    if(COMPILER_SUPPORTS_CXX11)
        message(STATUS "C++11 is supported.")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
    elseif(COMPILER_SUPPORTS_CXX0X)
        message(STATUS "C++0x is supported.")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
    else()
        message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
    endif()
endif()

set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR}/bin)
set(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR}/lib)

set(CPR_LIBRARIES cpr CACHE INTERNAL "")
set(CPR_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/include CACHE INTERNAL "")

macro(cpr_option OPTION_NAME OPTION_TEXT OPTION_DEFAULT)
    option(${OPTION_NAME} ${OPTION_TEXT} ${OPTION_DEFAULT})
    if(DEFINED ENV{${OPTION_NAME}})
        # Allow setting the option through an environment variable
        set(${OPTION_NAME} $ENV{${OPTION_NAME}})
    endif()
    if(${OPTION_NAME})
        add_definitions(-D${OPTION_NAME})
    endif()
    message(STATUS "  ${OPTION_NAME}: ${${OPTION_NAME}}")
endmacro()

message(STATUS "C++ Requests CMake Options")
message(STATUS "=======================================================")
cpr_option(USE_SYSTEM_CURL
    "If ON, this project will look in the system paths for an installed curl library" ON)
cpr_option(BUILD_CPR_TESTS "Set to ON to build cpr tests." OFF)
cpr_option(GENERATE_COVERAGE "Set to ON to generate coverage reports." OFF)
cpr_option(CPR_CURL_NOSIGNAL "Set to ON to disable use of signals in libcurl." OFF)
cpr_option(USE_SYSTEM_GTEST
    "If ON, this project will look in the system paths for an installed gtest library" OFF)
cpr_option(CMAKE_USE_OPENSSL "Use OpenSSL code. Experimental" ON)
message(STATUS "=======================================================")

if(BUILD_CPR_TESTS)
    enable_testing()
endif()

add_subdirectory(opt)
add_subdirectory(cpr)
if(BUILD_CPR_TESTS)
    add_subdirectory(test)
endif()

set(prefix                  "${CMAKE_INSTALL_PREFIX}")
set(exec_prefix             "\${prefix}")
set(includedir              "\${prefix}/include")
set(libdir                  "${CMAKE_INSTALL_PREFIX}/lib")

configure_file("${CMAKE_CURRENT_SOURCE_DIR}/libcpr.pc.in"
    "${CMAKE_CURRENT_BINARY_DIR}/libcpr.pc" @ONLY)

install(FILES "${CMAKE_CURRENT_BINARY_DIR}/libcpr.pc"
    DESTINATION "${CMAKE_INSTALL_PREFIX}/lib/pkgconfig")

install(DIRECTORY include/cpr/ DESTINATION "${CMAKE_INSTALL_PREFIX}/include/cpr")
