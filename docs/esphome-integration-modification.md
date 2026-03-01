# ESPHome Integration Modification (GDK101 rev:A)

## Summary
The GDK101 rev:A gamma sensor requires a significantly longer startup time before I²C communication becomes stable. Default ESPHome component initialization accessed the sensor too early, resulting in timeouts and repeated initialization failures after reboot.

To solve this, the ESPHome integration was implemented as an `external_components` module with an explicit initialization state machine and retry scheduler.

---

## Observed Problem

### Symptoms
After ESP32 reboot:

- early I²C reads frequently fail
- the component reports initialization errors
- sensor entities may be unavailable or remain in an error state

### Root Cause
The ESP32 and ESPHome stack become operational quickly after boot, while **GDK101 rev:A** needs additional time before responding reliably on the I²C bus. This leads to a timing mismatch during boot.

---

## Solution Approach

### Design Goals
- Do not block the main loop with long startup delays
- Automatically recover after reboot without manual intervention
- Avoid permanently failing the component (`mark_failed()`) due to slow sensor startup
- Provide predictable upper-bound for retries

### Implementation Overview
The component tracks initialization state and retries initialization in the background:

- `setup()` attempts initialization once
- on failure, a retry is scheduled every 2 seconds
- maximum of 45 attempts (~90 seconds)
- once successful, normal polling begins

---

## Key Implementation Details

### Initialization Flow
- `setup()`: resets internal state and tries initialization once
- `try_initialize_()`:
  - triggers sensor reset
  - checks reset acknowledgement (`data[0] == 1`)
  - waits briefly (`delay(10)`)
  - reads firmware register as readiness confirmation
- `schedule_init_retry_()`:
  - schedules a `set_timeout()` callback after 2000 ms
  - retries until success or until maximum attempts is reached

### Runtime Behavior
- `update()` exits early while not initialized
- once initialized, the component polls:
  - 1-minute average dose (µSv/h)
  - 10-minute average dose (µSv/h)
  - status register + vibration flag
  - measurement duration

---

## Result
With the retry-based initialization:

- the sensor reliably becomes available after reboot
- no manual restart cycles are required
- the component remains stable even with slow startup behavior of rev:A hardware

---

## Why external_components?
Using `external_components` keeps the fix:

- version-controlled together with the project
- independent from upstream ESPHome release cycles
- easy to deploy in a containerized ESPHome environment
