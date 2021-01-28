cmake_minimum_required(VERSION 3.10)
project(map-viewer-thing VERSION 0.1)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

if (EXISTS ./grend)
	message(STATUS "Found grend subdirectory, using that as library")
	set(GREND_PATH ./grend)
endif()

if (GREND_PATH)
	message(STATUS "GREND_PATH: ${GREND_PATH}")
	add_subdirectory(${GREND_PATH})
endif()

set (DEMO_LINK_LIBS)
if (ANDROID OR GREND_PATH)
	# everything needed should be pulled in here
	find_library(Grend Grend)
	list(APPEND DEMO_LINK_LIBS Grend)

# TODO: assume unixen, presumably there'll be different library lookup
#       stuff for windows/etc
else()
	add_compile_options(-DDEMO_PREFIX="${CMAKE_INSTALL_PREFIX}/share/map-viewer/")
	find_package(PkgConfig REQUIRED)
	pkg_check_modules(Grend REQUIRED IMPORTED_TARGET grend)
	list(APPEND DEMO_LINK_LIBS PkgConfig::Grend)
endif()

set(MAP_VIEWER_SRC src/main.cpp)

if (ANDROID)
	message(STATUS "Setting library for android")
	set(TARGET_NAME main)
	add_library(main SHARED ${MAP_VIEWER_SRC})
	install(TARGETS main DESTINATION lib)

else()
	message(STATUS "Setting standalone executable")
	set(TARGET_NAME map-viewer)
	add_executable(map-viewer ${MAP_VIEWER_SRC})
	install(TARGETS map-viewer DESTINATION bin)
endif()

target_include_directories(${TARGET_NAME} PUBLIC "${PROJECT_BINARY_DIR}")
target_link_libraries(${TARGET_NAME} ${DEMO_LINK_LIBS})
