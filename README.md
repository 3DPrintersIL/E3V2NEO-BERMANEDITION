# E3V2NEO-BERMANEDITION

# A full config of my modded Ender 3 V2 Neo

Printer.cfg - 

-----------------------------------------------------------------------------------

[virtual_sdcard]
path: ~/gcode_files
 
[include fluidd.cfg]

[bltouch]
sensor_pin: ^PC14
control_pin: PA1
x_offset: -31
y_offset: -41
#z_offset: 219
speed: 20
samples: 1
sample_retract_dist: 8.0
pin_move_time: 0.4
probe_with_touch_mode: True
pin_up_touch_mode_reports_triggered: False
stow_on_each_sample: True

[safe_z_home]
home_xy_position: 149,166 # Change coordinates to the center of your print bed
speed: 200
z_hop: 10               # Move up 10mm
z_hop_speed: 10

[bed_mesh]
speed: 200
horizontal_move_z: 4
mesh_min: 20,14         #need to handle head distance with bl_touch
mesh_max: 215,187     #max probe range
probe_count: 6,6
mesh_pps: 2,2
fade_start: 1
fade_end: 10
fade_target: 0

[stepper_x]
step_pin: PB13
dir_pin: !PB12
enable_pin: !PB14
microsteps: 16
rotation_distance: 40
endstop_pin: ^PC0
position_max: 247
position_endstop: -5
position_min: -5
homing_speed: 50

[tmc2209 stepper_x]
uart_pin: PC11
tx_pin: PC10
uart_address: 0
run_current: 0.580
stealthchop_threshold: 999999

[stepper_y]
step_pin: PB10
dir_pin: !PB2
enable_pin: !PB11
microsteps: 16
rotation_distance: 40
endstop_pin: ^PC1
position_endstop: -2
position_min: -2
position_max: 228
homing_speed: 50

[tmc2209 stepper_y]
uart_pin: PC11
tx_pin: PC10
uart_address: 2
run_current: 0.580
stealthchop_threshold: 999999

[stepper_z]
step_pin: PB0
dir_pin: PC5
enable_pin: !PB1
microsteps: 16
rotation_distance: 8
endstop_pin: probe:z_virtual_endstop
#position_endstop: 0.0
position_max: 250
position_min: -6
homing_speed: 4
second_homing_speed: 1
homing_retract_dist: 2.0

[screws_tilt_adjust]
screw1: 70, 40
screw1_name: front left screw
screw2: 240, 40
screw2_name: front right screw
screw3: 240, 212
screw3_name: rear right screw
screw4: 70, 212
screw4_name: rear left screw
horizontal_move_z: 10
speed: 200
screw_thread: CW-M4

[tmc2209 stepper_z]
uart_pin: PC11
tx_pin: PC10
uart_address: 1
run_current: 0.580
stealthchop_threshold: 999999

[extruder]
step_pin: PB3
dir_pin: !PB4
enable_pin: !PD2
microsteps: 16
rotation_distance: 7.69261056
nozzle_diameter: 0.400
filament_diameter: 1.750
heater_pin: PC8
sensor_type: EPCOS 100K B57560G104F
sensor_pin: PA0
#control: pid
#pid_Kp: 21.527
#pid_Ki: 1.063
#pid_Kd: 108.982
min_temp: 0
max_temp: 250

[tmc2209 extruder]
uart_pin: PC11
tx_pin: PC10
uart_address: 3
run_current: 0.650
stealthchop_threshold: 999999

[heater_bed]
heater_pin: PC9
sensor_type: ATC Semitec 104GT-2
sensor_pin: PC3
#control: pid
#pid_Kp: 54.027
#pid_Ki: 0.770
#pid_Kd: 948.182
min_temp: 0
max_temp: 130

[heater_fan heatbreak_cooling_fan]
pin: PC7

[fan]
pin: PC6

[mcu]
serial:/dev/serial/by-id/usb-Klipper_stm32f103xe_33FFD5055356343527571543-if00
restart_method: command


[printer]
kinematics: cartesian
max_velocity: 300
max_accel: 3000
max_z_velocity: 5
max_z_accel: 100

[display]
lcd_type: st7920
cs_pin: EXP1_7
sclk_pin: EXP1_6
sid_pin: EXP1_8
encoder_pins: ^EXP1_5, ^EXP1_3
click_pin: ^!EXP1_2

[output_pin beeper]
pin: EXP1_1

[static_digital_output usb_pullup_enable]
pins: !PA14

[temperature_sensor SKR]
sensor_type: temperature_mcu

[temperature_sensor RPI]
sensor_type: temperature_host

[board_pins]
aliases:
    # EXP1 header
    EXP1_1=PB5,  EXP1_3=PA9,   EXP1_5=PA10, EXP1_7=PB8,  EXP1_9=<GND>,
    EXP1_2=PA15, EXP1_4=<RST>, EXP1_6=PB9,  EXP1_8=PB15, EXP1_10=<5V>



[gcode_macro PRINT_START]
description: starts the print
gcode:

     #Get Bed and Extruder temperature from Slicer GCode
    {% set BED_TEMP = params.BED_TEMP|default(60)|float %}
    {% set EXTRUDER_TEMP = params.EXTRUDER_TEMP|default(200)|float %}
    #Preheat nozzle and bed
    M104 S{EXTRUDER_TEMP} T0                        
    M140 S{BED_TEMP} 

    M106 S255 #part fan full speed
    G92 E0 ; Reset Extruder
    G28 X 
    G28 Y 

    M190 S{BED_TEMP} 
    G28 Z                              
    M109 S{EXTRUDER_TEMP} T0


    #Heat nozzle and bed
   
  BED_MESH_CALIBRATE
  BED_MESH_PROFILE LOAD=default

    G1 Z2.0 F3000 ; Move Z Axis up little to prevent scratching of Heat Bed
    M106 S255 ; Part fan max strength
    G1 X10 Y20 Z0.3 F5000.0 ; Move to start position
    G1 X10 Y200.0 Z0.3 F1500.0 E15 ; Draw the first line
    G1 X10.4 Y200.0 Z0.3 F5000.0 ; Move to side a little
    G1 X10.4 Y20 Z0.3 F1500.0 E30 ; Draw the second line
    G92 E0 ; Reset Extruder
    G1 Z2.0 F3000 ; Move Z Axis up little to prevent scratching of Heat Bed
    G1 X5 Y20 Z0.3 F5000.0 ; Move over to prevent blob squish


[gcode_macro PRINT_START_NOLINE]
description: starts the print
gcode:

     #Get Bed and Extruder temperature from Slicer GCode
    {% set BED_TEMP = params.BED_TEMP|default(60)|float %}
    {% set EXTRUDER_TEMP = params.EXTRUDER_TEMP|default(200)|float %}
    #Preheat nozzle and bed
    M104 S{EXTRUDER_TEMP} T0                        
    M140 S{BED_TEMP} 

    M106 S255 #part fan full speed
    G92 E0 ; Reset Extruder
    G28 X 
    G28 Y 

    M190 S{BED_TEMP} 
    G28 Z                              
    M109 S{EXTRUDER_TEMP} T0


    #Heat nozzle and bed
   
  BED_MESH_CALIBRATE
  BED_MESH_PROFILE LOAD=default

    G1 Z2.0 F3000 ; Move Z Axis up little to prevent scratching of Heat Bed
    M106 S255 ; Part fan max strength
    G1 X10 Y20 Z0.3 F5000.0 ; Move to start position
    G92 E0 ; Reset Extruder
    G1 Z2.0 F3000 ; Move Z Axis up little to prevent scratching of Heat Bed
    G1 X5 Y20 Z0.3 F5000.0 ; Move over to prevent blob squish

[gcode_macro END_PRINT]
description: finishes the print
gcode:
     G91 ;Relative positioning
    G1 E-2 F2700 ;Retract a bit
    G1 E-2 Z0.2 F2400 ;Retract and raise Z
    G1 X5 Y5 F3000 ;Wipe out
    G1 Z10 ;Raise Z more
    G90 ;Absolute positioning

    M106 S0 #part fan off

    G1 X0 Y200 ;Present print
    M106 S0 ;Turn-off fan
    M104 S0 ;Turn-off hotend
    M140 S0 ;Turn-off bed

    M84 X Y E ;Disable all steppers but Z

[gcode_macro PID_EXTRUDER] description: PID Tune for the Extruder gcode:
gcode:
  {% set TARGET_TEMP = params.TARGET_TEMP|default(210)|float %}
  PID_CALIBRATE HEATER=extruder TARGET=210
  TURN_OFF_HEATERS 
  SAVE_CONFIG

[gcode_macro PID_BED] description: PID Tune for the Bed gcode:
gcode:
  {% set TARGET_TEMP = params.TARGET_TEMP|default(60)|float %}
  PID_CALIBRATE HEATER=heater_bed TARGET=60
  TURN_OFF_HEATERS 
  SAVE_CONFIG
  
----------------------------------------------------------------------------

Parts list -

The Printer Itself
--------------------------------
Criality Ender 3 V2 Neo - https://www.amazon.com/Official-Auto-Leveling-Full-Metal-Pre-Installed-220%C3%97220%C3%97250mm/dp/B0B8N64LM2/ref=sr_1_1_sspa?crid=2ZA0YKVU7C7YO&keywords=ender+3+v2+neo&qid=1694613867&sprefix=ender+3+v2+neo%2Caps%2C257&sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1

Criality Sprite Pro Extruder kit - https://www.amazon.com/Official-Creality-Extruder-Printers%EF%BC%8CSupport-Filament/dp/B0B878P36Y/ref=sr_1_2_sspa?crid=2FYBWGV17CTE9&keywords=sprite%2Bextruder%2Bpro&qid=1694613898&sprefix=Sprite%2Caps%2C207&sr=8-2-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&th=1

Hardened steel 0.6 nozzle - https://www.amazon.com/Creality-Printer-Nozzles-High-end-Hardened/dp/B0C99GFPLH/ref=sr_1_1_sspa?crid=DXZJZBHL4FLE&keywords=Hardened%2Bsteel%2B0.6%2Bnozzle&qid=1694613918&sprefix=hardened%2Bsteel%2B0.6%2Bnozzle%2Caps%2C192&sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&th=1

BTT SKR MINI E3V2 - https://www.amazon.com/BIGTREETECH-Upgrade-Control-TMC2209-Creality/dp/B0882QGFZR/ref=sr_1_3?crid=NZDHGFTRPJHK&keywords=btt+skr+mini+e3v2&qid=1694613958&sprefix=btt+skr+mini+e3v2%2Caps%2C194&sr=8-3

KlipperOS - https://www.klipper3d.org/
--------------------------------
Management system
--------------------------------
Raspberry pi 3B+ - https://www.amazon.com/ELEMENT-Element14-Raspberry-Pi-Motherboard/dp/B07P4LSDYV/ref=sr_1_3?crid=2H54ZG26SP5GH&keywords=Raspberry+pi+3B%2B&qid=1694614003&sprefix=raspberry+pi+3b%2B%2Caps%2C219&sr=8-3

Waveshare 4inch HDMI LCD - https://www.amazon.com/HDMI-LCD-IPS-Resolution-Screen/dp/B084L8Z9G9/ref=sr_1_2?crid=1UDAGA5A0OUXV&keywords=Waveshare+4inch+HDMI+LCD&qid=1694614025&sprefix=waveshare+4inch+hdmi+lcd%2Caps%2C191&sr=8-2

Logitech c170 Webcam (Today i recommand some higher FPS cameras like the C920) - https://www.amazon.com/Logitech-960-000880-C170-Webcam/dp/B009E47ZDI/ref=sr_1_3?crid=KRI7PLJLLIIY&keywords=Logitech+c170+Webcam&qid=1694614047&sprefix=logitech+c170+webcam%2Caps%2C192&sr=8-3

Random Thick 40x40x20 24V fan (Recommand some 12V noctua fan with some buck converters)

Fluidd WebUI

Telegram bot

Buck converters - https://he.aliexpress.com/item/1005005065706512.html?spm=a2g0o.productlist.main.23.21da1b6fWVhKGe&algo_pvid=a6fc8b6d-8f8e-4803-a43c-8ef6578ddb0b&algo_exp_id=a6fc8b6d-8f8e-4803-a43c-8ef6578ddb0b-11&pdp_npi=4%40dis%21ILS%2119.80%2113.68%21%21%215.08%21%21%4021038eda16946142037341242e14b5%2112000031503038743%21sea%21IL%212976067160%21S&curPageLogUid=mCJpcuzRjUsg
--------------------------------

