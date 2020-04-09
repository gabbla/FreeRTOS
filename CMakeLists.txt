cmake_minimum_required(VERSION 3.17)

project(TestFreeRTOS)

include(/home/gabbla/projects/stefano/tests/simplelink_sdk_cmake/cc26x2.cmake)

add_subdirectory(${CMAKE_SOURCE_DIR}/freertos)

add_executable(${PROJECT_NAME})

target_sources(${PROJECT_NAME}
    PRIVATE
    main.c
)

target_include_directories(${PROJECT_NAME}
    PUBLIC
    ${CMAKE_SOURCE_DIR}
)

target_link_libraries(${PROJECT_NAME}
    PRIVATE
    FreeRTOS::core
    FreeRTOS::timers
    FreeRTOS::event_groups
    FreeRTOS::stream_buffer
)
