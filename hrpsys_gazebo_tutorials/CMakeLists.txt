if(NOT USE_ROSBUILD)
  include(catkin.cmake)
  return()
endif()

cmake_minimum_required(VERSION 2.4.6)
include($ENV{ROS_ROOT}/core/rosbuild/rosbuild.cmake)

# Set the build type.  Options are:
#  Coverage       : w/ debug symbols, w/o optimization, w/ code-coverage
#  Debug          : w/ debug symbols, w/o optimization
#  Release        : w/o debug symbols, w/ optimization
#  RelWithDebInfo : w/ debug symbols, w/ optimization
#  MinSizeRel     : w/o debug symbols, w/ optimization, stripped binaries
#set(ROS_BUILD_TYPE RelWithDebInfo)

rosbuild_init()

#set the default path for built executables to the "bin" directory
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
#set the default path for built libraries to the "lib" directory
set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)


macro (generate_gazebo_urdf_file _robot_name)
  set(_out_dir "${PROJECT_SOURCE_DIR}/robot_models/${_robot_name}")
  set(_out_urdf_file "${_out_dir}/${_robot_name}.urdf")
  add_custom_command(OUTPUT ${_out_dir}/meshes
    COMMAND mkdir ${_out_dir}/meshes)
  add_custom_command(OUTPUT ${_out_dir}/hrpsys
    COMMAND mkdir ${_out_dir}/hrpsys)
  add_custom_command(OUTPUT ${_out_urdf_file}
    COMMAND ${PROJECT_SOURCE_DIR}/robot_models/install_robot_common.sh ${_robot_name}
    DEPENDS ${_out_dir}/hrpsys ${_out_dir}/meshes)
  add_custom_target(${_robot_name}_compile DEPENDS ${_out_urdf_file})
  set(ROBOT ${_robot_name})
  string(TOLOWER ${_robot_name} _sname)
  #configure_file(${PROJECT_SOURCE_DIR}/scripts/default_robot_hrpsys_bringup.launch.in ${PROJECT_SOURCE_DIR}/build/${_sname}_hrpsys_bringup.launch)
  if(NOT EXISTS ${PROJECT_SOURCE_DIR}/launch/gazebo_${_sname}_no_controllers.launch)
    configure_file(${PROJECT_SOURCE_DIR}/scripts/default_gazebo_robot_no_controllers.launch.in ${PROJECT_SOURCE_DIR}/launch/gazebo_${_sname}_no_controllers.launch)
  endif()
  list(APPEND compile_robots ${_robot_name}_compile)
endmacro()

generate_gazebo_urdf_file(SampleRobot)
generate_gazebo_urdf_file(HRP3HAND_L)
generate_gazebo_urdf_file(HRP3HAND_R)
generate_gazebo_urdf_file(HRP2JSK)
generate_gazebo_urdf_file(HRP2JSKNT)
generate_gazebo_urdf_file(HRP2JSKNTS)
generate_gazebo_urdf_file(STARO)
generate_gazebo_urdf_file(HRP4C)

add_custom_target(all_robots_compile ALL DEPENDS ${compile_robots})


#uncomment if you have defined messages
#rosbuild_genmsg()
#uncomment if you have defined services
#rosbuild_gensrv()

#common commands for building c++ executables and libraries
#rosbuild_add_library(${PROJECT_NAME} src/example.cpp)
#target_link_libraries(${PROJECT_NAME} another_library)
#rosbuild_add_boost_directories()
#rosbuild_link_boost(${PROJECT_NAME} thread)
#rosbuild_add_executable(example examples/example.cpp)
#target_link_libraries(example ${PROJECT_NAME})
