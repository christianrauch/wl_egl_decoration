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

pkg_check_modules(decoration QUIET libdecor-0)

link_directories(
    ${decoration_LIBRARY_DIRS}
)

add_executable(${PROJECT_NAME} simple-egl.c ${PROTO_SRC})
target_include_directories(${PROJECT_NAME} PUBLIC
  ${wayland-client_INCLUDE_DIRS}
  ${wayland-egl_INCLUDE_DIRS}
  ${wayland-cursor_INCLUDE_DIRS}
)
target_link_libraries(${PROJECT_NAME} PUBLIC
  ${wayland-client_LIBRARIES}
  ${wayland-egl_LIBRARIES}
  m
  OpenGL::EGL
  ${wayland-cursor_LIBRARIES}
)

if(decoration_FOUND)
    target_include_directories(${PROJECT_NAME} PUBLIC ${decoration_INCLUDE_DIRS})
    target_link_libraries(${PROJECT_NAME} PUBLIC ${decoration_LIBRARIES})
else()
    include(ExternalProject)
    ExternalProject_Add(libdecor
        PREFIX "${CMAKE_CURRENT_BINARY_DIR}/extern/libdecor"
        GIT_REPOSITORY https://gitlab.gnome.org/jadahl/libdecoration.git
        GIT_TAG wip/api-versions
        CONFIGURE_COMMAND meson --prefix "${CMAKE_CURRENT_BINARY_DIR}/install" --libdir "lib" ../libdecor
        BUILD_COMMAND ninja
        INSTALL_COMMAND ninja install
    )

    add_dependencies(${PROJECT_NAME} libdecor)

    target_include_directories(${PROJECT_NAME} PRIVATE "${CMAKE_CURRENT_BINARY_DIR}/install/include/libdecor-0")
    target_link_libraries(${PROJECT_NAME} PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/install/lib/libdecor-0${CMAKE_SHARED_LIBRARY_SUFFIX})
endif()
