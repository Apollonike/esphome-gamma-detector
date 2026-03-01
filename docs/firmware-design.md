# Firmware Design

The firmware is based on ESPHome with a custom external component for the GDK101.

## Key Design Decisions

- Polling-based I²C communication
- Explicit initialization state tracking
- Retry-based startup handling
- Separation of sensor logic and HA integration

## Data Handling

The sensor provides:

- 1-minute average radiation dose (µSv/h)
- 10-minute average radiation dose (µSv/h)
- Measurement duration
- Status flags
- Firmware version

Values are published as Home Assistant sensor entities.