cmake_minimum_required(VERSION 3.1)

set(project_name "OBSOpencastIngestPlugin")
project(${project_name})

message("================================================")
message("Project: ${project_name}")

# OBS lib
message("================================================")
find_package(LibObs REQUIRED)
message("Found: LibObs")
message("Include dirs: ${LIBOBS_INCLUDE_DIRS}")
message("Libraries: ${LIBOBS_LIBRARIES}")
set(${project_name}_INCLUDE_DIRS ${${project_name}_INCLUDE_DIRS} ${LIBOBS_INCLUDE_DIRS})
set(${project_name}_LIBRARIES ${${project_name}_LIBRARIES} ${LIBOBS_LIBRARIES})

# obs-frontend-api lib
message("================================================")
find_library(OBS_FRONTEND_API_LIBRARIES NAMES obs-frontend-api)
message("Using: obs-frontend-api")
message("Libraries: ${OBS_FRONTEND_API_LIBRARIES}")
set(${project_name}_LIBRARIES ${${project_name}_LIBRARIES} ${OBS_FRONTEND_API_LIBRARIES})


# OpencastIngest lib
message("================================================")
# prefere static linking here
find_library(OpencastIngest_LIBRARIES NAMES OpencastIngest_static OpencastIngest)
find_path(OpencastIngest_INCLUDE_DIRS libOpencastIngest.hpp)
message("Using: libOpencastIngest")
message("Include dirs: ${OpencastIngest_INCLUDE_DIRS}")
message("Libraries: ${OpencastIngest_LIBRARIES}")
set(${project_name}_INCLUDE_DIRS ${${project_name}_INCLUDE_DIRS} ${OpencastIngest_INCLUDE_DIRS})
set(${project_name}_LIBRARIES ${${project_name}_LIBRARIES} ${OpencastIngest_LIBRARIES})

# Curl lib
message("================================================")
find_library(LIBCURL_LIBRARIES NAMES curl)
message("Using: libcurl")
message("Libraries: ${LIBCURL_LIBRARIES}")
set(${project_name}_LIBRARIES ${${project_name}_LIBRARIES} ${LIBCURL_LIBRARIES})

# tinyxml2 lib
message("================================================")
find_library(TINYXML2_LIBRARIES NAMES tinyxml2)
message("Using: libtinyxml2")
message("Include dirs: ${TINYXML2_INCLUDE_DIRS}")
message("Libraries: ${TINYXML2_LIBRARIES}")
set(${project_name}_INCLUDE_DIRS ${${project_name}_INCLUDE_DIRS} ${TINYXML2_INCLUDE_DIRS})
set(${project_name}_LIBRARIES ${${project_name}_LIBRARIES} ${TINYXML2_LIBRARIES})

# libconfig
message("================================================")
find_library(LIBCONFIG_LIBRARIES NAMES config)
message("Using: libconfig")
message("Include dirs: ${LIBCONFIG_INCLUDE_DIRS}")
message("Libraries: ${LIBCONFIG_LIBRARIES}")
set(${project_name}_INCLUDE_DIRS ${${project_name}_INCLUDE_DIRS} ${LIBCONFIG_INCLUDE_DIRS})
set(${project_name}_LIBRARIES ${${project_name}_LIBRARIES} ${LIBCONFIG_LIBRARIES})

# Qt5 Core lib
message("================================================")
find_package(Qt5Core REQUIRED COMPONENTS Qt5::Core)
get_target_property(Qt5Core_LOCATION Qt5::Core LOCATION)
message("Using: Qt5 Core")
message("Include dirs: ${Qt5Core_INCLUDE_DIRS}")
message("Libraries: ${Qt5Core_LOCATION}")
set(${project_name}_INCLUDE_DIRS ${${project_name}_INCLUDE_DIRS} ${Qt5Core_INCLUDE_DIRS})
set(${project_name}_LIBRARIES ${${project_name}_LIBRARIES} ${Qt5Core_LOCATION})

# Qt5 Widgets lib
message("================================================")
find_package(Qt5Widgets REQUIRED COMPONENTS Qt5::Widgets)
get_target_property(Qt5Widgets_LOCATION Qt5::Widgets LOCATION)
message("Using: Qt5 Widgets")
message("Include dirs: ${Qt5Widgets_INCLUDE_DIRS}")
message("Libraries: ${Qt5Widgets_LOCATION}")
set(${project_name}_INCLUDE_DIRS ${${project_name}_INCLUDE_DIRS} ${Qt5Widgets_INCLUDE_DIRS})
set(${project_name}_LIBRARIES ${${project_name}_LIBRARIES} ${Qt5Widgets_LOCATION})


# git version

message("================================================")
if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/.git)
	find_package(Git)
	if(GIT_FOUND)
		execute_process(COMMAND ${GIT_EXECUTABLE} describe
		    WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
		    OUTPUT_VARIABLE ${project_name}_VERSION
		    ERROR_QUIET
		    OUTPUT_STRIP_TRAILING_WHITESPACE
		)
	else()
		set(${project_name}_VERSION "UNKNOWN")
	endif()
else()
	set(${project_name}_VERSION "UNKNOWN")
endif()
message("Build Version: ${${project_name}_VERSION}" )

configure_file(
	"${PROJECT_SOURCE_DIR}/src/obsOpencastIngestPlugin_version.h.in"
	"${PROJECT_BINARY_DIR}/src/obsOpencastIngestPlugin_version.h"
)
include_directories("${PROJECT_BINARY_DIR}/src")

# main qt

message("================================================")
set(main_qt_ui "src/ui/infoBox.ui" "src/ui/dialogopencast.ui" "src/ui/opencastSettings.ui")
qt5_wrap_ui(main_ui_HEADERS ${main_qt_ui})
message("Qt generating ui: ${main_qt_ui}")

foreach(loop_var IN ITEMS ${main_ui_HEADERS})
	get_filename_component(main_ui_HEADERS_PATH ${loop_var} DIRECTORY)
	set(main_ui_HEADERS_PATHS ${main_ui_HEADERS_PATHS} ${main_ui_HEADERS_PATH})
endforeach()

set(main_qt_moc "src/moc/guiLauncher.h" "src/moc/infoBox.h" "src/moc/dialogopencast.h" "src/moc/opencastSettings.h")
qt5_wrap_cpp(main_moc_SOURCE ${main_qt_moc})
message("Qt moc: ${main_qt_moc}")

set(main_qt_source "src/moc/guiLauncher.cpp" "src/moc/infoBox.cpp" "src/moc/dialogopencast.cpp" "src/moc/opencastSettings.cpp" ${main_qt_moc} ${main_moc_SOURCE} ${main_ui_HEADERS})
set(main_qt_include ${main_ui_HEADERS_PATHS})
message("Qt Source: ${main_qt_source}")
message("Qt Includes: ${main_qt_include}")

# main

message("================================================")

set(main_gnucxx_warn_err_compile_flags "-Wall -Wextra -pedantic-errors")

set(main_headers "src/obsOpencastIngestPlugin.hpp" "src/utilsAndConfig.hpp")
set(main_source ${main_headers} "src/obsOpencastIngestPlugin.cpp" "src/utilsAndConfig.cpp" ${main_qt_source})
set(main_include_dirs ${${project_name}_INCLUDE_DIRS} ${main_qt_include})
set(main_libraries ${${project_name}_LIBRARIES} ${Qt5_LIBRARIES})

message("Name (lib using C++11): ${project_name}")
message("Used source: ${main_source}")
message("Include dirs: ${main_include_dirs}")
message("Libraries: ${main_libraries}")

add_library(${project_name} SHARED ${main_source})

if(CMAKE_COMPILER_IS_GNUCXX)
	set_target_properties(${project_name} PROPERTIES COMPILE_FLAGS ${main_gnucxx_warn_err_compile_flags})
endif()
set_target_properties(${project_name} PROPERTIES
    CXX_STANDARD 11
    CXX_STANDARD_REQUIRED YES
    CXX_EXTENSIONS NO
)
target_include_directories(${project_name} PRIVATE ${main_include_dirs})
target_link_libraries(${project_name} PRIVATE ${main_libraries})


message("================================================")
# build a CPack driven installer package
include (InstallRequiredSystemLibraries)
set(CPACK_PACKAGE_NAME "${project_name}")
set(CPACK_PACKAGE_VERSION "${${project_name}_VERSION}")
set(CPACK_INCLUDE_TOPLEVEL_DIRECTORY false)
set(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}.${CMAKE_SYSTEM_PROCESSOR}")
set(CPACK_SOURCE_IGNORE_FILES "${CPACK_SOURCE_IGNORE_FILES};${CMAKE_BINARY_DIR};.git.*")
if(WIN32)
set(CPACK_GENERATOR "ZIP")
set(CPACK_SOURCE_GENERATOR "ZIP")
message("Packaging type: ZIP")
message("Packaging output: ${CPACK_PACKAGE_FILE_NAME}.zip")
else()
set(CPACK_GENERATOR "TGZ")
set(CPACK_SOURCE_GENERATOR "TGZ")
message("Packaging type: TGZ")
message("Packaging output: ${CPACK_PACKAGE_FILE_NAME}.tar.gz")
endif()
include(CPack)

install(TARGETS ${project_name} DESTINATION .)
