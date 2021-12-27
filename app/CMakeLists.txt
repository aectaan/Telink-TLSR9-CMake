cmake_minimum_required(VERSION 3.0)

project(apptest)

enable_language(C ASM)

set(CMAKE_VERBOSE_MAKEFILE ON)
set(CMAKE_SHARED_LIBRARY_LINK_C_FLAGS)

set(RISCV_TOOLCHAIN    ${CMAKE_CURRENT_LIST_DIR}/../toolchain/nds32le-elf-mculib-v5f/bin)
set(DRIVER_SDK         ${CMAKE_CURRENT_LIST_DIR}/../driver_sdk/B91_Driver_Demo)

set(CMAKE_SYSTEM_NAME  Generic)
set(CMAKE_C_COMPILER   ${RISCV_TOOLCHAIN}/riscv32-elf-gcc)
set(CMAKE_CXX_COMPILER ${RISCV_TOOLCHAIN}/riscv32-elf-g++)
set(CMAKE_ASM_COMPILER ${RISCV_TOOLCHAIN}/riscv32-elf-gcc)

set(OBJCOPY            ${RISCV_TOOLCHAIN}/riscv32-elf-objcopy)
set(OBJDUMP            ${RISCV_TOOLCHAIN}/riscv32-elf-objdump)
set(NM                 ${RISCV_TOOLCHAIN}/riscv32-elf-nm)
set(READELF            ${RISCV_TOOLCHAIN}/riscv32-elf-readelf)
set(SIZE               ${RISCV_TOOLCHAIN}/riscv32-elf-size)


add_definitions(-DMCU_STARTUP_FLASH_B91=1)

add_compile_options(-Og -g3 -mcpu=d25f -mext-dsp -mabi=ilp32f -std=c99)
add_compile_options(-ffunction-sections -fdata-sections -fpack-struct -fshort-enums -flto)
add_compile_options(-fmessage-length=0 -fomit-frame-pointer -fno-strict-aliasing -fshort-wchar -fuse-ld=bfd)
add_compile_options(-Wall -Wno-nonnull-compare -Wextra -Wshadow -Werror -Wno-gnu-zero-variadic-macro-arguments)

set(CMAKE_EXE_LINKER_FLAGS "-Og -g3 -nostartfiles -static -T\"${DRIVER_SDK}/link/flash_boot.link\" -Wl,--gc-sections")

link_libraries(-lB91)

aux_source_directory(${DRIVER_SDK}/common             SRCS_COMMON)
aux_source_directory(${DRIVER_SDK}/drivers/B91        SRCS_DRIVERS_B91)
aux_source_directory(${DRIVER_SDK}/vendor/common      SRCS_VENDOR_COMMON)
aux_source_directory(${DRIVER_SDK}/vendor/UART_DEMO   SRCS_APP)

set(SRCS
    ${SRCS_COMMON}
    ${SRCS_DRIVERS_B91}
    ${SRCS_VENDOR_COMMON}
    ${SRCS_APP}
    ${DRIVER_SDK}/boot/B91/cstartup_b91_flash.S)

include_directories(${CMAKE_CURRENT_LIST_DIR})
include_directories(${DRIVER_SDK})
include_directories(${DRIVER_SDK}/vendor/common)
include_directories(${DRIVER_SDK}/drivers/B91)
include_directories(${DRIVER_SDK}/common)

link_directories(${DRIVER_SDK}/drivers/B91)

add_executable(${PROJECT_NAME}.elf ${SRCS})

add_custom_target(BIN ALL
    COMMAND ${OBJCOPY} -S -O binary ${PROJECT_NAME}.elf ${PROJECT_NAME}.bin
    COMMAND ${SIZE} ${PROJECT_NAME}.elf
    COMMAND ${OBJDUMP} -x -d -C  ${PROJECT_NAME}.elf > objdump.txt
    COMMAND ${NM} -n -l -C ${PROJECT_NAME}.elf > symbol.txt
    COMMAND ${READELF} -a ${PROJECT_NAME}.elf  > readelf.txt
    DEPENDS ${PROJECT_NAME}.elf
)