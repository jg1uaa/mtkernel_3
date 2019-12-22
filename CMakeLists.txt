cmake_minimum_required(VERSION 3.1)
project(mtkernel_3 C)

# target definition
set(target iote_m367)
set(cpu_core cpu/core/acm3)
set(soc cpu/tx03_m367)
set(linker_script etc/linker/${target}/tkernel_map.ld)

set(CMAKE_ASM_FLAGS "${CMAKE_ASM_FLAGS} -mthumb -mcpu=cortex-m3")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -mthumb -mcpu=cortex-m3")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mthumb -mcpu=cortex-m3")

# global settings
#set(CMAKE_VERBOSE_MAKEFILE ON)
enable_language(ASM)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DTARGET_DIR=${target}")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DTARGET_DIR=${target}")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_CXX_FLAGS} -L ${CMAKE_SOURCE_DIR} -T ${linker_script} --specs=nosys.specs -Wl,-Map=${CMAKE_PROJECT_NAME}.map")

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/config)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)

# libtk
set(libtk_directory lib/libtk)
file(GLOB libtk_source ${libtk_directory}/*.c)
file(GLOB libtk_cpudepend_source ${libtk_directory}/sysdepend/${cpu_core}/*.[cS])
file(GLOB libtk_socdepend_source ${libtk_directory}/sysdepend/${soc}/*.[cS])
add_library(tk STATIC ${libtk_source} ${libtk_cpudepend_source} ${libtk_socdepend_source})

# libtm
set(libtm_directory lib/libtm)
file(GLOB libtm_source ${libtm_directory}/*.c)
file(GLOB libtm_sysdepend_source ${libtm_directory}/sysdepend/${target}/*.[cS])
add_library(tm STATIC ${libtm_source} ${libtm_sysdepend_source})
target_link_libraries(tm INTERFACE tk)

# kernel
set(kernel_directory kernel)
file(GLOB kernel_inittask_source ${kernel_directory}/inittask/*.c)
file(GLOB kernel_cpudepend_source ${kernel_directory}/sysdepend/${cpu_core}/*.[cS])
file(GLOB kernel_socdepend_source ${kernel_directory}/sysdepend/${soc}/*.[cS])
file(GLOB kernel_sysdepend_source ${kernel_directory}/sysdepend/${target}/*.[cS])
file(GLOB kernel_sysinit_source ${kernel_directory}/sysinit/*.c)
file(GLOB kernel_tkernel_source ${kernel_directory}/tkernel/*.c)
file(GLOB kernel_tstdlib_source ${kernel_directory}/tstdlib/*.c)
file(GLOB kernel_usermain_source ${kernel_directory}/usermain/*.c)
add_library(kernel STATIC ${kernel_inittask_source} ${kernel_cpudepend_source} ${kernel_socdepend_source} ${kernel_sysdepend_source} ${kernel_sysinit_source} ${kernel_tkernel_source} ${kernel_tstdlib_source} ${kernel_usermain_source})
target_include_directories(kernel PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/${kernel_directory}/knlinc)
target_link_libraries(kernel INTERFACE tk tm)

# usermain
set(usermain_directory app_sample)
file(GLOB usermain_source ${usermain_directory}/*.[cS])
add_executable(${PROJECT_NAME} ${usermain_source})
target_link_libraries(${PROJECT_NAME} tk tm kernel)
