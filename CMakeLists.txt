# Created by Manuel Blum on 2011-05-25.
# Copyright 2013 University of Freiburg.

CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

PROJECT(libgp CXX C)

# if no option is given, standard is release
IF(NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)
   SET(CMAKE_BUILD_TYPE Release)
ENDIF(NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)

set(CMAKE_CXX_FLAGS_RELEASE "-DCLSQUARE -Wall -O2 -fPIC")
set(CMAKE_CXX_FLAGS_DEBUG "-DCLSQUARE -Wall -g -fPIC")

OPTION(BUILD_TESTS "Build tests" ON)
OPTION(BUILD_EXAMPLES "Build examples" ON)

SET(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_MODULE_PATH})

FIND_PACKAGE(Eigen3 3.0.1 REQUIRED)

# ----- Add include directories -----
INCLUDE_DIRECTORIES(SYSTEM ${EIGEN3_INCLUDE_DIR})
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR}/include)

# ----- Add sources -----
INCLUDE("Sources.cmake")

# ----- Add library target -----
ADD_LIBRARY(gp ${LIBGP_SRC} ${LIBGP_INTERFACES})

# ----- Generate PKgconfig file ----
get_property(LIB64 GLOBAL PROPERTY FIND_LIBRARY_USE_LIB64_PATHS)
if ("${LIB64}" STREQUAL "TRUE")
    set(LIBSUFFIX 64)
else()
    set(LIBSUFFIX "")
endif()
set(INSTALL_LIB_DIR lib${LIBSUFFIX} CACHE PATH
    "Installation directory for libraries")
mark_as_advanced(INSTALL_LIB_DIR)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/contrib/libgp.pc.in 
	${CMAKE_BINARY_DIR}/libgp.pc @ONLY)

# ----- Add example targets -----
IF(BUILD_EXAMPLES)
  ADD_SUBDIRECTORY(examples)
ENDIF()

# ----- Testing -----
IF(BUILD_TESTS)
  ADD_SUBDIRECTORY(tests)
ENDIF()

# ----- Install -----
INSTALL(FILES ${LIBGP_INTERFACES} DESTINATION include/gp)
INSTALL(TARGETS gp
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION ${INSTALL_LIB_DIR}
  ARCHIVE DESTINATION ${INSTALL_LIB_DIR}
)
INSTALL(FILES ${CMAKE_BINARY_DIR}/libgp.pc DESTINATION ${INSTALL_LIB_DIR}/pkgconfig)
