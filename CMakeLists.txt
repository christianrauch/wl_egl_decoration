cmake_minimum_required(VERSION 3.10)
project(wl_egl_decoration)

include(FindPkgConfig)

set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} \
    -fsanitize=address \
    -fsanitize=bool \
    -fsanitize=bounds \
    -fsanitize=enum \
    -fsanitize=float-cast-overflow \
    -fsanitize=float-divide-by-zero \
    -fsanitize=nonnull-attribute \
    -fsanitize=returns-nonnull-attribute \
    -fsanitize=signed-integer-overflow \
    -fsanitize=undefined \
    -fsanitize=vla-bound \
    -fno-sanitize=alignment \
    -fsanitize=leak \
    -fsanitize=object-size \
")

pkg_check_modules(wayland-client REQUIRED wayland-client>=1.12)
pkg_check_modules(wayland-egl REQUIRED wayland-egl)
find_package(OpenGL REQUIRED COMPONENTS EGL)
pkg_check_modules(wayland-cursor REQUIRED wayland-cursor)

pkg_check_modules(decoration REQUIRED decoration)

include_directories(
  ${wayland-client_INCLUDE_DIRS}
  ${wayland-egl_INCLUDE_DIRS}
  ${wayland-cursor_INCLUDE_DIRS}
  ${decoration_INCLUDE_DIRS}
)

add_executable(${PROJECT_NAME} simple-egl.c ${PROTO_SRC})
target_link_libraries(${PROJECT_NAME}
  ${wayland-client_LIBRARIES}
  ${wayland-egl_LIBRARIES}
  m
  OpenGL::EGL
  ${wayland-cursor_LIBRARIES}
  ${decoration_LIBRARIES}
)
