cmake_minimum_required(VERSION 2.8.8)

include(ExternalProject)
ExternalProject_Add(qpoases_svn
  PREFIX qpoases_svn
  SVN_REPOSITORY https://projects.coin-or.org/svn/qpOASES/trunk
  SVN_REVISION -r 113
  SVN_TRUST_CERT 1
  PATCH_COMMAND patch -p0 < ${CMAKE_CURRENT_SOURCE_DIR}/linker_opts.diff
  BUILD_COMMAND ""
  INSTALL_COMMAND ""
)

project(qpoases_ros)

find_package(catkin REQUIRED)

catkin_package(
  #INCLUDE_DIRS TODO
  LIBRARIES qpOASES
)

ExternalProject_Get_Property(qpoases_svn SOURCE_DIR)

include_directories(${SOURCE_DIR}/include)

# "We do not recommend using GLOB to collect a list of source files from your source tree."
set(qpOASES_SRC
  ${SOURCE_DIR}/src/BLASReplacement.cpp
  ${SOURCE_DIR}/src/Bounds.cpp
  ${SOURCE_DIR}/src/Constraints.cpp
  ${SOURCE_DIR}/src/Flipper.cpp
  ${SOURCE_DIR}/src/Indexlist.cpp
  ${SOURCE_DIR}/src/LAPACKReplacement.cpp
  ${SOURCE_DIR}/src/Matrices.cpp
  ${SOURCE_DIR}/src/MessageHandling.cpp
  ${SOURCE_DIR}/src/Options.cpp
  ${SOURCE_DIR}/src/OQPinterface.cpp
  ${SOURCE_DIR}/src/QProblemB.cpp
  ${SOURCE_DIR}/src/QProblem.cpp
  ${SOURCE_DIR}/src/SolutionAnalysis.cpp
  ${SOURCE_DIR}/src/SQProblem.cpp
  ${SOURCE_DIR}/src/SubjectTo.cpp
  ${SOURCE_DIR}/src/Utils.cpp
)
# the files listed above are present when qpoases_svn (i.e. getting source files) is completed
add_custom_command(OUTPUT ${qpOASES_SRC}
  DEPENDS qpoases_svn)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D__NO_COPYRIGHT__ -O3 -finline-functions")

# c++ library
add_library(qpOASES
  ${qpOASES_SRC})

# python wrappers
add_custom_target(qpOASES_Python ALL
  COMMAND python setup.py build_ext --library-dirs ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_LIB_DESTINATION} --build-lib ${CATKIN_DEVEL_PREFIX}/${CATKIN_GLOBAL_PYTHON_DESTINATION}
  WORKING_DIRECTORY ${SOURCE_DIR}/interfaces/python
  COMMENT "Building Python wrappers."
)
add_dependencies(qpOASES_Python qpOASES)
