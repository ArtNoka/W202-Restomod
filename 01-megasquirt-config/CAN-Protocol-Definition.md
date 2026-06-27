# Megasquirt CAN Protocol - W202 Restomod

## Overview

This document defines the complete CAN message protocol for communicating between the Megasquirt ECU and the Raspberry Pi central hub.

## CAN Bus Configuration

### Physical Layer
- **Speed**: 500 kbps (standard automotive)
- **Protocol**: ISO 11898-1 (Standard 11-bit CAN)
- **Message Type**: Standard frames (not extended)
- **Termination**: 120Ω resistors at both ends of CAN bus

### Electrical Specifications
- **Voltage**: 12V automotive
- **CAN_H**: Nominal 2.5V (idle) to 3.5V (dominant)
- **CAN_L**: Nominal 2.5V (idle) to 1.5V (dominant)
- **Max Devices**: Up to 32 nodes

---

## Message Strategy

### Data Organization

We'll use **separate CAN messages** for different data groups to:
- Reduce latency on critical data (RPM, boost)
- Allow scalable growth
- Enable selective filtering
- Improve real-time performance

### Update Frequencies

| Message Type | Frequency | Priority | Purpose |
|--------------|-----------|----------|---------|
| **Engine Core** (RPM, Fuel) | 100 Hz (10ms) | HIGH | Critical for gauges |
| **Sensor Data** (Boost, Temps) | 50 Hz (20ms) | HIGH | Real-time monitoring |
| **Status/Warnings** | 10 Hz (100ms) | MEDIUM | Alerts & diagnostics |
| **Extended Data** | 5 Hz (200ms) | LOW | Non-critical info |

---

## CAN Message Definitions

### Message 1: Engine Core Data

**CAN ID**: `0x100` (256 decimal)
**DLC**: 8 bytes
**Frequency**: 100 Hz (10ms)
**Priority**: Highest

| Byte | Data | Range | Resolution | Unit |
|------|------|-------|------------|------|
| 0-1 | RPM | 0-8000 | 1 | rpm |
| 2 | Fuel Pulse Width | 0-100 | 0.5 | % |
| 3 | Engine Load | 0-100 | 0.5 | % |
| 4 | Throttle Position | 0-100 | 0.5 | % |
| 5 | Ignition Advance | -30 to +60 | 0.5 | degrees |
| 6 | Air-Fuel Ratio (AFR) | 10-20 | 0.1 | ratio |
| 7 | Status Flags | 0-255 | 1 | flags |

**Example CAN Frame**:
```
ID: 0x100
Data: 05 DC 2A 45 52 1E 0F 0E
      ↓    ↓    ↓   ↓   ↓   ↓   ↓   ↓
      RPM: 1500 FuelPW: 42% Load: 69% TPS: 82% IGN: 30° AFR: 14.7 Status: OK
```

---

### Message 2: Sensor Data - Primary

**CAN ID**: `0x101` (257 decimal)
**DLC**: 8 bytes
**Frequency**: 50 Hz (20ms)
**Priority**: High

| Byte | Data | Range | Resolution | Unit |
|------|------|-------|------------|------|
| 0-1 | Boost Pressure | 0-50 psi | 0.1 | psi |
| 2-3 | Oil Temperature | -40 to +150 | 0.5 | °C |
| 4-5 | Coolant Temperature | -40 to +150 | 0.5 | °C |
| 6 | Lambda (O2) Sensor | 0.7-1.3 | 0.01 | lambda |
| 7 | Reserved | - | - | - |

---

### Message 3: Sensor Data - Secondary

**CAN ID**: `0x102` (258 decimal)
**DLC**: 8 bytes
**Frequency**: 50 Hz (20ms)
**Priority**: High

| Byte | Data | Range | Resolution | Unit |
|------|------|-------|------------|------|
| 0-1 | MAF Sensor | 0-300 | 0.1 | g/s |
| 2-3 | Air Intake Temp (IAT) | -40 to +120 | 0.5 | °C |
| 4-5 | Vehicle Speed | 0-200 | 0.5 | km/h |
| 6 | Fuel Level | 0-100 | 1 | % |
| 7 | Reserved | - | - | - |

---

### Message 4: Status & Warnings

**CAN ID**: `0x103` (259 decimal)
**DLC**: 8 bytes
**Frequency**: 10 Hz (100ms)
**Priority**: Medium

| Byte | Data | Range | Values | Description |
|------|------|-------|--------|-------------|
| 0 | ECU Status | 0-255 | Flags | Engine status |
| 1 | Error Codes | 0-255 | Bitmask | Fault flags |
| 2 | Warnings | 0-255 | Bitmask | Alert flags |
| 3-4 | Fuel Pressure | 0-100 | 0.1 psi | Fuel rail pressure |
| 5 | Battery Voltage | 9-15 | 0.1 | Volts |
| 6 | Knock Count | 0-255 | 1 | - |
| 7 | Runtime | varies | 1 | Engine runtime (min) |

**ECU Status Flags (Byte 0)**:
```
Bit 0: Engine Running (1=yes, 0=no)
Bit 1: Fuel Injectors Enabled
Bit 2: Ignition Enabled
Bit 3: Boost Solenoid Active
Bit 4: VVT Active (if equipped)
Bit 5: Auto-tune Active
Bit 6: CAN Communication OK
Bit 7: Reserved
```

---

### Message 5: Extended Data

**CAN ID**: `0x104` (260 decimal)
**DLC**: 8 bytes
**Frequency**: 5 Hz (200ms)
**Priority**: Low

| Byte | Data | Range | Resolution | Unit |
|------|------|-------|------------|------|
| 0-1 | Manifold Absolute Pressure (MAP) | 0-300 | 0.1 | kPa |
| 2-3 | Boost Target | 0-50 | 0.1 | psi |
| 4 | Gear Selection | 0-6 | 1 | - |
| 5 | Shift Counter | 0-255 | 1 | - |
| 6-7 | Total Runtime | 0-65535 | 1 | hours |

---

### Message 6: Tuning Command (Raspberry Pi → ECU)

**CAN ID**: `0x300` (768 decimal)
**DLC**: 8 bytes
**Frequency**: On-demand
**Priority**: Medium

**Command Structure**:

| Byte | Data | Description |
|------|------|-------------|
| 0 | Command ID | Type of adjustment |
| 1 | Parameter | Which parameter to adjust |
| 2-3 | Value | New value (16-bit) |
| 4-5 | Checksum | CRC for validation |
| 6-7 | Reserved | Future use |

**Command Types**:
- `0x01` = Boost target adjustment
- `0x02` = Fuel map adjustment
- `0x03` = Ignition timing adjustment
- `0x04` = RPM limit adjustment
- `0x05` = Parameter reset
- `0x06` = Fault code clear

---

## Safety Limits (Hard Limits in ECU)

```
Max Boost:        1.5 bar (22 psi)
Max RPM:          7500
Min Oil Temp:     60°C (warning below)
Max Oil Temp:     120°C (critical)
Min Fuel Pressure: 35 psi
Max Fuel Pressure: 60 psi
Lean Limit:       16.5 AFR
Rich Limit:       12.0 AFR
```

---

## CAN Bus Arbitration

### Priority (Lower ID = Higher Priority)

```
0x100 (Engine Core)        → Priority 1 (Highest)
0x101 (Sensors Primary)    → Priority 2
0x102 (Sensors Secondary)  → Priority 3
0x103 (Status/Warnings)    → Priority 4
0x104 (Extended Data)      → Priority 5
0x300 (Tuning Command)     → Priority 2 (From hub)
```

---

## Implementation Checklist

- [ ] Configure Megasquirt for CAN output
- [ ] Set CAN bitrate to 500 kbps
- [ ] Define message IDs in Megasquirt firmware
- [ ] Configure update frequencies
- [ ] Implement byte-order (little-endian)
- [ ] Set up error handling
- [ ] Test with CAN analyzer
- [ ] Document any deviations

---

## Next Steps

1. Configure these parameters in your Megasquirt ECU
2. Test CAN communication with an analyzer
3. Move to sensor mapping configuration
4. Verify data formats with actual ECU output
