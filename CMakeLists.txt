cmake_minimum_required(VERSION 3.20)

project(Tilengine)


file(GLOB SOURCES
     "src/*.c"
)


# main library
list(REMOVE_ITEM SOURCES "src/Test.c")
add_library(${PROJECT_NAME} ${SOURCES})
add_library(tilengine::Tilengine ALIAS ${PROJECT_NAME})

if (UNIX AND NOT APPLE)
	set(CMAKE_C_FLAGS "-m64 -msse2")
elseif (UNIX AND APPLE)
    target_link_libraries(${PROJECT_NAME} PRIVATE "framework")
endif()

if(ENABLE_UBSAN)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fno-omit-frame-pointer -fsanitize=undefined")
    set(CMAKE_LINKER_FLAGS "${CMAKE_LINKER_FLAGS} -fno-omit-frame-pointer -fsanitize=undefined")
endif()

if(ENABLE_ASAN)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fno-omit-frame-pointer -fsanitize=address")
    set(CMAKE_LINKER_FLAGS "${CMAKE_LINKER_FLAGS} -fno-omit-frame-pointer -fsanitize=address")
endif()

target_link_libraries(${PROJECT_NAME} PRIVATE "c")
target_link_libraries(${PROJECT_NAME} PRIVATE "z")
target_link_libraries(${PROJECT_NAME} PRIVATE "png")

find_package(SDL2 REQUIRED)
target_link_libraries(${PROJECT_NAME} PRIVATE ${SDL2_LIBRARIES})

target_include_directories(${PROJECT_NAME} PUBLIC include)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99 -O2 -fpic -DLIB_EXPORTS")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-unused-result -Wno-format-truncation")

file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/include/Tilengine.h
     DESTINATION ${CMAKE_CURRENT_BINARY_DIR})

# test executable
add_executable(Test src/Test.c)
target_include_directories(Test PRIVATE include)
target_link_libraries(Test PRIVATE "Tilengine")
target_link_libraries(Test PRIVATE "c")
target_link_libraries(Test PRIVATE "z")
target_link_libraries(Test PRIVATE "png")
target_link_libraries(Test PRIVATE "m")

# samples
add_subdirectory(samples)