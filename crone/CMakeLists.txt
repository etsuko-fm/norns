cmake_minimum_required(VERSION 3.7)
project(crone VERSION 1.0.0)

add_subdirectory(softcut/softcut-lib)

set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall")
set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS_DEBUG} -O0")
set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS_RELEASE} -O2")

set (SRC src/main.cpp
        src/BufDiskWorker.cpp
        src/Commands.cpp
        src/MixerClient.cpp
        src/OscInterface.cpp
        src/SoftcutClient.cpp
        src/Taper.cpp
        src/Window.cpp
        softcut/softcut-lib/src/FadeCurves.cpp
        softcut/softcut-lib/src/ReadWriteHead.cpp
        softcut/softcut-lib/src/SubHead.cpp
        softcut/softcut-lib/src/Svf.cpp
        softcut/softcut-lib/src/Voice.cpp )

add_executable(crone ${SRC})

include_directories(. ./faust ./softcut/softcut-lib/include lib/readerwriterqueue)

if(UNIX)
    if(APPLE)
        find_library(lo_lib lo REQUIRED HINTS /opt/homebrew/lib /usr/local/lib)
        find_library(sndfile_lib sndfile REQUIRED HINTS /opt/homebrew/lib /usr/local/lib)
        find_library(jack_lib jack REQUIRED HINTS /opt/homebrew/lib /usr/local/lib)
        # If the static libsndfile.a was chosen (it bundles vorbis/ogg), link those
        # deps explicitly so the linker can resolve their symbols.
        find_library(vorbis_lib vorbis HINTS /opt/homebrew/lib /usr/local/lib)
        find_library(vorbisenc_lib vorbisenc HINTS /opt/homebrew/lib /usr/local/lib)
        find_library(vorbisfile_lib vorbisfile HINTS /opt/homebrew/lib /usr/local/lib)
        find_library(ogg_lib ogg HINTS /opt/homebrew/lib /usr/local/lib)
        target_link_libraries(crone ${lo_lib} ${sndfile_lib} ${jack_lib})
        if(vorbis_lib)
            target_link_libraries(crone ${vorbis_lib} ${vorbisenc_lib} ${vorbisfile_lib} ${ogg_lib})
        endif()
        include_directories(/opt/homebrew/include /usr/local/include)
    else()
        target_link_libraries(crone lo)
        target_link_libraries(crone jack)
        target_link_libraries(crone pthread)
        target_link_libraries(crone asound)
        target_link_libraries(crone sndfile)
    endif()
    target_link_libraries(crone softcut)
else()
    # nope
endif()


set_target_properties(crone PROPERTIES
CXX_STANDARD 14
CXX_STANDARD_REQUIRED YES
CXX_EXTENSIONS YES
)

target_compile_options(crone PRIVATE -Wall -Wextra -pedantic)
