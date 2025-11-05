cmake_minimum_required(VERSION 2.8.3)
project(unitree_legged_real)

add_compile_options(-std=c++11)

find_package(catkin REQUIRED COMPONENTS
    roscpp
    geometry_msgs
    unitree_legged_msgs
)

catkin_package()

include_directories(
    include
    ${Boost_INCLUDE_DIR}
    ${catkin_INCLUDE_DIRS}
)

set(CMAKE_CXX_FLAGS "-O3")

# -----------------------------------------------------------
# 1. Detect architecture automatically (amd64 / arm64)
# -----------------------------------------------------------
if(NOT DEFINED UNITREE_PLATFORM)
    string(TOLOWER "${CMAKE_SYSTEM_PROCESSOR}" UNITREE_PLATFORM)
    if(UNITREE_PLATFORM STREQUAL "x86_64" OR UNITREE_PLATFORM STREQUAL "amd64")
        set(UNITREE_PLATFORM "amd64")
    elseif(UNITREE_PLATFORM STREQUAL "arm64" OR UNITREE_PLATFORM STREQUAL "aarch64")
        set(UNITREE_PLATFORM "arm64")
    else()
        message(WARNING "Unknown architecture '${CMAKE_SYSTEM_PROCESSOR}', defaulting to amd64")
        set(UNITREE_PLATFORM "amd64")
    endif()
endif()
message(STATUS "Detected UNITREE_PLATFORM = ${UNITREE_PLATFORM}")

# -----------------------------------------------------------
# 2. Auto-detect Unitree SDK path
# -----------------------------------------------------------
if(NOT DEFINED UNITREE_LEGGED_SDK_PATH)
    # Assume it's next to this package directory
    get_filename_component(PARENT_DIR "${CMAKE_CURRENT_SOURCE_DIR}" DIRECTORY)
    set(UNITREE_LEGGED_SDK_PATH "${PARENT_DIR}/unitree_legged_sdk")

    if(NOT EXISTS "${UNITREE_LEGGED_SDK_PATH}/include" OR NOT EXISTS "${UNITREE_LEGGED_SDK_PATH}/lib")
        message(FATAL_ERROR
            "Could not find Unitree SDK in ${UNITREE_LEGGED_SDK_PATH}.\n"
            "Please make sure unitree_legged_sdk is next to unitree_legged_real, or set UNITREE_LEGGED_SDK_PATH manually."
        )
    endif()
endif()
message(STATUS "Using UNITREE_LEGGED_SDK_PATH = ${UNITREE_LEGGED_SDK_PATH}")

# -----------------------------------------------------------
# 3. Include & link SDK
# -----------------------------------------------------------
include_directories(${UNITREE_LEGGED_SDK_PATH}/include)
link_directories(${UNITREE_LEGGED_SDK_PATH}/lib)

string(CONCAT LEGGED_SDK_NAME "libunitree_legged_sdk_" ${UNITREE_PLATFORM} ".so")
set(EXTRA_LIBS ${LEGGED_SDK_NAME} lcm)

add_definitions(-DSDK3_2)

# -----------------------------------------------------------
# 4. Build executables
# -----------------------------------------------------------
add_executable(lcm_server ${UNITREE_LEGGED_SDK_PATH}/examples/lcm_server.cpp)
target_link_libraries(lcm_server ${EXTRA_LIBS} ${catkin_LIBRARIES})
add_dependencies(lcm_server ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

add_executable(position_lcm src/exe/position_mode.cpp)
target_link_libraries(position_lcm ${EXTRA_LIBS} ${catkin_LIBRARIES})
add_dependencies(position_lcm ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

add_executable(velocity_lcm src/exe/velocity_mode.cpp)
target_link_libraries(velocity_lcm ${EXTRA_LIBS} ${catkin_LIBRARIES})
add_dependencies(velocity_lcm ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

add_executable(torque_lcm src/exe/torque_mode.cpp)
target_link_libraries(torque_lcm ${EXTRA_LIBS} ${catkin_LIBRARIES})
add_dependencies(torque_lcm ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

add_executable(walk_lcm src/exe/walk_mode.cpp)
target_link_libraries(walk_lcm ${EXTRA_LIBS} ${catkin_LIBRARIES})
add_dependencies(walk_lcm ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

