cmake_minimum_required(VERSION 3.10)

# Toplevel check
set(OSI_IS_TOP_LEVEL OFF)
if(CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
  set(OSI_IS_TOP_LEVEL ON)
endif()

# Set a default build type if none was specified
set(default_build_type "Release")

if(OSI_IS_TOP_LEVEL AND NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message(STATUS "Setting build type to '${default_build_type}' as none was specified.")
  set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE
      STRING "Choose the type of build." FORCE)
  # Set the possible values of build type for cmake-gui
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
               "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

# read the version number from the file "VERSION"
file(STRINGS "open-simulation-interface/VERSION" VERSION_CONTENTS)
foreach(LINE ${VERSION_CONTENTS})
  string(REGEX REPLACE " |\t" "" LINE ${LINE})
  string(REGEX MATCH "^[^=]+" VERSION_NAME ${LINE})
  string(REPLACE "${VERSION_NAME}=" "" VERSION_VALUE ${LINE})
  set(${VERSION_NAME} "${VERSION_VALUE}")
endforeach()
set(OPEN_SIMULATION_INTERFACE_VERSION ${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH})

project(
  open_simulation_interface
  DESCRIPTION "A generic interface for the environmental perception of automated driving functions in virtual scenarios."
  HOMEPAGE_URL "https://github.com/OpenSimulationInterface/open-simulation-interface"
  VERSION ${OPEN_SIMULATION_INTERFACE_VERSION}
  LANGUAGES CXX
)

# Set the C++ standard
set(CMAKE_CXX_STANDARD 17 CACHE STRING "C++ standard to be used")
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(MSVC)
  add_compile_options(/wd4141 /wd4244 /wd4251 /wd4267 /wd4305 /wd4355 /wd4800 /wd4996)
endif()

set(${PROJECT_NAME}_SOVERSION "${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")
set(${PROJECT_NAME}_LIBVERSION "${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")

# Offer the user the choice of overriding the installation directories
set(OSI_INSTALL_LIB_DIR lib CACHE PATH "Installation directory for libraries")
set(OSI_INSTALL_INCLUDE_DIR include CACHE PATH "Installation directory for header files")

if(WIN32 AND NOT CYGWIN)
  set(DEF_INSTALL_CMAKE_DIR CMake/${PROJECT_NAME}-${VERSION_MAJOR})
else()
  set(DEF_INSTALL_CMAKE_DIR lib/cmake/${PROJECT_NAME}-${VERSION_MAJOR})
endif()
set(OSI_INSTALL_CMAKE_DIR ${DEF_INSTALL_CMAKE_DIR} CACHE PATH "Installation directory for CMake files")

set(OSI_INSTALL_LIB_DIR ${OSI_INSTALL_LIB_DIR})
set(OSI_INSTALL_INCLUDE_DIR ${OSI_INSTALL_INCLUDE_DIR}/osi${VERSION_MAJOR})

configure_file(open-simulation-interface/osi_version.proto.in osi_version.proto)

find_package(Protobuf REQUIRED)
include_directories(${Protobuf_INCLUDE_DIRS})

file(GLOB OSI_PROTO_FILES open-simulation-interface/*.proto)
list(INSERT OSI_PROTO_FILES 0 ${CMAKE_CURRENT_BINARY_DIR}/osi_version.proto)

protobuf_generate_cpp(PROTO_SRCS PROTO_HEADERS ${OSI_PROTO_FILES})

add_library(${PROJECT_NAME}_static STATIC ${PROTO_SRCS} ${PROTO_HEADERS})
add_library(${PROJECT_NAME}::${PROJECT_NAME}_static ALIAS ${PROJECT_NAME}_static)
target_include_directories(${PROJECT_NAME}_static
    PUBLIC
        ${PROTOBUF_INCLUDE_DIR}
        $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
        $<INSTALL_INTERFACE:${OSI_INSTALL_INCLUDE_DIR}>
)
target_link_libraries(${PROJECT_NAME}_static PUBLIC protobuf::libprotobuf)
install(TARGETS ${PROJECT_NAME}_static
        EXPORT ${PROJECT_NAME}_targets
        ARCHIVE DESTINATION "${OSI_INSTALL_LIB_DIR}" COMPONENT lib)


add_library(${PROJECT_NAME}_obj OBJECT ${PROTO_SRCS} ${PROTO_HEADERS})
add_library(${PROJECT_NAME}::${PROJECT_NAME}_obj ALIAS ${PROJECT_NAME}_obj)
target_include_directories(${PROJECT_NAME}_obj
    PUBLIC
        ${PROTOBUF_INCLUDE_DIR}
        $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
        $<INSTALL_INTERFACE:${OSI_INSTALL_INCLUDE_DIR}>
)
set_property(TARGET ${PROJECT_NAME}_obj PROPERTY POSITION_INDEPENDENT_CODE ON)


add_library(${PROJECT_NAME}_pic STATIC $<TARGET_OBJECTS:${PROJECT_NAME}_obj>)
add_library(${PROJECT_NAME}::${PROJECT_NAME}_pic ALIAS ${PROJECT_NAME}_pic)
target_include_directories(${PROJECT_NAME}_pic
    PUBLIC
        ${PROTOBUF_INCLUDE_DIR}
        $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
        $<INSTALL_INTERFACE:${OSI_INSTALL_INCLUDE_DIR}>
)
target_link_libraries(${PROJECT_NAME}_pic PUBLIC protobuf::libprotobuf)

set_property(TARGET ${PROJECT_NAME}_pic PROPERTY POSITION_INDEPENDENT_CODE ON)

install(TARGETS ${PROJECT_NAME}_pic
        EXPORT ${PROJECT_NAME}_targets
        ARCHIVE DESTINATION "${OSI_INSTALL_LIB_DIR}" COMPONENT lib)

add_library(${PROJECT_NAME} SHARED $<TARGET_OBJECTS:${PROJECT_NAME}_obj>)
add_library(${PROJECT_NAME}::${PROJECT_NAME} ALIAS ${PROJECT_NAME})
target_include_directories(${PROJECT_NAME}
    PUBLIC
        ${PROTOBUF_INCLUDE_DIR}
        $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
        $<INSTALL_INTERFACE:${OSI_INSTALL_INCLUDE_DIR}>
)

set_property(
    TARGET ${PROJECT_NAME}
    PROPERTY SOVERSION ${${PROJECT_NAME}_SOVERSION}
)
set_property(
    TARGET ${PROJECT_NAME}
    PROPERTY VERSION ${${PROJECT_NAME}_LIBVERSION}
)

target_link_libraries(${PROJECT_NAME} PUBLIC protobuf::libprotobuf)
install(TARGETS ${PROJECT_NAME}
        EXPORT ${PROJECT_NAME}_targets
        DESTINATION "${OSI_INSTALL_LIB_DIR}" COMPONENT lib)

# Copy proto headers to where they are expected by the package config file
add_custom_command(
    TARGET ${PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E make_directory
    ${CMAKE_CURRENT_BINARY_DIR}/${OSI_INSTALL_INCLUDE_DIR}
    COMMAND ${CMAKE_COMMAND} -E copy
    ${PROTO_HEADERS}
    ${CMAKE_CURRENT_BINARY_DIR}/${OSI_INSTALL_INCLUDE_DIR})

# Create the package config files
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/open_simulation_interface-config-version.cmake"
    VERSION ${OPEN_SIMULATION_INTERFACE_VERSION}
    COMPATIBILITY SameMajorVersion
)

export(EXPORT ${PROJECT_NAME}_targets
    FILE "${CMAKE_CURRENT_BINARY_DIR}/open_simulation_interface-targets.cmake"
    NAMESPACE ${PROJECT_NAME}::
)

configure_package_config_file(open_simulation_interface-config.cmake.in
    "${CMAKE_CURRENT_BINARY_DIR}/open_simulation_interface-config.cmake"
    INSTALL_DESTINATION ${OSI_INSTALL_CMAKE_DIR}
    PATH_VARS OSI_INSTALL_INCLUDE_DIR
    NO_CHECK_REQUIRED_COMPONENTS_MACRO
)

# Install the *cmake files
install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/open_simulation_interface-config.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/open_simulation_interface-config-version.cmake"
    DESTINATION "${OSI_INSTALL_CMAKE_DIR}"
    COMPONENT dev)

# Header files
install(FILES ${PROTO_HEADERS}
        DESTINATION "${OSI_INSTALL_INCLUDE_DIR}")

# Install the export set for use with the install-tree
install(EXPORT ${PROJECT_NAME}_targets
    FILE open_simulation_interface-targets.cmake
    NAMESPACE ${PROJECT_NAME}::
    DESTINATION "${OSI_INSTALL_CMAKE_DIR}"
    COMPONENT dev)
