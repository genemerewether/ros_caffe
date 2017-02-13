cmake_minimum_required(VERSION 2.8.3)
project(ros_caffe)

#add_definitions(-DCPU_ONLY=1)

find_package(catkin REQUIRED COMPONENTS
  cv_bridge
  std_msgs
  sensor_msgs
  image_transport
  roscpp
  roslib
  cmake_modules
  message_generation
  dynamic_reconfigure
)

find_package(Boost REQUIRED COMPONENTS system)
find_package(OpenCV REQUIRED)
add_definitions(-DUSE_ACCELERATE)
#set(CUDA_TOOLKIT_ROOT_DIR "/Developer/NVIDIA/CUDA-8.0")
find_package(CUDA REQUIRED)
#find_package(Caffe REQUIRED)

include(FindPkgConfig)

unset(Caffe_FOUND)

find_path(Caffe_INCLUDE_DIRS NAMES caffe/caffe.hpp caffe/common.hpp caffe/net.hpp caffe/proto/caffe.pb.h caffe/util/io.hpp
  HINTS
  /usr/local/caffe/include)

find_library(Caffe_LIBRARIES NAMES caffe
  HINTS
  /usr/local/caffe/lib)

if(Caffe_LIBRARIES AND Caffe_INCLUDE_DIRS)
    set(Caffe_FOUND 1)
endif()

include_directories(${Caffe_INCLUDE_DIRS})
add_definitions(${Caffe_DEFINITIONS})

catkin_package(
        INCLUDE_DIRS include
        LIBRARIES ${PACKAGE_NAME}_lib
        CATKIN_DEPENDS
                cv_bridge
                std_msgs
                sensor_msgs
                image_transport
                roscpp
                cmake_modules
		message_runtime
        DEPENDS system_lib
)

include_directories( include
    ${catkin_INCLUDE_DIRS}
    ${OpenCV_INCLUDE_DIRS}
    ${Caffe_INCLUDE_DIRS}
    ${CUDA_INCLUDE_DIRS}
)

add_library(${PROJECT_NAME}_lib src/ros_caffe.cpp)
link_directories(${Caffe_LIBRARIES})

add_executable(${PROJECT_NAME}_node src/ros_caffe.cpp)
add_executable(${PROJECT_NAME}_test src/ros_caffe_test.cpp)

target_link_libraries(${PROJECT_NAME}_lib
    ${catkin_LIBRARIES}
    ${OpenCV_LIBRARIES}
    ${Caffe_LIBRARIES}
    glog
)

target_link_libraries(${PROJECT_NAME}_node
  ${PROJECT_NAME}_lib
)

target_link_libraries(${PROJECT_NAME}_test
  ${PROJECT_NAME}_lib
)

install(TARGETS ${PROJECT_NAME}_lib ${PROJECT_NAME}_node ${PROJECT_NAME}_test
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

# install(TARGETS 
#   RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
# )

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.h"
  PATTERN ".svn" EXCLUDE
)

install(DIRECTORY data/
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/data
  PATTERN ".svn" EXCLUDE
)

install(DIRECTORY cfg/
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/cfg
  PATTERN ".svn" EXCLUDE
)

install(DIRECTORY launch/
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/launch
  PATTERN ".svn" EXCLUDE
)

## Mark other files for installation (e.g. launch and bag files, etc.)
# install(FILES
#    ${${PROJECT_NAME}_LIBRARIES}
#    DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
# )
