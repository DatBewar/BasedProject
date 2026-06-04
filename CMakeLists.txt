cmake_minimum_required(VERSION 4.0)
project(Project)

set(CMAKE_CXX_STANDARD 20)

# Setup Cmake
set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/CMake;${CMAKE_MODULE_PATH}")

#option(ENABLE_CPP20_MODULE "Enable C++ 20 module support for Vulkan" OFF)

# Dependencies

include(FetchContent)

FetchContent_Declare(
    glm
    GIT_REPOSITORY "https://github.com/g-truc/glm.git"
    GIT_TAG "0.9.9.8"
    GIT_SHALLOW TRUE
    OVERRIDE_FIND_PACKAGE
)

set(GLFW_BUILD_DOCS OFF)
set(GLFW_BUILD_TESTS OFF)
set(GLFW_BUILD_EXAMPLES OFF)
FetchContent_Declare(
    glfw3
    GIT_REPOSITORY "https://github.com/glfw/glfw.git"
    GIT_TAG "3.3.8"
    GIT_SHALLOW TRUE
    OVERRIDE_FIND_PACKAGE
)

FetchContent_Declare(
    tinyobjloader
    GIT_REPOSITORY "https://github.com/tinyobjloader/tinyobjloader.git"
    GIT_TAG "v2.0.0rc13"
    GIT_SHALLOW TRUE
    OVERRIDE_FIND_PACKAGE
)


set(IMGUI_IMPL_VULKAN_HAS_DYNAMIC_RENDERING ON)
FetchContent_Declare(
    imgui
    GIT_REPOSITORY "https://github.com/ocornut/imgui"
    GIT_TAG "v1.89.8"
    GIT_SHALLOW TRUE
    OVERRIDE_FIND_PACKAGE
)

FetchContent_MakeAvailable(imgui)

# Build ImGui from source (available via FetchContent)
add_library(imgui STATIC
    ${imgui_SOURCE_DIR}/imgui.cpp
    ${imgui_SOURCE_DIR}/imgui_draw.cpp
    ${imgui_SOURCE_DIR}/imgui_tables.cpp
    ${imgui_SOURCE_DIR}/imgui_widgets.cpp
    ${imgui_SOURCE_DIR}/backends/imgui_impl_glfw.cpp
    ${imgui_SOURCE_DIR}/backends/imgui_impl_vulkan.cpp
)

target_include_directories(imgui PUBLIC ${imgui_SOURCE_DIR} ${imgui_SOURCE_DIR}/backends)

target_link_libraries(imgui PUBLIC glfw Vulkan::Vulkan)

# Setup Vulkan

# Enable C++ module dependency scanning only if C++ 20 module is enabled
if(ENABLE_CPP20_MODULE)
    set(CMAKE_CXX_SCAN_FOR_MODULES ON)
endif()

find_package(glfw3 REQUIRED)
find_package(glm REQUIRED)
find_package(tinyobjloader REQUIRED)
find_package(Vulkan REQUIRED)
find_program(SLANGC_EXECUTABLE slangc HINTS "${Vulkan_SLANGC_EXECUTABLE_DIR}" ENV PATH REQUIRED)
#find_package(tinygltf REQUIRED)

include(${CMAKE_CURRENT_LIST_DIR}/cmake/function.cmake)


file(GLOB_RECURSE ProjectSources CONFIGURE_DEPENDS
    "${CMAKE_CURRENT_SOURCE_DIR}/engine/*.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/engine/*.h*"
)

add_executable(Project ${ProjectSources})

target_link_libraries(Project PRIVATE Vulkan::cppm glm glfw imgui)

target_include_directories(Project PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/deps" PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/engine" "${CMAKE_CURRENT_SOURCE_DIR}/engine/core")

target_compile_features(Project PRIVATE cxx_std_20)

#target_precompile_headers(Project PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/engine/precomp.h")

# Shaders
add_slang_shader_target(CompileShaders SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/engine/shaders/demo.slang")
add_dependencies(Project CompileShaders)
