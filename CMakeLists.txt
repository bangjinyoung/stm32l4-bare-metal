cmake_minimum_required(VERSION 3.22.1)

enable_language(C ASM)
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS ON)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Edit project name
project(stm32l4-bare-metal)

# Edit CPU options
set(CPU_OPTIONS
        -mcpu=cortex-m4
        -mthumb
        -mfpu=fpv4-sp-d16
        -mfloat-abi=hard
)

# Edit Link script file
set(LINK_SCRIPT ${CMAKE_CURRENT_SOURCE_DIR}/STM32L475VGTx_FLASH.ld)


# Add C source files
file(GLOB C_SOURCES CONFIGURE_DEPENDS
        # Add project source files
        src/*.c

        # Add STM32 HAL source files
        lib/stm32l4_hal/Src/*.c
        lib/cmsis_l4/Source/Templates/system_stm32l4xx.c
)

# Add assembly files
set(ASM_SOURCES
        lib/cmsis_l4/Source/Templates/gcc/startup_stm32l475xx.s
)

set(EXECUTABLE ${PROJECT_NAME}.elf)

add_executable(${EXECUTABLE} ${C_SOURCES} ${ASM_SOURCES})

# Add defines
target_compile_definitions(${EXECUTABLE} PRIVATE
        USE_HAL_DRIVER
        STM32L475xx
)

# Add includes paths
target_include_directories(${EXECUTABLE} PRIVATE
        # Add project include paths
        src

        # Add CMSIS include paths
        lib/cmsis_5/CMSIS/Core/Include

        # Add STM32 HAL include paths
        lib/cmsis_l4/Include
        lib/stm32l4_hal/Inc
        lib/stm32l4_hal/Inc/Legacy
)

# Add compile options
target_compile_options(${EXECUTABLE} PRIVATE
        ${CPU_OPTIONS}
        
        $<$<CONFIG:Debug>:-Og -g3 -ggdb>
        $<$<CONFIG:Release>:-Og -g0>

        -Wall
        -fdata-sections
        -ffunction-sections
        -fdiagnostics-color=always

        $<$<COMPILE_LANGUAGE:ASM>:-x assembler-with-cpp>

        --specs=nano.specs
)

# Add link options
target_link_options(${EXECUTABLE} PRIVATE
        ${CPU_OPTIONS}

        -T${LINK_SCRIPT}

        -Wl,-Map=${PROJECT_NAME}.map
        -Wl,--gc-sections
        -Wl,--start-group -lc -lm
        -Wl,--end-group
        -Wl,--no-warn-rwx-segments
        -Wl,--print-memory-usage

        --specs=nano.specs
        # -u _printf_float
)

# Print file size
add_custom_command(TARGET ${EXECUTABLE}
        POST_BUILD
        COMMAND ${CMAKE_SIZE_UTIL} ${EXECUTABLE})

# Output to hex and binary files
add_custom_command(TARGET ${EXECUTABLE}
        POST_BUILD
        COMMAND ${CMAKE_OBJCOPY} -O ihex ${EXECUTABLE} ${PROJECT_NAME}.hex
        COMMAND ${CMAKE_OBJCOPY} -O binary ${EXECUTABLE} ${PROJECT_NAME}.bin)