# =============================================================================
# CMake Configuration
# =============================================================================

cmake_minimum_required(VERSION 3.20)

# -----------------------------------------------------------------------------
# Build / Indexing
# -----------------------------------------------------------------------------

set(CMAKE_EXPORT_COMPILE_COMMANDS TRUE)

set(CMAKE_C_COMPILER_FORCED TRUE)
set(CMAKE_CXX_COMPILER_FORCED TRUE)


# =============================================================================
# Toolchain Configuration
# =============================================================================

include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/target.cmake)
include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/gcc-arm-none-eabi.cmake)


# =============================================================================
# Project Definition
# =============================================================================

project(563L_Economy_Display_Key_Dot_4)

enable_language(C ASM)

set(PROJECT_NAME 563L_Economy_Display_Key_Dot_4)

add_executable(${PROJECT_NAME})


# =============================================================================
# Build Target Configuration
#
# BUILD_TARGET:
#   FULL       -> Normal development build
#   BOOTLOADER -> Bootloader build
#   MAIN       -> Main firmware build
# =============================================================================

set(BUILD_TARGET "FULL" CACHE STRING "Firmware build target")

set_property(CACHE BUILD_TARGET PROPERTY STRINGS
    FULL
    BOOTLOADER
    MAIN
)


# -----------------------------------------------------------------------------
# Select Firmware Target
# -----------------------------------------------------------------------------

if(BUILD_TARGET STREQUAL "BOOTLOADER")

    set(BOOTLOADER_VALUE 1U)

    set(FLASH_ORIGIN 0x08000000)
    set(FLASH_LENGTH 6K)

elseif(BUILD_TARGET STREQUAL "MAIN")

    set(BOOTLOADER_VALUE 0U)

    set(FLASH_ORIGIN 0x08001800)
    set(FLASH_LENGTH 24K)

else()

    # FULL
    # Normal development firmware

    set(BOOTLOADER_VALUE 0U)

    set(FLASH_ORIGIN 0x08000000)
    set(FLASH_LENGTH 32K)

endif()


# =============================================================================
# Build Information
# =============================================================================

message(STATUS "========================================")
message(STATUS "Firmware Build Configuration")
message(STATUS "========================================")
message(STATUS "Build Type      : ${CMAKE_BUILD_TYPE}")
message(STATUS "Build Target    : ${BUILD_TARGET}")
message(STATUS "BOOTLOADER      : ${BOOTLOADER_VALUE}")
message(STATUS "FLASH ORIGIN    : ${FLASH_ORIGIN}")
message(STATUS "FLASH LENGTH    : ${FLASH_LENGTH}")
message(STATUS "========================================")


# =============================================================================
# Project Components
# =============================================================================

include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/flags.cmake)
include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/components.cmake)
include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/files.cmake)


# =============================================================================
# Common Source Files
# =============================================================================

target_sources(${PROJECT_NAME} PRIVATE

    # -------------------------------------------------------------------------
    # GPIO
    # -------------------------------------------------------------------------

    Drivers/gpio/gpio.c
    Drivers/gpio/gpio_hal.c


    # -------------------------------------------------------------------------
    # TimeCore
    # -------------------------------------------------------------------------

    Drivers/timecore/timecore.c
    Drivers/timecore/timecore_hal.c


    # -------------------------------------------------------------------------
    # System Clock
    # -------------------------------------------------------------------------

    Drivers/systemclock/systemclock.c


    # -------------------------------------------------------------------------
    # UART
    # -------------------------------------------------------------------------

    Drivers/uart/uart_driver.c
    HAL/uart/uart_hal.c


    # -------------------------------------------------------------------------
    # LED
    # -------------------------------------------------------------------------

    Drivers/led/led.c


    # -------------------------------------------------------------------------
    # Libraries
    # -------------------------------------------------------------------------

    Libraries/containers/RingBuffer/ring_buffer.c
    Libraries/crc/crc.c

    # -------------------------------------------------------------------------
    # Application
    # -------------------------------------------------------------------------

    App/app.c

)


# =============================================================================
# Conditional Source Files
# =============================================================================

if(BOOTLOADER_VALUE EQUAL 1)

    target_sources(${PROJECT_NAME} PRIVATE

        # ---------------------------------------------------------------------
        # Bootloader Core
        # ---------------------------------------------------------------------

        Bootloader/Core/bootloader.c

        # ---------------------------------------------------------------------
        # Bootloader Protocol
        # ---------------------------------------------------------------------

        Bootloader/Protocol/bl_protocol.c

        # ---------------------------------------------------------------------
        # Bootloader Transport
        # ---------------------------------------------------------------------

        Bootloader/Transport/bl_transport.c
        Bootloader/Transport/uart/bootloader_uart.c

        # ---------------------------------------------------------------------
        # Bootloader Flash
        # ---------------------------------------------------------------------

        Bootloader/Flash/bl_flash.c

        # ---------------------------------------------------------------------
        # Bootloader Application
        # ---------------------------------------------------------------------

        Bootloader/bootloader_app.c

        # ---------------------------------------------------------------------
        # Bootloader Metadata
        # ---------------------------------------------------------------------

        Bootloader/metadata/bl_metadata.c

        # ---------------------------------------------------------------------
        # Bootloader Jump
        # ---------------------------------------------------------------------

        Bootloader/jump/bl_jump.c

    ) 

else()

    target_sources(${PROJECT_NAME} PRIVATE

        # ---------------------------------------------------------------------
        # Main Application
        # ---------------------------------------------------------------------

        Firmware/firmware.c
        App/app.c


        # -------------------------------------------------------------------------
        # Logger
        # -------------------------------------------------------------------------

        Services/logger/core/logger.c
        Services/logger/core/logger_hal.c
        Services/logger/backends/uart/logger_uart.c


    )

endif()


# =============================================================================
# Include Directories
# =============================================================================

target_include_directories(${PROJECT_NAME} PRIVATE

    # -------------------------------------------------------------------------
    # Core / CMSIS
    # -------------------------------------------------------------------------

    ${CMAKE_CURRENT_SOURCE_DIR}/CMSIS


    # -------------------------------------------------------------------------
    # Application / Drivers
    # -------------------------------------------------------------------------

    ${CMAKE_CURRENT_SOURCE_DIR}/App
    ${CMAKE_CURRENT_SOURCE_DIR}/Drivers
    ${CMAKE_CURRENT_SOURCE_DIR}/BSP


    # -------------------------------------------------------------------------
    # Services
    # -------------------------------------------------------------------------

    ${CMAKE_CURRENT_SOURCE_DIR}/Services


    # -------------------------------------------------------------------------
    # Configuration
    # -------------------------------------------------------------------------

    ${CMAKE_CURRENT_SOURCE_DIR}/Config


    # -------------------------------------------------------------------------
    # Libraries
    # -------------------------------------------------------------------------

    ${CMAKE_CURRENT_SOURCE_DIR}/Libraries


    # -------------------------------------------------------------------------
    # HAL
    # -------------------------------------------------------------------------

    ${CMAKE_CURRENT_SOURCE_DIR}/HAL


    # -------------------------------------------------------------------------
    # Bootloader
    # -------------------------------------------------------------------------

    ${CMAKE_CURRENT_SOURCE_DIR}/Bootloader

    # -------------------------------------------------------------------------
    # Firmware
    # -------------------------------------------------------------------------

    ${CMAKE_CURRENT_SOURCE_DIR}/Firmware

)
 

# =============================================================================
# Compiler Options
# =============================================================================

target_compile_options(${PROJECT_NAME} PRIVATE

    -flto

)


# =============================================================================
# Compiler Definitions
# =============================================================================

target_compile_definitions(${PROJECT_NAME} PRIVATE

    BOOTLOADER=${BOOTLOADER_VALUE}
    STM32G030xx

)


# =============================================================================
# Linker Script Configuration
#
# One linker template is used for all configurations.
# CMake generates the actual linker script inside the build directory.
# =============================================================================

set(LINKER_SCRIPT
    ${CMAKE_CURRENT_SOURCE_DIR}/stm32g030x6_flash.ld
)

configure_file(
    ${LINKER_SCRIPT}
    ${CMAKE_BINARY_DIR}/stm32g030x6_flash.ld
    @ONLY
)


# =============================================================================
# Linker Options
# =============================================================================

target_link_options(${PROJECT_NAME} PRIVATE

    -flto
    "-T${CMAKE_BINARY_DIR}/stm32g030x6_flash.ld"

)


# =============================================================================
# Linker Directories
# =============================================================================

target_link_directories(${PROJECT_NAME} PRIVATE

    # User defined linker search directories

)


# =============================================================================
# Linker Libraries
# =============================================================================

target_link_libraries(${PROJECT_NAME} PRIVATE

    # User defined linker libraries

)


# =============================================================================
# BIN and HEX Generator
# =============================================================================

add_custom_command(
    TARGET ${PROJECT_NAME}
    POST_BUILD

    # Generate .bin
    COMMAND arm-none-eabi-objcopy
            -O binary
            $<TARGET_FILE:${PROJECT_NAME}>
            ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.bin

    # Generate .hex
    COMMAND arm-none-eabi-objcopy
            -O ihex
            $<TARGET_FILE:${PROJECT_NAME}>
            ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.hex

    COMMENT "Generating ${PROJECT_NAME}.bin and ${PROJECT_NAME}.hex"

    VERBATIM
)
