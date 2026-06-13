# Apografi — Warehouse Inventory Android App

A native Android app for warehouse stock-taking. Staff scan product barcodes to record counts, the app reconciles the physical count against the expected stock, and a correction flow resolves the differences — all in real time, across multiple people counting at once.

> Built with Kotlin, Firebase Realtime Database, and ML Kit. Stock data is loaded into Firebase from the company's Excel exports via a separate Python script.

**Status:** In production — used to run full stock-takes of large warehouses, with no data issues.

---

## Demo

<!-- Σύρε εδώ το βίντεο (το ανεβάζει το GitHub και παίζει inline) -->


## What it does

- **Barcode scanning** to count items quickly, one after another
- **Live count tracking** per product, location and bin, with full measurement history (who, when, how much)
- **Difference detection** — compares the physical count against the expected stock and lists only the products that don't match
- **Correction flow** — open a mismatched product, adjust, and mark it as checked
- **Multi-user, real-time** — several people can count the same warehouse at once; the app shows who is currently checking each item
- **Role-based views** — admins see expected vs. counted; regular users see only what they need
- **In-app updates** — checks a version file and installs new builds over the air

---

## Tech stack

- **Language:** Kotlin (native Android)
- **Backend:** Firebase Realtime Database + Firebase Auth (Google Sign-In)
- **Scanning:** Google ML Kit (barcode) on top of CameraX
- **Data import:** Python script that normalises the company's Excel stock export into the Firebase schema

---

## Engineering highlights

These are the parts I'm most proud of — where the real problem-solving was:

**1. Scanner targeting that matches what you see.**
The camera preview uses `FILL_CENTER`, so the image is cropped to fill the screen. To make "only the barcode inside the cyan frame is accepted" actually true, the on-screen frame is mapped back into image coordinates accounting for that crop — and for the camera's 90°/270° rotation, where width and height swap. Without this, the accepted zone silently drifts away from the visible frame.

**2. A lightweight, derived difference index.**
Instead of reading the entire product tree every time the differences screen opens (expensive on Realtime Database), an admin builds a small `/diff_index` once. It stores only the products that have a mismatch, with the totals already computed. The screen then reads this tiny index live. The index is **derived data** — if it's ever wrong, it's rebuilt from the source of truth, and it never writes back to the real stock.

**3. Concurrency for multiple counters.**
When someone opens a product to check it, a soft lock marks it as "being checked by X". If their app closes or loses connection, an `onDisconnect` handler resets it to "open" so nothing stays stuck. Stock totals are updated with Realtime Database **transactions**, so two people counting the same bin can't overwrite each other.

**4. Server-enforced access control.**
Users sign in once with Google. Write access isn't trusted to the client — Firebase rules deny everything by default and only allow reads/writes from accounts on a whitelist (`/users`) in the database. That whitelist can't be edited from the app (it's managed out-of-band), so a user can't add themselves or escalate, and each user can only read their own record.

---

*Built by Ioannis Kermizidis to solve a real warehouse Cycle-Count Problem.*
