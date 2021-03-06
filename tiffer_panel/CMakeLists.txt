cmake_minimum_required(VERSION 2.8.3)
list( APPEND CMAKE_CXX_FLAGS "-std=c++17 ${CMAKE_CXX_FLAGS}")

# PACKAGE SETUP ########################################

project(tiffer_panel)

find_package(catkin REQUIRED COMPONENTS std_msgs tf roscpp rviz)

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES ${PROJECT_NAME}
  CATKIN_DEPENDS std_msgs
)

# QT ########################################

set(CMAKE_AUTOMOC ON)
set(CMAKE_INCLUDE_CURRENT_DIR ON)

## This plugin includes Qt widgets, so we must include Qt.
## We'll use the version that rviz used so they are compatible.
if(rviz_QT_VERSION VERSION_LESS "5")
  message(tiffer "Using Qt4 based on the rviz_QT_VERSION: ${rviz_QT_VERSION}")
  find_package(Qt4 ${rviz_QT_VERSION} EXACT REQUIRED QtCore QtGui)
  ## pull in all required include dirs, define QT_LIBRARIES, etc.
  include(${QT_USE_FILE})
  qt4_wrap_cpp(QT_MOC include/tiffer_panel/tiffer_panel.h include/tiffer_panel/tiffer_locationManager.h include/tiffer_panel/tiffer_cruiseLocation.h)
else()
  message(tiffer "Using Qt5 based on the rviz_QT_VERSION: ${rviz_QT_VERSION}")
  find_package(Qt5 ${rviz_QT_VERSION} EXACT REQUIRED Core Widgets Multimedia)
  ## make target_link_libraries(${QT_LIBRARIES}) pull in all required dependencies
  set(QT_LIBRARIES Qt5::Core Qt5::Widgets Qt5::Multimedia)
  qt5_wrap_cpp(QT_MOC include/tiffer_panel/tiffer_panel.h include/tiffer_panel/tiffer_locationManager.h include/tiffer_panel/tiffer_cruiseLocation.h include/tiffer_panel/tiffer_record.h include/tiffer_panel/clean_path.h)
endif()

add_definitions(-DQT_NO_KEYWORDS)

# FILES ########################################

include_directories(
  ${catkin_INCLUDE_DIRS}
  include/
)
set(SOURCES src/tiffer_panel.cpp src/tiffer_logger.cpp src/tiffer_locationManager.cpp src/tiffer_cruiseLocation.cpp ${QT_MOC})

# LIBRARY ########################################
add_library(${PROJECT_NAME} ${SOURCES})

target_link_libraries(${PROJECT_NAME} ${QT_LIBRARIES} ${catkin_LIBRARIES})

install(TARGETS
  ${PROJECT_NAME}
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(FILES
  plugin_description.xml
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})

install(DIRECTORY icons/
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/icons)
