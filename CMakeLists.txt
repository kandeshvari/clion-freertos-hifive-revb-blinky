cmake_minimum_required(VERSION 3.16)
project(blinky-demo)

set(CMAKE_C_STANDARD 11)
#set(CMAKE_VERBOSE_MAKEFILE 1)

enable_language(ASM)

set(COMPILER_PREFIX /opt/riscv/bin/riscv64-unknown-elf)

SET(CMAKE_ASM_COMPILER ${COMPILER_PREFIX}-gcc)
SET(CMAKE_C_COMPILER "${COMPILER_PREFIX}-gcc")
SET(CMAKE_AR "${COMPILER_PREFIX}-gcc-ar")
SET(CMAKE_LINKER "${COMPILER_PREFIX}-gcc-ld")
SET(CMAKE_NM "${COMPILER_PREFIX}-gcc-nm")
SET(CMAKE_OBJDUMP "${COMPILER_PREFIX}-gcc-objdump")
SET(CMAKE_RANLIB "${COMPILER_PREFIX}-gcc-ranlib")


add_subdirectory(freedom-metal)

add_executable(blinky-freertos-demo
        freertos/portable/GCC/RISC-V/port.c
        freertos/portable/GCC/RISC-V/portASM.S
        freertos/portable/MemMang/heap_4.c
        freertos/croutine.c
        freertos/event_groups.c
        freertos/list.c
        freertos/queue.c
        freertos/stream_buffer.c
        freertos/tasks.c
        freertos/timers.c
        src/blinky_demo/main_blinky.c
        src/main.c
        )

target_include_directories(blinky-freertos-demo
        PUBLIC
        src
        freertos/include
        freertos/portable/GCC/RISC-V
        freertos/portable/GCC/RISC-V/chip_specific_extensions/RV32I_CLINT_no_extensions
#        ../bsp/install/include

        )

target_compile_options(blinky-freertos-demo
        PRIVATE
        -O3 -Wall -Wextra -c -fmessage-length=0
        -march=rv32imac -mabi=ilp32 -mcmodel=medlow
        -DportasmHANDLE_INTERRUPT=handle_trap
        -ffunction-sections -fdata-sections --specs=nano.specs -Wno-unused-parameter
        )

target_link_options(blinky-freertos-demo
        PRIVATE
        -march=rv32imac
        -mabi=ilp32
        -mcmodel=medlow
        -T${CMAKE_CURRENT_SOURCE_DIR}/bsp/metal.default.lds

        -nostartfiles
        -Xlinker
        --gc-sections
        -Wl,-Map,RTOSDemo.map
        -Wl,--start-group
        -Wl,--end-group
        --specs=nano.specs

        )

target_link_libraries(blinky-freertos-demo
        PRIVATE
        metal-lib
        )

add_custom_command(TARGET blinky-freertos-demo POST_BUILD
        COMMAND ${COMPILER_PREFIX}-objcopy
        ARGS -O ihex ${CMAKE_BINARY_DIR}/blinky-freertos-demo ${CMAKE_BINARY_DIR}/blinky-freertos-demo.hex
        COMMENT "Generating HEX image: ${CMAKE_BINARY_DIR}/blinky-freertos-demo.hex"
        VERBATIM
        )

add_custom_command(TARGET blinky-freertos-demo POST_BUILD
        COMMAND ${COMPILER_PREFIX}-size
        ARGS ${CMAKE_BINARY_DIR}/blinky-freertos-demo
        COMMENT "Show ELF size"
        VERBATIM
        )

