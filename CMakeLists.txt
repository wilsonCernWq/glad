# ┌──────────────────────────────────────────────────────────────────┐
# │  Global Properties                                               │
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
add_library(GLAD 
SHARED
  include/glad/glad.h
  include/KHR/khrplatform.h
  src/glad.c
)

if(TARGET OpenGL::OpenGL)
  target_link_libraries(GLAD PUBLIC OpenGL::OpenGL)
else()
  target_link_libraries(GLAD PUBLIC OpenGL::GL)
endif()
if(TARGET OpenGL::GLU)
  target_link_libraries(GLAD PUBLIC OpenGL::GLU)
endif()

target_link_libraries(GLAD 
PUBLIC 
  ${CMAKE_DL_LIBS}
)

target_include_directories(GLAD 
PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_PREFIX}/include>
)

target_compile_definitions(GLAD
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
install(TARGETS GLAD EXPORT GLADConfig) # Windows

if(INSTALL_DEV_HEADERS)

  install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

  # This makes the project importable from the install directory
  # Put config file in per-project dir (name MUST match), can also
  # just go into 'cmake'.
  install(EXPORT GLADConfig
    NAMESPACE GLAD::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/glad
  )

  # This makes the project importable from the build directory
  export(TARGETS GLAD
    NAMESPACE GLAD::
    FILE ${CMAKE_INSTALL_LIBDIR}/cmake/glad/GLADConfig.cmake
  )

endif()
