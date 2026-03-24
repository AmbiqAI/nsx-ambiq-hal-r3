# nsx-ambiq-hal-r3

`nsx-ambiq-hal-r3` exposes the AmbiqSuite R3 HAL surface used by NSX.

This wrapper module does not own a full SDK copy. It expects an active
`nsx-ambiqsuite-r3` provider and publishes:
- CMSIS and device include paths
- MCU and HAL include paths
- the prebuilt `libam_hal.a` link dependency
- minimal utility sources required by NSX apps
