cmake_minimum_required(VERSION 3.10)
project(wl_egl_decoration)

include(FindPkgConfig)

pkg_check_modules(wayland-client REQUIRED wayland-client>=1.12)
pkg_check_modules(wayland-egl REQUIRED wayland-egl)
find_package(OpenGL REQUIRED COMPONENTS EGL)
pkg_check_modules(wayland-cursor REQUIRED wayland-cursor)
pkg_check_modules(gtk REQUIRED gtk+-3.0)

pkg_get_variable(WAYLAND_SCANNER wayland-scanner wayland_scanner)
pkg_get_variable(WAYLAND_PROTOCOLS_DIR wayland-protocols pkgdatadir)

# generate protocols bindings
macro(gen_proto NAME PROT_DEF)
add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/${NAME}-client-protocol.h
    COMMAND ${WAYLAND_SCANNER} client-header ${PROT_DEF} ${NAME}-client-protocol.h)
add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/${NAME}-client-protocol.c
    COMMAND ${WAYLAND_SCANNER} private-code ${PROT_DEF} ${NAME}-client-protocol.c
    DEPENDS ${NAME}-client-protocol.h)
list(APPEND PROTO_SRC
  ${CMAKE_CURRENT_BINARY_DIR}/${NAME}-client-protocol.c
  ${CMAKE_CURRENT_BINARY_DIR}/${NAME}-client-protocol.h
)
endmacro()

include_directories(
  ${CMAKE_CURRENT_BINARY_DIR}
  ${wayland-client_INCLUDE_DIRS}
  ${wayland-egl_INCLUDE_DIRS}
  ${wayland-cursor_INCLUDE_DIRS}
  ${gtk_INCLUDE_DIRS}
)

# xdg-shell
gen_proto(xdg-shell "${WAYLAND_PROTOCOLS_DIR}/stable/xdg-shell/xdg-shell.xml")

add_executable(${PROJECT_NAME} simple-egl.c ${PROTO_SRC})
target_link_libraries(${PROJECT_NAME}
  ${wayland-client_LIBRARIES}
  ${wayland-egl_LIBRARIES}
  m
  OpenGL::EGL
  ${wayland-cursor_LIBRARIES}
  ${gtk_LIBRARIES}
)
