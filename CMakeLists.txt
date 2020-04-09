cmake_minimum_required(VERSION 3.17)

project(FreeRTOS)

# FreeRTOS version
set(PROJECT_MAJOR_VERSION 10)
set(PROJECT_MINOR_VERSION 3)
set(PROJECT_PATCH_VERSION 1)
set(PROJECT_VERSION
    ${PROJECT_MAJOR_VERSION}.${PROJECT_MINOR_VERSION}.${PROJECT_PATCH_VERSION})

# Building using GCC
set(FREERTOS_COMPILER "GCC" CACHE STRING "FreeRTOS compiler")

set(FREERTOS_ARCH "ARM_CM4F")

# Target details
IF(NOT FREERTOS_ARCH)
    message(ERROR "You must define the target arch!")
ENDIF()

# Heap allocator, see https://www.freertos.org/a00111.html
IF(NOT FREERTOS_HEAP_ALLOCATOR)
    set(FREERTOS_HEAP_ALLOCATOR 2 CACHE INT "FreeRTOS heap allocator")
ELSEIF(FREERTOS_HEAP_ALLOCATOR LESS 1 OR FREERTOS_HEAP_ALLOCATOR GREATER 5)
    message(FATAL_ERROR "Invalid heap allocator n ${FREERTOS_HEAP_ALLOCATOR}")
ENDIF()

IF(NOT EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/FreeRTOS/Source/portable/${FREERTOS_COMPILER}/${FREERTOS_ARCH})
    message(FATAL_ERROR "Wrong FREERTOS_COMPILER or FREERTOS_ARCH")
ENDIF()

# Config dir
IF(NOT FREERTOS_CONFIG_DIR OR NOT EXISTS ${FREERTOS_CONFIG_DIR}/FreeRTOSConfig.h)
    message(FATAL_ERROR "You must provide a configuration file!")
ENDIF()

message(STATUS "Building FreeRTOS version ${PROJECT_VERSION}")
message(STATUS "Building FreeRTOS with ${FREERTOS_COMPILER}")
message(STATUS "Building FreeRTOS for ${FREERTOS_ARCH}")
message(STATUS "Using FreeRTOS heap allocator nr ${FREERTOS_HEAP_ALLOCATOR}")

################################################################################
# FreeRTOS core
################################################################################
add_library(FreeRTOS OBJECT)
add_library(FreeRTOS::core ALIAS FreeRTOS)

target_sources(FreeRTOS
    PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/FreeRTOS/Source/tasks.c
    ${CMAKE_CURRENT_SOURCE_DIR}/FreeRTOS/Source/queue.c
    ${CMAKE_CURRENT_SOURCE_DIR}/FreeRTOS/Source/list.c
    ${CMAKE_CURRENT_SOURCE_DIR}/FreeRTOS/Source/portable/${FREERTOS_COMPILER}/${FREERTOS_ARCH}/port.c
    ${CMAKE_CURRENT_SOURCE_DIR}/FreeRTOS/Source/portable/MemMang/heap_${FREERTOS_HEAP_ALLOCATOR}.c
)

target_include_directories(FreeRTOS
    PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/FreeRTOS/Source/include
    ${CMAKE_CURRENT_SOURCE_DIR}/FreeRTOS/Source/portable/${FREERTOS_COMPILER}/${FREERTOS_ARCH}
    ${FREERTOS_CONFIG_DIR}
)

################################################################################
# FreeRTOS timer
################################################################################
add_library(FreeRTOSTimers OBJECT)
add_library(FreeRTOS::timers ALIAS FreeRTOSTimers)

target_sources(FreeRTOSTimers
    PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/FreeRTOS/Source/timers.c
)

target_link_libraries(FreeRTOSTimers
    PUBLIC
    FreeRTOS::core
)

################################################################################
# FreeRTOS event groups
################################################################################
add_library(FreeRTOSEvtGroups OBJECT)
add_library(FreeRTOS::event_groups ALIAS FreeRTOSEvtGroups)

target_sources(FreeRTOSEvtGroups
    PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/FreeRTOS/Source/event_groups.c
)

target_link_libraries(FreeRTOSEvtGroups
    PUBLIC
    FreeRTOS::core
)

################################################################################
# FreeRTOS stream buffer
################################################################################
add_library(FreeRTOSStreamBuffer OBJECT)
add_library(FreeRTOS::stream_buffer ALIAS FreeRTOSStreamBuffer)

target_sources(FreeRTOSStreamBuffer
    PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/FreeRTOS/Source/stream_buffer.c
)

target_link_libraries(FreeRTOSStreamBuffer
    PUBLIC
    FreeRTOS::core
)
