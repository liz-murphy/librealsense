cmake_minimum_required(VERSION 2.8.3)
project(realsense)

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
find_package(catkin REQUIRED COMPONENTS
)

find_package(PkgConfig REQUIRED)
pkg_search_module(LIBUSB1 REQUIRED libusb-1.0)
pkg_search_module(GLFW REQUIRED glfw3)

find_package(OpenGL REQUIRED)

set(BACKEND "V4L2")
set(CMAKE_CXX_FLAGS "-std=c++11 -fPIC -pedantic -mssse3 -O3 -Wno-missing-field-initializers -Wno-switch -Wno-multichar -DRS_USE_${BACKEND}_BACKEND")
set(CMAKE_C_FLAGS "-std=c11 -fPIC -pedantic -DRS_USE_${BACKEND}")

catkin_package(
    INCLUDE_DIRS include 
    LIBRARIES realsense 
    CATKIN_DEPENDS 
    DEPENDS libusb
    )

file(GLOB OBJECTS_CPP
    "src/*.cpp"
    )

file(GLOB UVC_C
    "src/libuvc/*.c")

include_directories(include src/no-jdk ${LIBUSB1_INCLUDE_DIRS})
add_library(realsense ${OBJECTS_CPP})

add_library(uvc ${UVC_C}) #Does not appear necessary for 4.2.6 kernel
add_executable(multicamera examples/cpp-multicam.cpp)
add_executable(enumerate_devices examples/cpp-enumerate-devices.cpp)
target_link_libraries(realsense ${LIBUSB1_LIBRARIES})
target_link_libraries(multicamera realsense uvc ${GLFW_LIBRARIES} ${OPENGL_LIBRARIES}) #4.2.6 kernel does not need uvc
target_link_libraries(enumerate_devices realsense uvc ${GLFW_LIBRARIES} ${OPENGL_LIBRARIES}) #4.2.6 kernel does not need uvc

install(TARGETS realsense multicamera enumerate_devices
    ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(DIRECTORY include/librealsense
    DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
)

install(PROGRAMS
    scripts/install_dependencies-4.4.sh
    scripts/patch-uvcvideo-4.4.sh
    scripts/install_glfw3.sh 
    DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

#install(DIRECTORY examples/assets/
#    DESTINATION ${CATKIN_DEVEL_PREFIX}/lib/${PROJECT_NAME}/examples/assets)
#file(COPY examples/assets/
#    DESTINATION ${CATKIN_DEVEL_PREFIX}/lib/${PROJECT_NAME}/examples/assets)
