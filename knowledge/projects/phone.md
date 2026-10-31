# phone — Samsung Galaxy S21+ 5G

- Device: SM-G996U (`t2q`), Android 15, XAA unlocked, 7.5 GB RAM, 105 GB storage.

- 2026-10-31: **debloat pass 1.** Read-only audit found 51 GB free, 7.24/7.5 GB RAM used, 288 process records, 10.1 GB in disabled apps, 5.8 GB cache. Removed the Secure Folder profile (`pm remove-user 150`) and disabled `com.samsung.knox.securefolder`; cleared data and disabled `com.microsoft.appmanager`. Free space went 51 → 54 GB. Rollback: `pm enable com.microsoft.appmanager` (data is gone); `pm enable com.samsung.knox.securefolder` (profile and data cannot be restored). A reboot clears the last stale user-150 process record.
- Pending candidates: cache (Chrome 1.68 GB, Play Store 0.98 GB, YouTube 0.71 GB, Reddit 0.60 GB); disabled apps (MovieBox 2.46 GB, Snapchat 1.29 GB, Facebook Lite 0.65 GB, Morphe Photos 0.84 GB, etc.); CapCut (1.09 GB, 3 processes); Z Route (2.59 GB).
