cmake_minimum_required(VERSION 2.8.3)
project(rviz_calibration)

add_definitions(-std=c++11)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x") 
set(CMAKE_CXX_FLAGS "-Wall ${CMAKE_CXX_FLAGS}")

find_package(catkin REQUIRED COMPONENTS
        message_generation
        apriltags2
        pcl_conversions
        pcl_ros
        OpenCV
        cv_bridge
        image_transport
        roscpp
        sensor_msgs
        std_msgs
        rviz
        tf
        cmake_modules      
)

## Generate messages in the 'msg' folder
add_message_files(FILES
    projection_matrix.msg
    PointsImage.msg
    AprilTagDetection.msg
    AprilTagDetectionArray.msg
    AprilTagPoint.msg
)

# find PCL package
find_package(PCL REQUIRED)
message(STATUS "    PCL version: ${PCL_VERSION}")
message(STATUS "    PCL include path: ${PCL_INCLUDE_DIRS}")

#find OpenCV
find_package(OpenCV REQUIRED)
message(STATUS "    OpenCV version: ${OpenCV_VERSION}")
message(STATUS "    OpenCV include path: ${OpenCV_INCLUDE_DIRS}")

#find Eigen
find_package(Eigen REQUIRED)


generate_messages(DEPENDENCIES
  std_msgs
)

catkin_package(
	INCLUDE_DIRS
  DEPENDS rviz OpenCV PCL Eigen message_runtime
  CATKIN_DEPENDS apriltags2
)


## This setting causes Qt's "MOC" generation to happen automatically.
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)


## Compile as C++11, supported in ROS Kinetic and newer
# add_compile_options(-std=c++11)

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages

## This plugin includes Qt widgets, so we must include Qt.
## We'll use the version that rviz used so they are compatible.
if(rviz_QT_VERSION VERSION_LESS "5")
  message(STATUS "Using Qt4 based on the rviz_QT_VERSION: ${rviz_QT_VERSION}")
  find_package(Qt4 ${rviz_QT_VERSION} EXACT REQUIRED QtCore QtGui)
  qt4_wrap_ui(UI_HEADERS
            src/calibration.ui
            )
  ## pull in all required include dirs, define QT_LIBRARIES, etc.
else()
  message(STATUS "Using Qt5 based on the rviz_QT_VERSION: ${rviz_QT_VERSION}")
  find_package(Qt5 ${rviz_QT_VERSION} EXACT REQUIRED Core Widgets)
  set(QT_LIBRARIES Qt5::Widgets)
  qt5_wrap_ui(UI_HEADERS
            src/calibration.ui
            )
  
  ## make target_link_libraries(${QT_LIBRARIES}) pull in all required dependencies
endif()

## I prefer the Qt signals and slots to avoid defining "emit", "slots",
## etc because they can conflict with boost signals, so define QT_NO_KEYWORDS here.

find_package(PkgConfig REQUIRED)
pkg_check_modules(YAML_CPP REQUIRED yaml-cpp)
find_path(YAML_CPP_INCLUDE_DIR
          NAMES yaml_cpp.h
          PATHS ${YAML_CPP_INCLUDE_DIRS})
find_library(YAML_CPP_LIBRARY
             NAMES YAML_CPP
             PATHS ${YAML_CPP_LIBRARY_DIRS})

include_directories(
  include 
  ${catkin_INCLUDE_DIRS}
  ${CMAKE_CURRENT_SOURCE_DIR}
  ${CMAKE_CURRENT_BINARY_DIR}
  ${catkin_INCLUDE_DIRS}
  ${Eigen_INCLUDE_DIRS}
  src
  ${PCL_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS}     
)

link_directories(${catkin_LIBRARY_DIRS} ${YAML_CPP_LIBRARY_DIRS})

add_definitions(-DQT_NO_KEYWORDS)

## Here we specify which header files need to be run through "moc",
## Qt's meta-object compiler.
 
## Here we specify the list of source files, including the output of
## the previous command which is stored in ``${MOC_FILES}``.
set(SRC_FILES
  src/ThreadSafeQueue.h
  src/calibration.cpp
  src/mainwindow.cpp
  src/shape.cpp
  src/line.cpp
  src/rect.cpp
  src/paintwidget.cpp
  src/view.cpp
  src/apriltags.cpp
  ${MOC_FILES} 
  ${UI_HEADERS}
)
 
## An rviz plugin is just a shared library, so here we declare the
## library to be called ``${PROJECT_NAME}`` (which is
## "rviz_plugin_tutorials", or whatever your version of this project
## is called) and specify the list of source files we collected above
## in ``${SOURCE_FILES}``.
add_library(${PROJECT_NAME} ${SRC_FILES})

set_target_properties(${PROJECT_NAME}
        PROPERTIES
        COMPILE_FLAGS
        "-DIMAGE_DEFAULT_PATH=${CMAKE_CURRENT_SOURCE_DIR}"
        )

 
## Link the library with whatever Qt libraries have been defined by
## the ``find_package(Qt4 ...)`` line above, and with whatever libraries
## catkin has included.
##
## Although this puts "rviz_plugin_tutorials" (or whatever you have
## called the project) as the name of the library, cmake knows it is a
## library and names the actual file something like
## "librviz_plugin_tutorials.so", or whatever is appropriate for your
## particular OS.
target_link_libraries(${PROJECT_NAME} ${QT_LIBRARIES} ${catkin_LIBRARIES} ${OpenCV_LIBRARIES} ${Eigen_LIBRARIES})

install(TARGETS
  ${PROJECT_NAME}
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(FILES
  plugin_description.xml
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})

add_dependencies(${PROJECT_NAME}
  ${catkin_EXPORTED_TARGETS}
  ${OpenCV_LIBS}
  ${PCL_LIBRARIES}
  )
