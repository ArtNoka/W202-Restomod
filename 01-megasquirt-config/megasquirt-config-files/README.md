# Megasquirt Configuration Files

This directory contains Megasquirt ECU configuration files and firmware settings.

## Files Included

- **MS3-Base-Config.txt** - Base Megasquirt 3 configuration
- **CAN-Output-Setup.txt** - CAN bus output configuration
- **Sensor-Calibration.txt** - Sensor input calibration values
- **Safety-Limits.txt** - Hard safety limit configuration
- **Tuning-Maps.txt** - Base fuel and ignition maps

## Configuration Steps

1. **Start with MS3-Base-Config.txt**
2. **Configure CAN-Output-Setup.txt** for message output
3. **Calibrate sensors** in Sensor-Calibration.txt
4. **Set safety limits** in Safety-Limits.txt
5. **Load tuning maps** in Tuning-Maps.txt

## Export/Import

All configurations can be exported from Megasquirt tuning software (TunerStudio, MegaLogViewer).

## Testing

After configuration:
1. Connect CAN analyzer
2. Verify message output with candump
3. Validate sensor readings
4. Test tuning command reception

## Next Steps

Move to the parent directory for sensor mappings and CAN protocol details.
