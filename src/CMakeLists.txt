find_package(Threads REQUIRED)
set(THREADS_PREFER_PTHREAD_FLAG ON)

if(MINGW)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fpermissive -DAL_LIBTYPE_STATIC")
endif(MINGW)

file(GLOB_RECURSE ${PROJECT}_SOURCES "*.cpp" "*.h" "*.rc")

function(header_directories RETURN_LIST)
    file(GLOB_RECURSE ALL_SRCS *.h *.cpp *.c)
    set(RELDIRS)
    foreach(SRC ${ALL_SRCS})
        file(RELATIVE_PATH RELSRC "${CMAKE_CURRENT_SOURCE_DIR}" "${SRC}")
        get_filename_component(RELDIR "${RELSRC}" DIRECTORY)
        list(APPEND RELDIRS ${RELDIR})
    endforeach()
    list(REMOVE_DUPLICATES RELDIRS)
    set(${RETURN_LIST} ${RELDIRS} PARENT_SCOPE)
endfunction()

header_directories(${PROJECT}_INCLUDES)

configure_file("${CMAKE_CURRENT_SOURCE_DIR}/extras/GitSHA1.cpp.in" "${CMAKE_CURRENT_BINARY_DIR}/extras/GitSHA1.cpp" @ONLY)
list(APPEND ${PROJECT}_SOURCES "${CMAKE_CURRENT_BINARY_DIR}/extras/GitSHA1.cpp")

add_executable(${EXECUTABLE} WIN32
    ${${PROJECT}_SOURCES}
)

target_link_libraries(${EXECUTABLE} PRIVATE
    librw::librw
    Threads::Threads
)
if(MINGW AND MINGW_FORCE_EXTRA_LIBS)
    target_link_libraries(${EXECUTABLE} PRIVATE
        mpg123
        shlwapi
        gdi32
        oleaut32
        user32
        mingw32
        opus
        opusfile
        ogg
        winmm
        imm32
        oleaut32
    )
endif(MINGW AND MINGW_FORCE_EXTRA_LIBS)

target_include_directories(${EXECUTABLE}
    PRIVATE
        $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>
        $<BUILD_INTERFACE:${${PROJECT}_INCLUDES}>
)

target_compile_definitions(${EXECUTABLE}
    PRIVATE
        $<IF:$<CONFIG:DEBUG>,DEBUG,NDEBUG>
        LIBRW
        CMAKE_NO_AUTOLINK
)

if(LIBRW_PLATFORM_D3D9)
    target_compile_definitions(${EXECUTABLE}
        PUBLIC
            USE_D3D9
    )
endif()

target_compile_definitions(${EXECUTABLE} PRIVATE CMAKE_BUILD)
target_compile_definitions(${EXECUTABLE} PRIVATE USE_OUR_VERSIONING)

if(${PROJECT}_AUDIO STREQUAL "OAL")
    find_package(OpenAL REQUIRED)
    if(TARGET OpenAL::OpenAL)
        target_link_libraries(${EXECUTABLE} PRIVATE OpenAL::OpenAL)
    else()
        target_include_directories(${EXECUTABLE} PRIVATE ${OPENAL_INCLUDE_DIR})
        target_link_libraries(${EXECUTABLE} PRIVATE ${OPENAL_LIBRARY})
        target_compile_definitions(${EXECUTABLE} PRIVATE ${OPENAL_DEFINITIONS})
    endif()
    target_compile_definitions(${EXECUTABLE} PRIVATE AUDIO_OAL)
elseif(${PROJECT}_AUDIO STREQUAL "MSS")
    find_package(MilesSDK REQUIRED)
    target_compile_definitions(${EXECUTABLE} PRIVATE AUDIO_MSS)
    target_link_libraries(${EXECUTABLE} PRIVATE MilesSDK::MilesSDK)
endif()

find_package(mpg123 REQUIRED)
target_link_libraries(${EXECUTABLE} PRIVATE
    MPG123::libmpg123
)
if(${PROJECT}_WITH_OPUS)
    find_package(opusfile REQUIRED)
    target_link_libraries(${EXECUTABLE} PRIVATE
        opusfile::opusfile
    )
    target_compile_definitions(${EXECUTABLE} PRIVATE AUDIO_OPUS)
endif()
if(${PROJECT}_WITH_LIBSNDFILE)
    find_package(SndFile REQUIRED)
    target_link_libraries(${EXECUTABLE} PRIVATE
        SndFile::SndFile
    )
    target_compile_definitions(${EXECUTABLE} PRIVATE AUDIO_OAL_USE_SNDFILE)
endif()

target_compile_definitions(${EXECUTABLE} PRIVATE )

option(${PROJECT}_WITH_SANITIZERS "Use UB sanitizers (better crash log)" OFF)
option(${PROJECT}_WITH_ASAN "Use Address sanitizer (better crash log)" OFF)

if(${PROJECT}_WITH_SANITIZERS)
    target_compile_options(${EXECUTABLE} PUBLIC
            -fsanitize=undefined,float-divide-by-zero,integer,implicit-conversion,implicit-integer-truncation,implicit-integer-arithmetic-value-change,local-bounds,nullability
            -g3 -fno-omit-frame-pointer)
    target_link_options(${EXECUTABLE} PUBLIC -fsanitize=undefined,float-divide-by-zero,integer,implicit-conversion,implicit-integer-truncation,implicit-integer-arithmetic-value-change,local-bounds,nullability)
endif()

if(${PROJECT}_WITH_ASAN)
    target_compile_options(${EXECUTABLE} PUBLIC -fsanitize=address -g3 -fno-omit-frame-pointer)
    target_link_options(${EXECUTABLE} PUBLIC -fsanitize=address)
endif()

if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU" OR CMAKE_CXX_COMPILER_ID STREQUAL "Clang" OR CMAKE_CXX_COMPILER_ID STREQUAL "AppleClang")
    target_compile_options(${EXECUTABLE}
        PRIVATE
            "-Wall"
    )
    if (NOT LIBRW_PLATFORM_PS2)
        target_compile_options(${EXECUTABLE}
            PRIVATE
                -Wextra
                -Wdouble-promotion
                -Wpedantic
        )
    endif()
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    target_compile_options(${EXECUTABLE}
        PUBLIC
            /Zc:sizedDealloc-
    )
endif()

if(NINTENDO_SWITCH)
    set(${PROJECT}_C_CXX_EXTENSIONS ON)
else()
    set(${PROJECT}_C_CXX_EXTENSIONS OFF)
endif()

if(LIBRW_PLATFORM_GL3 AND LIBRW_GL3_GFXLIB STREQUAL "GLFW")
	include(CheckSymbolExists)

	set(CMAKE_REQUIRED_LIBRARIES glfw)
	set(CMAKE_REQUIRED_DEFINITIONS -DGLFW_EXPOSE_NATIVE_X11)
	check_symbol_exists(glfwGetX11Display "GLFW/glfw3.h;GLFW/glfw3native.h" GLFW_HAS_X11)
	unset(CMAKE_REQUIRED_DEFINITIONS)
	unset(CMAKE_REQUIRED_LIBRARIES)

	if (GLFW_HAS_X11)
		find_package(X11 REQUIRED)
		target_link_libraries(${EXECUTABLE} PRIVATE X11::X11)
		target_compile_definitions(${EXECUTABLE} PRIVATE GET_KEYBOARD_INPUT_FROM_X11)
	endif (GLFW_HAS_X11)
endif()

set_target_properties(${EXECUTABLE}
    PROPERTIES
        C_STANDARD 11
        C_EXTENSIONS ${${PROJECT}_C_CXX_EXTENSIONS}
        C_STANDARD_REQUIRED ON
        CXX_STANDARD 11
        CXX_EXTENSIONS ${${PROJECT}_C_CXX_EXTENSIONS}
        CXX_STANDARD_REQUIRED ON
)

set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR})

if(${PROJECT}_INSTALL)
    install(
        TARGETS ${EXECUTABLE}
        EXPORT ${EXECUTABLE}-targets
        RUNTIME DESTINATION "."
    )
    if(MSVC)
        install(FILES $<TARGET_PDB_FILE:${EXECUTABLE}> DESTINATION "." OPTIONAL)
    endif()
endif()

reVC_platform_target(${EXECUTABLE} INSTALL)
