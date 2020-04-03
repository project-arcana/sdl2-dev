cmake_minimum_required(VERSION 3.8)
project(SDL2)

add_library(sdl2 SHARED IMPORTED GLOBAL)

# prevent SDL from defining main as SDL_main, see https://wiki.libsdl.org/SDL_SetMainReady
target_compile_definitions(sdl2 INTERFACE SDL_MAIN_HANDLED)

if (MSVC) # use repo version

    target_include_directories(sdl2 INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/include)

    # Support both 32 and 64 bit builds
    if (${CMAKE_SIZEOF_VOID_P} MATCHES 8)
        set_target_properties(sdl2 PROPERTIES IMPORTED_IMPLIB ${CMAKE_CURRENT_SOURCE_DIR}/lib/x64/SDL2.lib)
        set(ARC_SDL2_DEV_DLL_PATH "${CMAKE_CURRENT_SOURCE_DIR}/lib/x64/SDL2.dll")
    else()
        set_target_properties(sdl2 PROPERTIES IMPORTED_IMPLIB ${CMAKE_CURRENT_SOURCE_DIR}/lib/x86/SDL2.lib)
        set(ARC_SDL2_DEV_DLL_PATH "${CMAKE_CURRENT_SOURCE_DIR}/lib/x86/SDL2.dll")
    endif()

    # copy the dll to the runtime output folder now
    file(COPY "${ARC_SDL2_DEV_DLL_PATH}" DESTINATION "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}")

else() # use system package

    find_package(SDL2 REQUIRED)

    target_include_directories(sdl2 INTERFACE ${SDL2_INCLUDE_DIRS})
    target_link_libraries(sdl2 INTERFACE ${SDL2_LIBRARIES})

endif()
