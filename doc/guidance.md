# Mamba guidance system

## Components

- Pixhawk 1 hw version 2.4.5
- A pair of SiK telemetry radios
- GPS with compass u-blox LEA-6H with shielding
- Hobbyking Skywalker Quattro 25Ax4 (4 in 1 ESC for 2-4S LiPo) with ubec 3A@5.25V
- Power module
- Safety switch
- Piezo buzzer

## Wiring

![pixhawk wiring](../img/pixhawk.jpg)

## Custom mods

- The connector of my safety switch broke. While I fix it I have set the `Circuit breaker for IO safety` (safety switch) from 0 (enable) to 22027 (disabled)

## Troubleshooting

### GPS Power issue

There seems to be a problem with the GPS port. If I power the Pixhawk with the GPS connected, the VCC only supplies around 1V. Seems to be a problem with some of the [clones](https://www.rcgroups.com/forums/showthread.php?2472499-PIXHAWK-not-powering-GPS). If I power up the Pixhawk **and then** connect the GPS everything works fine and thn VCC supplies and maintains around 5.2V. It looks like if at boot time the GPS is drawing too much current from this line (VDD peripheral) and the line is disabled. 

If the power comes from a serial with VDD brick line (TELEM1 for instance) there is ho issue. GPS obtains a 3D lock and all works normal.

Workarounds:
	- Plug the GPS after booting
