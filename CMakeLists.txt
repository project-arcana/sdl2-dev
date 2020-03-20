cmake_minimum_required(VERSION 3.8)
project(SDL2)

add_library(sdl2 SHARED IMPORTED GLOBAL)

# prevent SDL from defining main as SDL_main, see https://wiki.libsdl.org/SDL_SetMainReady
target_compile_definitions(sdl2 INTERFACE SDL_MAIN_HANDLED)

if (MSVC)

# use repo version
set(SDL2_INCLUDE_DIRS "${CMAKE_CURRENT_SOURCE_DIR}/include")
# Support both 32 and 64 bit builds
if (${CMAKE_SIZEOF_VOID_P} MATCHES 8)
    set(SDL2_LIBRARIES "${CMAKE_CURRENT_SOURCE_DIR}/lib/x64/SDL2.lib;${CMAKE_CURRENT_SOURCE_DIR}/lib/x64/SDL2main.lib")
else()
    set(SDL2_LIBRARIES "${CMAKE_CURRENT_SOURCE_DIR}/lib/x86/SDL2.lib;${CMAKE_CURRENT_SOURCE_DIR}/lib/x86/SDL2main.lib")
endif()
string(STRIP "${SDL2_LIBRARIES}" SDL2_LIBRARIES)

else()

# use system package
find_package(SDL2 REQUIRED)

endif()

target_include_directories(sdl2 INTERFACE ${SDL2_INCLUDE_DIRS})
target_link_libraries(sdl2 INTERFACE ${SDL2_LIBRARIES})

message(STATUS "[sdl2] include dir ${SDL2_INCLUDE_DIRS}")
message(STATUS "[sdl2] libs ${SDL2_LIBRARIES}")
