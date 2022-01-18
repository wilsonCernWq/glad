# ┌──────────────────────────────────────────────────────────────────┐
# │  Global Properties                                               │
# │  TODO: Support other Core APIs in new branches                   │
# └──────────────────────────────────────────────────────────────────┘
cmake_minimum_required(VERSION 3.3)
project(GLAD CXX)

option(INSTALL_DEV_HEADERS "Whether to install GLAD headers" OFF)

# ┌──────────────────────────────────────────────────────────────────┐
# │  Configurate Packages                                            │
# └──────────────────────────────────────────────────────────────────┘
set(OpenGL_GL_PREFERENCE GLVND)
find_package(OpenGL REQUIRED)

# ┌──────────────────────────────────────────────────────────────────┐
# │  Configure Source Files                                          │
# └──────────────────────────────────────────────────────────────────┘
set_source_files_properties(src/glad.c PROPERTIES LANGUAGE CXX)

add_library(glad 
SHARED
  include/glad/glad.h
  include/KHR/khrplatform.h
  src/glad.c
)

if(TARGET OpenGL::OpenGL)
  target_link_libraries(glad PUBLIC OpenGL::OpenGL)
else()
  target_link_libraries(glad PUBLIC OpenGL::GL)
endif()
if(TARGET OpenGL::GLU)
  target_link_libraries(glad PUBLIC OpenGL::GLU)
endif()

target_link_libraries(glad 
PUBLIC 
  ${CMAKE_DL_LIBS}
)

target_include_directories(glad 
PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_PREFIX}/include>
)

target_compile_definitions(glad
PUBLIC 
  GLAD_GLAPI_EXPORT
PRIVATE 
  GLAD_GLAPI_EXPORT_BUILD
PUBLIC
  GLAD_GLAPI_CORE_VERSION_MAJOR=3
  GLAD_GLAPI_CORE_VERSION_MINOR=3
)

# ┌──────────────────────────────────────────────────────────────────┐
# │  Configure Install                                               │
# └──────────────────────────────────────────────────────────────────┘
# 'make install' to the correct locations (provided by GNUInstallDirs).
install(TARGETS glad EXPORT gladExport) # Windows

if(INSTALL_DEV_HEADERS)

  install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

  # This makes the project importable from the install directory
  # Put config file in per-project dir (name MUST match), can also
  # just go into 'cmake'.
  install(EXPORT GLADExport
    # NAMESPACE GLAD::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/glad
  )

  # This makes the project importable from the build directory
  export(TARGETS glad
    # NAMESPACE GLAD::
    FILE ${CMAKE_INSTALL_LIBDIR}/cmake/glad/gladConfig.cmake
  )

endif()
