#--------------------------------------------------------------------
# List of available command-line options:
#
#   -DLOCAL_INSTALL:BOOL=<ON or OFF>
#    If set to on, install the library in the folder 
#    BUILD_DIR/install
#    
#   -DCMAKE_INSTALL_PREFIX:PATH=<RGESolver installation directory>
#    This variable cannot be modified 
#    when -DLOCAL INSTALL ALL=ON is set.
#   
#   -DDEBUG_MODE:BOOL=<ON or OFF>
#    Prints on-screen cache and normal variables
#
#   -DBOOST_INCLUDE_DIR:PATH=<include path>/boost/
#    Path to boost headers
#
#   -DGSL_CONFIG_DIR:PATH=<gsl-config directory>
#    Path to gsl-config
#
#--------------------------------------------------------------------

cmake_minimum_required(VERSION 3.1)
#set(LIBTYPE OBJECT)




project(RGESolver VERSION 1.0 DESCRIPTION "C++ library to solve RGEs in the context of the SMEFT")
#add_library(RGESolver RGESolver.cc)

#set(LIBTYPE OBJECT)



# default boolean flags
option(LOCAL_INSTALL "Local installation" OFF)
option(DEBUG_MODE "Debug mode" OFF)



#Local installation option
if(LOCAL_INSTALL)
  set(CMAKE_INSTALL_PREFIX ${CMAKE_CURRENT_BINARY_DIR}/install
      CACHE PATH "RGESolver installation directory" FORCE)
endif()


#User provided RGESolver installation path
if(NOT IS_ABSOLUTE ${CMAKE_INSTALL_PREFIX})
   set(CMAKE_INSTALL_PREFIX ${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_INSTALL_PREFIX}
       CACHE PATH "RGESolver installation directory" FORCE)
endif()
string(REGEX REPLACE "^~" "$ENV{HOME}" TMP_DIR ${CMAKE_INSTALL_PREFIX})
set(CMAKE_INSTALL_PREFIX ${TMP_DIR} CACHE PATH "RGESolver installation directory" FORCE)
unset(TMP_DIR)

#Printing the installation directory
message(STATUS "Installation directory : ${CMAKE_INSTALL_PREFIX}$")


##########  BOOST  ##########

# find_package(Boost) is not used.

set(BOOST_ERROR_MESSAGE "Boost headers could not be found. Please specify the Boost include path with ${RedBold}-DBOOST_INCLUDE_DIR=<include path>/boost/${ColReset}.")

if(NOT BOOST_INCLUDE_DIR)
  find_path(BOOST_INCLUDE_DIR "boost/tokenizer.hpp" HINTS ENV C_INCLUDE_PATH ENV CPLUS_INCLUDE_PATH DOC "Boost include path")
  if(BOOST_INCLUDE_DIR)
    set(BOOST_INCLUDE_DIR "${BOOST_INCLUDE_DIR}/boost/" CACHE PATH "Boost include path" FORCE)
  else()
    message(FATAL_ERROR "${BOOST_ERROR_MESSAGE}")
  endif()
else()
  set(BOOST_INCLUDE_DIR ${BOOST_INCLUDE_DIR} CACHE PATH "Boost include path" FORCE)
  find_path(TMP_BOOST_INC "tokenizer.hpp" PATHS ${BOOST_INCLUDE_DIR} NO_DEFAULT_PATH)
  if(NOT TMP_BOOST_INC)
    unset(TMP_BOOST_INC CACHE)
    message(FATAL_ERROR "${BOOST_ERROR_MESSAGE}")
  else()
    unset(TMP_BOOST_INC CACHE)
  endif()
endif()

message(STATUS "Found Boost headers in ${BOOST_INCLUDE_DIR}")
string(REGEX REPLACE "/boost$" "" BOOST_INC ${BOOST_INCLUDE_DIR})




##########  GSL  ##########

if(NOT GSL_CONFIG_DIR)
  find_program(GSL_CONFIG gsl-config)
  if(GSL_CONFIG)
    string(REGEX REPLACE "/gsl-config$" "" TMP_PATH ${GSL_CONFIG})
    set(GSL_CONFIG_DIR ${TMP_PATH} CACHE PATH "Path to gsl-config")
    unset(TMP_PATH)
  endif()
else()
  set(GSL_CONFIG_DIR ${GSL_CONFIG_DIR} CACHE PATH "Path to gsl-config" FORCE)
  find_program(GSL_CONFIG gsl-config PATHS ${GSL_CONFIG_DIR} NO_DEFAULT_PATH)
endif()
if(NOT GSL_CONFIG)
  unset(GSL_CONFIG CACHE)
  message(FATAL_ERROR "gsl-confg could not be found. Please specify the directory of gsl-config with ${RedBold}-DGSL_CONFIG_DIR=<directory>${ColReset}.")
else()  
  message(STATUS "Found gsl-config in ${GSL_CONFIG_DIR}")
  execute_process(COMMAND ${GSL_CONFIG} --version
                  OUTPUT_VARIABLE GSL_VERSION
                  OUTPUT_STRIP_TRAILING_WHITESPACE)
  message(STATUS "GSL version: ${GSL_VERSION}")
  execute_process(COMMAND ${GSL_CONFIG} --cflags
                  OUTPUT_VARIABLE GSL_CFLAGS
                  OUTPUT_STRIP_TRAILING_WHITESPACE)
  message(STATUS "GSL cflags: ${GSL_CFLAGS}")
  execute_process(COMMAND ${GSL_CONFIG} --libs
                  OUTPUT_VARIABLE GSL_LIBS
                  OUTPUT_STRIP_TRAILING_WHITESPACE)
  message(STATUS "GSL libs: ${GSL_LIBS}")
  string(REGEX MATCH "^-L[^ ]+" GSL_LIB_PATH ${GSL_LIBS})
  if(NOT GSL_LIB_PATH)
    string(REGEX REPLACE "bin$" "lib" GSL_LIB_PATH ${GSL_CONFIG_DIR})
  else()
    string(REGEX REPLACE "^-L" "" GSL_LIB_PATH ${GSL_LIB_PATH})
  endif()#CXX_FLAGS
  unset(GSL_CONFIG CACHE)
endif()

#######################################################

#Compiler flags: optimization and warnings 
set ( CMAKE_CXX_FLAGS "-O3 -Wall -Wmissing-braces -pedantic ${GSL_CFLAGS}")
include_directories(${BOOST_INC})

# Fix behavior of CMAKE_CXX_STANDARD when targeting macOS.
if (POLICY CMP0025)
    cmake_policy(SET CMP0025 NEW)
endif ()	
	

#Printing the compiler flags
message(STATUS "Compiler flags : ${CMAKE_CXX_FLAGS}")
 
 
 
##########  DEBUG  ##########
if(DEBUG_MODE)
  message("\nCache variables:")
  message("  CMAKE_CXX_COMPILER:FILEPATH = ${CMAKE_CXX_COMPILER}")
  message("  CMAKE_INSTALL_PREFIX:PATH = ${CMAKE_INSTALL_PREFIX}")
  message("  CMAKE_CURRENT_SOURCE_DIR:PATH = ${CMAKE_CURRENT_SOURCE_DIR}")
  message("  DEBUG_MODE:BOOL = ${DEBUG_MODE}")
  message("  LOCAL_INSTALL:BOOL = ${LOCAL_INSTALL}")
  message("  BOOST_INCLUDE_DIR:PATH = ${BOOST_INCLUDE_DIR}")
  message("  GSL_CONFIG_DIR:PATH = ${GSL_CONFIG_DIR}")
  message("\nNormal variables:")
  #message("  LIBTYPE = ${LIBTYPE}")
  message("  BOOST_INC = ${BOOST_INC}")
  message("  GSL_VERSION = ${GSL_VERSION}")
  message("  GSL_CFLAGS = ${GSL_CFLAGS}")
  message("  GSL_LIBS = ${GSL_LIBS}")
  message("  GSL_LIB_PATH = ${GSL_LIB_PATH}")
  message("")

endif(DEBUG_MODE)
 
#############################

#set(PROJECTLIST gslpp Solver) 
 
# sub-directories

#foreach(_project ${PROJECTLIST})
#   add_subdirectory(${_project})
#endforeach(_project)

#RGESolver library libRGESolver.a

file(GLOB srcs "${CMAKE_CURRENT_SOURCE_DIR}/gslpp/src/*.cpp")
file(GLOB headers "${CMAKE_CURRENT_SOURCE_DIR}/gslpp/src/*.h")
add_library(gslpp OBJECT ${srcs} )
add_library(RGESolver $<TARGET_OBJECTS:gslpp>  "${CMAKE_CURRENT_SOURCE_DIR}/Solver/src/RGESolver.cpp" )
target_include_directories(RGESolver PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/gslpp/src/)
target_compile_features(gslpp PUBLIC cxx_std_11)
target_compile_features(RGESolver PUBLIC cxx_std_11)

#include(CMakePrintHelpers)
#cmake_print_properties(TARGETS RGESolver PROPERTIES
#                       INTERFACE_COMPILE_FEATURES CXX_STANDARD)
get_target_property(pippo RGESolver CXX_STANDARD)
if(NOT ${pippo} STREQUAL "pippo-NOTFOUND")
  set(stdflag "-std=c++${pippo}")
else()
  set(stdflag "")
endif()

#add_library(RGESolver STATIC $<TARGET_OBJECTS:gslpp> $<TARGET_OBJECTS:Solver>)

set_target_properties(RGESolver PROPERTIES PUBLIC_HEADER "${CMAKE_CURRENT_SOURCE_DIR}/Solver/src/RGESolver.h;${headers}")

######### make install ############### 

#INSTALL(TARGETS RGESolver DESTINATION lib COMPONENT library DESTINATION include COMPONENT public_header)
INSTALL(TARGETS RGESolver 
        LIBRARY DESTINATION lib
        PUBLIC_HEADER DESTINATION include
)

#INSTALL(FILES ${CMAKE_CURRENT_SOURCE_DIR}/Solver/src/RGESolver.h ${CMAKE_CURRENT_SOURCE_DIR}/gslpp/src/gslpp.h
#        DESTINATION include COMPONENT header)

 configure_file("${CMAKE_CURRENT_SOURCE_DIR}/RGESolver.pc.in" 
                 "${CMAKE_CURRENT_BINARY_DIR}/RGESolver.pc" @ONLY)
  configure_file("${CMAKE_CURRENT_SOURCE_DIR}/rgesolver-config.in" 
                 "${CMAKE_CURRENT_BINARY_DIR}/rgesolver-config" @ONLY)
  INSTALL(PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/rgesolver-config ${CMAKE_CURRENT_BINARY_DIR}/RGESolver.pc 
          DESTINATION bin COMPONENT rgesolver-config)

 
###### make uninstall ###############
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
             "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake" @ONLY)
add_custom_target(uninstall COMMAND ${CMAKE_COMMAND} 
                -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)

 
 
 
