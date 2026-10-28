## Windows environment

- 2026-10-21: telemetry blocked machine-wide. Office: connected experiences off (privacy = 2), diagnostic data off (sendtelemetry = 3, DisableTelemetry = 1), customer data and feedback off, first-run movie and opt-in suppressed, set at both HKLM and HKCU.
- Windows: AllowTelemetry = 0, WER disabled, CEIP policy 0, DiagTrack and dmwappushservice disabled, and five telemetry scheduled tasks disabled (Compatibility Appraiser Exp, CEIP Consolidator, UsbCeip, SustainabilityTelemetry, Office Performance Monitor).
- Edge: AllowTelemetry, DiagnosticData and Edge3PSerpTelemetryEnabled = 0, BackgroundModeEnabled = 0. An earlier debloat pass had already set SmartScreenEnabled = 0 and a wide feature-off set; that was pre-existing and left untouched.
- Rollback and the pre-change registry exports live in `C:\Users\BenKaesy\Documents\telemetry-rollback-2026-10-21\` (run `rollback.ps1` elevated).
- Windows Terminal in a Tabby-like theme: Tabby dark and light schemes, auto-switching with the OS theme, Cascadia Code, ligatures off, opaque, no acrylic.
- PowerShell prompt: lean powerline, light and dark aware, with git state and an ownership bar.
- US keyboard layout.

