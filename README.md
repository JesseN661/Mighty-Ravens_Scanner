# Mighty Ravens Scanner

A mobile app for **Ravens Nexus Energy** (a subsidiary of **Mighty Engineering Co**) that lets field and office staff register the identifying codes of solar kits quickly and accurately.

## About

When Ravens Nexus Energy sells or hires out solar kits to clients under a signed sale/hire agreement, every agreement/contract must carry three identifying codes:

- IMEI / device ID number
- Serial number of the device
- SIM card number of the kit (client)

Today these codes are written down by hand. That process is slow, error-prone, and makes it hard for staff to look up or search this information later (for example, when there is an issue with a kit).

## Proposed Solution

A simple phone app that lets staff scan the barcode/QR code on each kit box to capture the three identifying codes without typing them manually, and store every kit against the correct client record so the data can be found instantly later.

## Functional Requirements

1. **Barcode / QR scanning** — the phone camera scans the barcode/QR on each solar kit box to capture the three identifying codes.
2. **Manual entry fallback** — every code can also be typed in by hand whenever a barcode is damaged, missing, or unreadable.
3. **Import / export** — documents (client lists / contract records) can be imported or exported, and the contents of an import can be pasted directly into a dedicated section of the app for review before saving.
4. **Client matching** — the three codes are attached to the correct client/customer record, and can be matched to the person by their phone number or ID number.
5. **Searchable records** — stored records can be searched and retrieved by client name, phone number, or by any of the codes.
6. **Duplicate alerts** — staff are warned about duplicate data, and the app blocks a code that has already been scanned/allocated to another client, preventing double-allocation of a kit.
7. **Offline-first** — the app works fully offline when there is no internet connection, records remain stored on the device, and a connection is restored it syncs back automatically.
8. **Simple login + ease of use** — a basic login for field/office staff only, and an interface simple enough that no technical training is required.

## Key Decisions Made

- **Storage:** Local-only for now. All data lives in an on-device SQLite database. The architecture (UUID ids, update timestamps, and a change-log "outbox" table) is designed so a cloud/back-end sync engine can be plugged in later.
- **Barcode contents:** One code per barcode. Staff scan the device barcode, the serial barcode, and the SIM barcode separately for each kit.
- **Platforms:** Android and iOS (Flutter).
- **Import/export formats:** CSV and Excel (.xlsx). Data can be imported from a file or by pasting text into the app; exported files can be shared or saved.
- **Login:** Local username/password accounts inside the app; an admin account is created on first run and can add staff accounts.

## App Structure

```
lib/
  main.dart                     App entry point (blank for now)
  models/
    user_model.dart             Staff / admin user
    contract_model.dart         Sale-hire agreement with the 3 codes
    client_model.dart           Client/customer record
  services/
    database_service.dart       SQLite schema, CRUD, duplicate checks, seeds
    auth_service.dart           Local login / account management
    import_service.dart         CSV + Excel + pasted-text import with preview
    export_service.dart         CSV + Excel export, share/save files
    sync_service.dart           Stub for future cloud sync + offline indicator
  screens/
    login_screen.dart           Login + first-run admin setup
    dashboard_screen.dart       Home: Scan, Search, Clients, Import/Export, Settings
    scan_screen.dart            Camera scanning (IMEI / Serial / SIM modes)
    new_contract_screen.dart    Manual entry of a kit + attach to client
    contract_list_screen.dart   List of saved agreements/kits
    contract_detail_screen.dart Single agreement with its 3 codes
    client_list_screen.dart
    client_form_screen.dart
    search_screen.dart          Search by name, phone, ID, or any code
    import_screen.dart
    export_screen.dart
    settings_screen.dart        Manage staff accounts, app info
  widgets/
    code_tile.dart              Displays one identifying code
    sync_indicator.dart         Offline/sync status chip
    duplicate_warning_dialog.dart  Blocks/warns on duplicate codes
    empty_state.dart
  utils/
    validators.dart             IMEI / phone / ID checks
    formatters.dart
test/                           Widget + import parser + duplicate-detection tests
```

## Data Model (summary)

- **Users** — username, password hash, role (`admin`/`staff`).
- **Clients** — name, phone number, national ID number.
- **Contracts** — client reference, agreement number, IMEI code, serial code, SIM code, notes, scanned by, status. The three code columns are unique to prevent double-allocation.
- **Outbox / change log** — records every change so a future sync engine can push/pull to a central server.

## How to Run

```bash
flutter pub get
flutter run
```

For a release build on a physical phone:

```bash
flutter build apk       # Android
flutter build ios       # iOS (requires macOS + Apple Developer account)
```

## Future Work

- Cloud/back-end integration so records are automatically synced across all devices when connected (the sync service layer is already reserved for this).
- Web/PWA version of the app.
- PDF report generation per client/contract.