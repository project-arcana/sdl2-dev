cmake_minimum_required(VERSION 3.8)
project(SDL2)

option(ARC_SDL2_FROM_SOURCE "if true, builds SDL2 from source instead of using system or prebuilt version" OFF)

macro(arc_sdl2_from_source)

    set(SDL_SHARED ON CACHE BOOL "" FORCE)
    add_subdirectory(sdl2)

    add_library(sdl2 INTERFACE)
    target_link_libraries(sdl2 INTERFACE SDL2)
    target_include_directories(sdl2 INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/include)

endmacro()

# if explicit opt-in, build from source
if (ARC_SDL2_FROM_SOURCE)

    arc_sdl2_from_source()

else()

    if (MSVC) # use repo version

        add_library(sdl2 SHARED IMPORTED GLOBAL)

        # prevent SDL from defining main as SDL_main, see https://wiki.libsdl.org/SDL_SetMainReady
        target_compile_definitions(sdl2 INTERFACE SDL_MAIN_HANDLED)

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

    else() # try to use system package

        find_package(SDL2)

        if (SDL2_FOUND) # if found, use system version

            add_library(sdl2 SHARED IMPORTED GLOBAL)
            
            # prevent SDL from defining main as SDL_main, see https://wiki.libsdl.org/SDL_SetMainReady
            target_compile_definitions(sdl2 INTERFACE SDL_MAIN_HANDLED)
            target_include_directories(sdl2 INTERFACE ${SDL2_INCLUDE_DIRS})
            target_link_libraries(sdl2 INTERFACE ${SDL2_LIBRARIES})

        else() # otherwise build from source

            arc_sdl2_from_source()

        endif()

    endif()

endif()