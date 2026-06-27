# W202 Restomod - Complete IVI & Instrument Cluster System

A comprehensive dual-screen infotainment and instrument cluster system for a 1996 Mercedes-Benz C-Class (W202) restomod with standalone Megasquirt ECU integration.

## Project Overview

This project integrates modern technology into a classic W202 with:
- **Instrument Cluster Screen** (12.3" display): Real-time gauges, boost, temps, diagnostics
- **IVI Screen** (12.3" display): Audio, Navigation, Bluetooth, Apple CarPlay, Android Auto, tuning hub
- **Physical Controls**: Climate control switches with integrated LCD displays
- **Standalone ECU**: Megasquirt with full CAN bus communication
- **Sensors**: Boost, Oil Temp, Lambda/O2, Air Intake Temp, MAF + existing OEM sensors

## Development Phases

- [ ] Phase 1: Megasquirt Configuration & CAN Setup
- [ ] Phase 2: Raspberry Pi & Hardware Integration
- [ ] Phase 3: Backend Development
- [ ] Phase 4: IC App Development
- [ ] Phase 5: IVI App Development
- [ ] Phase 6: Climate Control System
- [ ] Phase 7: Testing & Vehicle Installation

## Repository Structure

```
W202-Restomod/
├── 01-megasquirt-config/      # ECU configuration & CAN protocol
├── 02-hardware/               # Hardware setup & wiring
├── 03-backend/                # Python backend & API
├── 04-ic-app/                 # Instrument cluster display
├── 05-ivi-app/                # Infotainment system
├── 06-climate-control/        # Climate control modules
├── 07-testing/                # Testing & validation
├── 08-documentation/          # Guides & references
├── 09-resources/              # Datasheets & docs
└── README.md
```

## Quick Start

1. Navigate to **01-megasquirt-config/** to begin
2. Follow each directory's README in order
3. Refer to **08-documentation/** for detailed guides

## Status

🚧 **Phase 1: Foundation** - Project initialized

---

**Last Updated**: 2026-06-27