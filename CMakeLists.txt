cmake_minimum_required(VERSION 3.0)

include(tacklelib/Project)
include(tacklelib/_3dparty/Qt)

project("qwtpolar")

set(PROJECT_LIB_NAME qwtpolar)

tkl_check_var(REQUIRED PATH QT5_ROOT)
tkl_check_var(REQUIRED PATH QWT_ROOT)

file(GLOB_RECURSE all_headers ${CMAKE_CURRENT_LIST_DIR}/src/*.h*)
file(GLOB_RECURSE all_sources ${CMAKE_CURRENT_LIST_DIR}/src/*.c*)

# if Qt enabled
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTORCC ON)

add_library(${PROJECT_LIB_NAME} ${all_headers};${all_sources})

tkl_initialize_library_target_defaults(${PROJECT_LIB_NAME} "32bit")

tkl_source_groups_from_dir_list("Header Files" FILES ${CMAKE_CURRENT_LIST_DIR} *.h*)
tkl_source_groups_from_dir_list("Source Files" FILES ${CMAKE_CURRENT_LIST_DIR} *.c*)

tkl_source_groups_from_dir_list("Generated Files" FILES ${PROJECT_ROOT}/_out/build qrc_*.c*)
tkl_source_groups_from_dir_list("Generated Files" FILES ${PROJECT_ROOT}/_out/build mocs_*.c*)

# Qt

if(QT5_ROOT)
  set(CMAKE_PREFIX_PATH ${QT5_ROOT}/lib/cmake)
  if(EXISTS ${CMAKE_PREFIX_PATH}/)
    set(QT_REQUIRED 1)
    set(DESIRED_QT_VERSION 5)

    find_qt_component(QT5_ROOT Qt5Core 1)
    find_qt_component(QT5_ROOT Qt5Gui 1)
    find_qt_component(QT5_ROOT Qt5Widgets 1)
    find_qt_component(QT5_ROOT Qt5Concurrent 1) # for QwtPolar
  endif()
endif()

if(Qt5Core_FOUND)
  message(STATUS "(*) Found `QT`: `${QT_VERSION_STRING}` Location: \"${QT5_ROOT}\" Components: [${QT_COMPONENTS}] Libs: [${QT_LIBRARIES}] Plugins: [${QT_PLUGINS}]")

  link_qt_components(${PROJECT_LIB_NAME} PUBLIC PUBLIC QT_LIBRARIES QT_COMPONENTS)
else()
  message(WARNING "(*) QT is not found")
endif()

# Qwt

find_package(QWT_ROOT Qwt REQUIRED)

if(QWT_FOUND)
  message(STATUS "(*) Found `Qwt`: `${QWT_VERSION_STRING}` Location: \"${QWT_ROOT_DIR}\" IncludeDirs: [${QWT_INCLUDE_DIRS}] LibDirs: [${QWT_LIBRARY_DIRS}] Libs: [${QWT_LIBRARIES}]")

  tkl_add_target_compile_definitions(${PROJECT_LIB_NAME} *
    PUBLIC
      QWT_DLL # issue: https://stackoverflow.com/questions/30985698/lnk2001-unresolved-external-symbol-when-using-qwt-plot-in-qt-creator
  )

  target_include_directories(${PROJECT_LIB_NAME}
    PUBLIC
      ${QWT_ROOT}/include
  )

  if (QWT_LIBRARIES)
    target_link_libraries(${PROJECT_LIB_NAME}
      PUBLIC
        ${QWT_LIBRARIES}
    )
  endif()
else()
  message(WARNING "(*) Qwt is not found")
endif()
