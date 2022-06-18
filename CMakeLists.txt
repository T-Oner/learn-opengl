cmake_minimum_required(VERSION 3.21)

project(learn_opengl)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/modules/")

link_directories(${CMAKE_SOURCE_DIR}/lib)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/bin)

IF (APPLE)
    INCLUDE_DIRECTORIES(/System/Library/Frameworks)
    FIND_LIBRARY(COCA_LIBRARY Cocoa)
    FIND_LIBRARY(OpenGL_LIBRARY OpenGL)
    FIND_LIBRARY(IOKit_LIBRARY IOKit)
    MARK_AS_ADVANCED(COCA_LIBRARY OpenGL_LIBRARY)
    SET(APPLE_LIBS ${COCA_LIBRARY} ${IOKit_LIBRARY} ${OpenGL_LIBRARY})
    SET(APPLE_LIBS ${APPLE_LIBS} /usr/local/lib/libglfw.dylib)
ENDIF (APPLE)

set(LIBS ${LIBS} ${APPLE_LIBS})

set(CHAPTERS
        1.getting_started
        )

set(1.getting_started
        1.hello_window
        2.hello_triangle)

add_library(GLAD "src/glad.c")

set(LIBS ${LIBS} GLAD)

foreach (CHAPTER ${CHAPTERS})
    foreach (DEMO ${${CHAPTER}})
        file(GLOB SOURCE
                "src/${CHAPTER}/${DEMO}/*.h"
                "src/${CHAPTER}/${DEMO}/*.cpp"
                )
        add_executable(${DEMO} ${SOURCE})
        target_link_libraries(${DEMO} ${LIBS})
        set_target_properties(${DEMO}
                PROPERTIES
                RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/${CHAPTER}"
                )
        # copy shader files to build directory
        file(GLOB SHADERS
                "src/${CHAPTER}/${DEMO}/*.vs"
                "src/${CHAPTER}/${DEMO}/*.frag"
                "src/${CHAPTER}/${DEMO}/*.gs"
                )
        foreach (SHADER ${SHADERS})
            add_custom_command(TARGET ${DEMO} PRE_BUILD COMMAND ${CMAKE_COMMAND} -E
                    copy ${SHADER} $<TARGET_FILE_DIR:${DEMO}>)
        endforeach (SHADER)

        if (MSVC)
            configure_file(${CMAKE_SOURCE_DIR}/configuration/visualstudio.vcxproj.user.in ${CMAKE_CURRENT_BINARY_DIR}/${DEMO}.vcxproj.user $ONLY)
        endif (MSVC)
    endforeach (DEMO)

endforeach (CHAPTER)

set(CMAKE_CXX_STANDARD 14)

include_directories(include)
include_directories(${CMAKE_SOURCE_DIR}/includes)