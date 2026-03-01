# Hardware Design

## Sensor Selection

Traditional Geiger-Müller tube designs require:

- High-voltage modules (300–500V)
- Calibration routines
- Drift compensation
- Mechanical protection

The GDK101 rev:A was selected because:

- Solid-state design
- Factory calibration
- Direct µSv/h output
- No high-voltage requirements
- Lower mechanical sensitivity

## Voltage Considerations

The ESP32 operates at 3.3V logic level.  
The GDK101 requires 5V I²C signaling.

A bidirectional level shifter was implemented to safely translate I²C signals between the devices.

## Prototype Strategy

The first hardware revision was built on a perfboard carrier board:

- Rapid iteration
- Low manufacturing overhead
- Suitable for validation phase

A professional PCB layout may follow in a future hardware revision.