cmake_minimum_required(VERSION 2.8.3)

project(rosbag_storage)

include(GNUInstallDirs)

if(NOT WIN32)
  set_directory_properties(PROPERTIES COMPILE_OPTIONS "-Wall;-Wextra")
endif()

find_package(console_bridge REQUIRED)
#find_package(catkin REQUIRED COMPONENTS cpp_common roscpp_serialization roscpp_traits rostime roslz4)
#find_package(Boost REQUIRED COMPONENTS date_time filesystem program_options regex)
find_package(BZip2 REQUIRED)
IF (BZIP2_FOUND)
   INCLUDE(CheckLibraryExists)
   CHECK_LIBRARY_EXISTS(${BZIP2_LIBRARIES} BZ2_bzCompressInit "" BZIP2_NEED_PREFIX)
ENDIF (BZIP2_FOUND)

set(Boost_INCLUDE_DIRS "/opt/crystax-ndk-10.3.2/sources/boost/1.59.0/include/")
set(Boost_LIBRARIES "-L/opt/crystax-ndk-10.3.2/sources/boost/1.59.0/libs/armeabi-v7a/gnu-4.9/ -lboost_date_time -lboost_filesystem -lboost_program_options -lboost_regex")

# catkin_package(
#   INCLUDE_DIRS include
#   LIBRARIES rosbag_storage
#   CATKIN_DEPENDS roslz4
#   DEPENDS console_bridge Boost
# )

# Support large bags (>2GB) on 32-bit systems
add_definitions(-D_FILE_OFFSET_BITS=64)

message(STATUS ${BZIP2_INCLUDE_DIR})

include_directories(include ${catkin_INCLUDE_DIRS} ${console_bridge_INCLUDE_DIRS} ${Boost_INCLUDE_DIRS} ${BZIP2_INCLUDE_DIR})
add_definitions(${BZIP2_DEFINITIONS})

add_library(rosbag_storage
  src/bag.cpp
  src/bag_player.cpp
  src/buffer.cpp
  src/bz2_stream.cpp
  src/lz4_stream.cpp
  src/chunked_file.cpp
  src/message_instance.cpp
  src/query.cpp
  src/stream.cpp
  src/view.cpp
  src/uncompressed_stream.cpp
)

find_library(crystax_LIBRARIES NAMES crystax)
if (NOT crystax_LIBRARIES)
  message(FATAL_ERROR "crystax library not found")
endif()

target_link_libraries(rosbag_storage ${catkin_LIBRARIES} ${Boost_LIBRARIES} ${crystax_LIBRARIES} ${console_bridge_LIBRARIES})

install(TARGETS ${PROJECT_NAME}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} #${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} #${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} #${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(DIRECTORY include/
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR} #${CATKIN_GLOBAL_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.h")
