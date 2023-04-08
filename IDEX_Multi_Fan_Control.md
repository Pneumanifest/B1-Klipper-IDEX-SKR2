# Copy from this source; https://klipper.discourse.group/t/idex-parts-cooling-fan-turned-off/2160

# This goes in the T0 and T1 macros

```
## add to T0 macro
SET_GCODE_VARIABLE MACRO=FAN_VARIABLE VARIABLE=active_fan VALUE=0
CARRIAGE_PRINT_FAN 

## add to T1 macro
SET_GCODE_VARIABLE MACRO=FAN_VARIABLE VARIABLE=active_fan VALUE=1
CARRIAGE_PRINT_FAN
```

# This is to define the fans

```
# X1 print cooling fan
[fan_generic fan]
pin: PB7
cycle_time: 0.0100
kick_start_time: 1.00

# X2 print cooling fan
[fan_generic fan1]
pin: PB6
cycle_time: 0.0100
kick_start_time: 1.000
```

# Alternative option to use more than 1 fan pin for one controll

```
# X2 print cooling with 2 separate controllable fans
[multi_pin T1_fans]
pins: PB6, PB5

[fan_generic fan1]
pin: multi_pin:T1_fans
# cycle_time: 0.0100
# kick_start_time: 1.000
```

# This is the Macro to use the M106 and M107 input

```
[gcode_macro FAN_VARIABLE]
gcode:

variable_active_fan: 0

#--- Carriages print cooling FANs ---
variable_fan: 'fan'    # carriage X1
variable_fan1: 'fan1'   # carriage X2
#--- Other user define FANs ---
variable_fan2: 'fan2'     
variable_fan3: 'fan3'
variable_fan4: 'fan4'  
variable_fan5: 'fan5'


##########################################################
# PRINTER FANS MANAGEMENT
# Redefine G-code command M106 and M107 
##########################################################

[gcode_macro M106]
description: Override "M106" to allow multiple extruders.
gcode:

    {% set fan_vars = printer["gcode_macro FAN_VARIABLE"] %}
    {% set raw_speed = params.S|default(0)|float %}
    {% set fan_speed = (raw_speed / 255.0)|round(2) %}     
        
    {% set target_fan = params.P|default(fan_vars.active_fan)|int %}
    {% set default_fan = ('fan', 'fan1', 'fan2', 'fan3', 'fan4', 'fan5')[target_fan] %}     
    
    {% if target_fan in [0,1] %}
       ### carriages print cooling FAN   
        CARRIAGE_PRINT_FAN SPEED={fan_speed}
    {% else %}
       ### other user define FAN
        SET_FAN_SPEED FAN={default_fan} SPEED={fan_speed}
    {% endif %}
 

[gcode_macro M107]
description: Override "M107" to allow multiple extruders.
gcode:
    M106 S0
     
 
[gcode_macro CARRIAGE_PRINT_FAN]
description: Set automatically the print fan speed for dual carriages modes 
gcode:
    {% set fan_vars = printer["gcode_macro FAN_VARIABLE"] %}      
    
    {% if params.SPEED is defined %}
        {% set fan_speed = params.SPEED|float %}
    {% else %}
        ### read print fan speed from active carriage/extruder
        {% set fan_speed = printer["fan_generic " + fan_vars.fan].speed|float %}
        {% set fan1_speed = printer["fan_generic " + fan_vars.fan1].speed|float %}
        {% set fan_speed = [fan_speed, fan1_speed]|max %}
    {% endif %}
        
    {% if fan_vars.active_fan == 0 %}
        ### FAN on carriage X1
        SET_FAN_SPEED FAN={fan_vars.fan} SPEED={fan_speed}
        SET_FAN_SPEED FAN={fan_vars.fan1} SPEED=0
    {% elif fan_vars.active_fan == 1 %}
        ### FAN on carriage X2
        SET_FAN_SPEED FAN={fan_vars.fan} SPEED=0
        SET_FAN_SPEED FAN={fan_vars.fan1} SPEED={fan_speed}
    {% endif %}
    ```
