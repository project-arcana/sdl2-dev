# How to Update SDL2

In our SDL2 usage, we require the includes to be prefixed SDL2/
(#include <SDL2/SDL.h> instead of just <SDL.h>)

This is required because we use three different deployments of SDL2:
1. Binaries (Windows)
2. find_package (Linux)
3. Build from source (CI)

SDL2's source natively does not have this prefix, but the distributions installed by linux package managers do.
To make 3. compatible with the rest, we move all headers in include/ to include/SDL2/

## What to Download

1. Head to https://www.libsdl.org/download-2.0.php
2. Download Win32 binaries: "Development Libraries", you care only about the lib/ folder
    - Replace the contents of ./lib/ with it
3. Download the Source Code zip
    - Discard everything but the cmake/, include/, and src/ folder as well as all top level files

## Steps when Updating

1. Completely replace the contents of sdl2/ with the new files
2. Move all files in sdl2/include/ to a new subfolder sdl2/include/SDL2
3. Adjust the ./sdl2/CMakeLists.txt (NOT the top level one in this folder) as follows:

    3.1 Point the general include to the subfolder instead of just ${SDL2_SOURCE_DIR}/include
    In SDL 2.0.14 this was line 278, it should now look like this:
    
        # General includes
        include_directories(${SDL2_BINARY_DIR}/include ${SDL2_SOURCE_DIR}/include/SDL2)

    3.2 Change the path for file removals
    In SDL 2.0.14 this was line 2346, it should now look like this:

        foreach(_FNAME ${BIN_INCLUDE_FILES})
            get_filename_component(_INCNAME ${_FNAME} NAME)
            list(REMOVE_ITEM INCLUDE_FILES ${SDL2_SOURCE_DIR}/include/SDL2/${_INCNAME})
        endforeach()

    3.3 Change the path for the SDL_Config.h generation. Make sure to ONLY change the first path here!
    In SDL 2.0.12 this was line 2067, it should now look like this:

        configure_file("${SDL2_SOURCE_DIR}/include/SDL2/SDL_config.h.cmake"
            "${SDL2_BINARY_DIR}/include/SDL_config.h")

4. Fix non-relative includes in sdl2/src (of the form `#include "../../include/*"` to `"../../include/SDL2/*"`) (Replace: `include/SDL_` with `include/SDL2/SDL_`)
    
    1. in src/events/scancodes_*.h
