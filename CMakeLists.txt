if(NOT DEFINED NSX_AMBIQSUITE_ROOT)
    message(FATAL_ERROR "NSX_AMBIQSUITE_ROOT must be configured before ns-ambiq-hal-r3 is added.")
endif()
if(NOT DEFINED NSX_BOARD_FLAGS_TARGET)
    message(FATAL_ERROR "Board must define NSX_BOARD_FLAGS_TARGET before ns-ambiq-hal-r3 is added.")
endif()
if(NOT DEFINED NSX_AMBIQ_PART_NAME OR NOT DEFINED NSX_AMBIQ_MCU_DIR OR NOT DEFINED NSX_AMBIQ_HAL_DIR OR NOT DEFINED NSX_AMBIQ_HAL_MCU_DIR)
    message(FATAL_ERROR "Board must define NSX_AMBIQ_* MCU/HAL paths before ns-ambiq-hal-r3 is added.")
endif()
if(NOT NSX_SDK_PROVIDER STREQUAL "ambiqsuite-r3")
    message(FATAL_ERROR "ns-ambiq-hal-r3 requires NSX_SDK_PROVIDER=ambiqsuite-r3.")
endif()

option(NSX_AMBIQSUITE_USE_PREBUILT "Use prebuilt AmbiqSuite static libraries" ON)
option(NSX_AMBIQSUITE_USE_SOURCE_FALLBACK "Use local source fallback if prebuilt is disabled" ON)

set(NSX_AMBIQ_INSTALL_BASE "${CMAKE_INSTALL_INCLUDEDIR}/nsx/ambiqsuite/${NSX_AMBIQSUITE_VERSION}")
set(NSX_HAL_LIB_DEFAULT "${NSX_AMBIQSUITE_ROOT}/lib/${NSX_AMBIQ_PART_NAME}/libam_hal.a")
set(NSX_HAL_LIB "${NSX_HAL_LIB_DEFAULT}" CACHE FILEPATH "Path to AmbiqSuite HAL static library")

set(NSX_AMBIQ_HAL_BUILD_INCLUDES
    ${NSX_AMBIQSUITE_ROOT}/CMSIS/AmbiqMicro/Include
    ${NSX_AMBIQSUITE_ROOT}/devices
    ${NSX_AMBIQ_MCU_DIR}
    ${NSX_AMBIQ_HAL_DIR}
    ${NSX_AMBIQ_HAL_MCU_DIR}
    ${NSX_AMBIQSUITE_ROOT}/utils
)

set(NSX_AMBIQ_HAL_INSTALL_INCLUDES
    ${NSX_AMBIQ_INSTALL_BASE}/CMSIS/AmbiqMicro/Include
    ${NSX_AMBIQ_INSTALL_BASE}/devices
    ${NSX_AMBIQ_INSTALL_BASE}/mcu/${NSX_AMBIQ_PART_NAME}
    ${NSX_AMBIQ_INSTALL_BASE}/mcu/${NSX_AMBIQ_PART_NAME}/hal
    ${NSX_AMBIQ_INSTALL_BASE}/mcu/${NSX_AMBIQ_PART_NAME}/hal/mcu
    ${NSX_AMBIQ_INSTALL_BASE}/utils
)

set(NSX_AMBIQ_HAL_INCLUDE_ARGS "")
foreach(p IN LISTS NSX_AMBIQ_HAL_BUILD_INCLUDES)
    list(APPEND NSX_AMBIQ_HAL_INCLUDE_ARGS "$<BUILD_INTERFACE:${p}>")
endforeach()
foreach(p IN LISTS NSX_AMBIQ_HAL_INSTALL_INCLUDES)
    list(APPEND NSX_AMBIQ_HAL_INCLUDE_ARGS "$<INSTALL_INTERFACE:${p}>")
endforeach()

add_library(nsx_ambiq_hal STATIC
    ${NSX_AMBIQSUITE_ROOT}/src/am_util_stdio.c
    ${NSX_AMBIQSUITE_ROOT}/src/am_util_delay.c
)
set_target_properties(nsx_ambiq_hal PROPERTIES EXPORT_NAME ambiq_hal_r3)
add_library(nsx::ambiq_hal ALIAS nsx_ambiq_hal)
add_library(nsx::ambiq_hal_r3 ALIAS nsx_ambiq_hal)
target_link_libraries(nsx_ambiq_hal PUBLIC
    ${NSX_BOARD_FLAGS_TARGET}
    nsx::cmsis_core
)
target_include_directories(nsx_ambiq_hal PUBLIC ${NSX_AMBIQ_HAL_INCLUDE_ARGS})
target_compile_definitions(nsx_ambiq_hal PUBLIC NSX_AMBIQSUITE_R3)

if(NSX_AMBIQSUITE_USE_PREBUILT)
    nsx_validate_prebuilt_abi()
    nsx_assert_file_exists("${NSX_HAL_LIB}")
    target_link_libraries(nsx_ambiq_hal PUBLIC "${NSX_HAL_LIB}")
elseif(NSX_AMBIQSUITE_USE_SOURCE_FALLBACK)
    message(FATAL_ERROR "ns-ambiq-hal-r3 source fallback is not implemented yet.")
else()
    message(FATAL_ERROR "Prebuilt disabled and source fallback disabled for ns-ambiq-hal-r3.")
endif()

install(DIRECTORY
    ${NSX_AMBIQSUITE_ROOT}/CMSIS/AmbiqMicro/Include
    DESTINATION ${NSX_AMBIQ_INSTALL_BASE}/CMSIS/AmbiqMicro
)
install(DIRECTORY
    ${NSX_AMBIQSUITE_ROOT}/devices
    DESTINATION ${NSX_AMBIQ_INSTALL_BASE}
)
install(DIRECTORY
    ${NSX_AMBIQSUITE_ROOT}/mcu/${NSX_AMBIQ_PART_NAME}
    DESTINATION ${NSX_AMBIQ_INSTALL_BASE}/mcu
)
install(FILES
    ${NSX_AMBIQSUITE_ROOT}/mcu/am_sdk_version.h
    DESTINATION ${NSX_AMBIQ_INSTALL_BASE}/mcu
)
install(DIRECTORY
    ${NSX_AMBIQSUITE_ROOT}/utils
    DESTINATION ${NSX_AMBIQ_INSTALL_BASE}
)

install(TARGETS
    nsx_ambiq_hal
    EXPORT nsxTargets
)
