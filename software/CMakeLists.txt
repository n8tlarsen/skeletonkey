#-------------------------------------------------------------------------------
# Download DFP
#-------------------------------------------------------------------------------
set(pack_dir "${CMAKE_CURRENT_SOURCE_DIR}/pack/Microchip.SAMD21_DFP.3.5.132")
set(pack_file "${pack_dir}/Microchip.SAMD21_DFP.3.5.132.atpack")
if(NOT EXISTS ${pack_file})
    execute_process(COMMAND curl https://packs.download.microchip.com/Microchip.SAMD21_DFP.3.5.132.atpack --create-dirs --output "${pack_file}" 
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    )
    execute_process(COMMAND tar -xf ${pack_file}
        WORKING_DIRECTORY ${pack_dir}
    )
endif()

#-------------------------------------------------------------------------------
# Part Specific Compiler Options
#-------------------------------------------------------------------------------
set(target cortex-m0plus)
set(arch   armv6-m)
set(part   ATSAMD21G15B)

set(TARGET_FLAGS "-mcpu=${target} -mtune=${target} -march=${arch} -mthumb -D${part}=1")
set(C_CXX_FLAGS  "-Wall -mno-sched-prolog -fno-builtin -ffunction-sections \
-fdata-sections -fno-exceptions -fno-unwind-tables --specs=nosys.specs")
set(CMAKE_EXE_LINKER_FLAGS  "-T ${CMAKE_CURRENT_SOURCE_DIR}/${PROJECT_NAME}.ld -Xlinker --gc-sections -Xlinker -Map=${PROJECT_NAME}.map --specs=nano.specs")

#-------------------------------------------------------------------------------
# Target Executable
#-------------------------------------------------------------------------------
add_executable(${PROJECT_NAME}.elf
    ${pack_dir}/samd21b/gcc/gcc/startup_samd21g15b.c
    ${pack_dir}/samd21b/gcc/system_samd21g15b.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/main.cpp
)

target_include_directories(${PROJECT_NAME}.elf
    PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/CMSIS/CMSIS/Core/Include
    PUBLIC ${pack_dir}/samd21b/include
    PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/src
)

add_custom_command(TARGET ${PROJECT_NAME}.elf
    DEPENDS ${PROJECT_NAME}.elf
    COMMAND ${CMAKE_SIZE} ARGS "${PROJECT_NAME}.elf" -A
)

set(MCU_BIN "${PROJECT_NAME}-${PROJECT_VERSION}.bin")
add_custom_command(TARGET ${PROJECT_NAME}.elf
    COMMAND ${CMAKE_OBJCOPY} -O binary ${PROJECT_NAME}.elf ${MCU_BIN}
    DEPENDS ${PROJECT_NAME}.elf
    BYPRODUCTS
        ${MCU_BIN}
)

set(MCU_HEX "${PROJECT_NAME}-${PROJECT_VERSION}.hex")
add_custom_command(TARGET ${PROJECT_NAME}.elf
    COMMAND ${CMAKE_OBJCOPY} -O ihex ${PROJECT_NAME}.elf ${MCU_HEX}
    DEPENDS ${PROJECT_NAME}.elf
    BYPRODUCTS
        ${MCU_HEX}
)