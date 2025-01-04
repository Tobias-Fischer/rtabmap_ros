cmake_minimum_required(VERSION 3.20)
project(rtabmap_rviz_plugins)

FIND_PACKAGE(Qt5 COMPONENTS Widgets Core Gui REQUIRED)

find_package(catkin REQUIRED COMPONENTS
             roscpp sensor_msgs std_msgs pcl_conversions pluginlib rviz tf rtabmap_conversions rtabmap_msgs)

## We also use Ogre for rviz plugins
# this file doesn't exist post-noetic, but pkg_check_modules still works
find_package(PkgConfig REQUIRED)
pkg_check_modules(OGRE OGRE)

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES rtabmap_rviz_plugins
  CATKIN_DEPENDS roscpp sensor_msgs std_msgs pcl_conversions pluginlib rviz tf rtabmap_conversions rtabmap_msgs
)

###########
## Build ##
###########

include_directories(
  ${CMAKE_CURRENT_SOURCE_DIR}/include
  ${Qt5_INCLUDE_DIRS}
  ${Qt5Core_INCLUDE_DIRS}
  ${Qt5Widgets_INCLUDE_DIRS}
  ${catkin_INCLUDE_DIRS}
  ${OGRE_INCLUDE_DIRS}
)

link_directories( ${OGRE_LIBRARY_DIRS} )

SET(Libraries
  ${catkin_LIBRARIES}
  ${rviz_DEFAULT_PLUGIN_LIBRARIES}
)

## RVIZ plugin
  IF(WIN32)
    qt5_wrap_cpp(MOC_FILES
      include/${PROJECT_NAME}/MapCloudDisplay.h
      include/${PROJECT_NAME}/MapGraphDisplay.h
      include/${PROJECT_NAME}/InfoDisplay.h
    )
  ELSE()
    qt5_wrap_cpp(MOC_FILES
      include/${PROJECT_NAME}/MapCloudDisplay.h
      include/${PROJECT_NAME}/MapGraphDisplay.h
      include/${PROJECT_NAME}/InfoDisplay.h
      include/${PROJECT_NAME}/OrbitOrientedViewController.h
    )
  ENDIF()

# tf:message_filters, mixing boost and Qt signals
IF(WIN32)
  set_property(
    SOURCE src/MapCloudDisplay.cpp src/MapGraphDisplay.cpp src/InfoDisplay.cpp
    PROPERTY COMPILE_DEFINITIONS QT_NO_KEYWORDS
  )
ELSE()
  set_property(
    SOURCE src/MapCloudDisplay.cpp src/MapGraphDisplay.cpp src/InfoDisplay.cpp src/OrbitOrientedViewController.cpp
    PROPERTY COMPILE_DEFINITIONS QT_NO_KEYWORDS
  )
ENDIF()
SET(SRC_FILES 
   src/MapCloudDisplay.cpp
   src/MapGraphDisplay.cpp
   src/InfoDisplay.cpp
   ${MOC_FILES}
)
IF(NOT WIN32)
  SET(SRC_FILES 
    ${SRC_FILES} 
    src/OrbitOrientedViewController.cpp
  )
ENDIF(NOT WIN32)
add_library(rtabmap_rviz_plugins
   ${SRC_FILES}
)
target_link_libraries(rtabmap_rviz_plugins
  ${Libraries}
  Qt5::Core
  Qt5::Widgets
  Qt5::Gui
)

#############
## Install ##
#############

## Mark cpp header files for installation
install(DIRECTORY include/${PROJECT_NAME}/
   DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
   FILES_MATCHING PATTERN "*.h"
)

install(TARGETS 
   rtabmap_rviz_plugins
   ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
   LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
   RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
 )
install(FILES
   rviz_plugins.xml
   DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)
