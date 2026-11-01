# MediStock — Kotlin Android medical shop app

A college project for a medical shop owner to track stock, take customer orders, and hand them over later. Built with **native Kotlin + Jetpack Compose**. Android only; no website or server required.

## Open and run

1. Open this folder in Android Studio.
2. Use **JDK 17** as the Gradle JDK. Install **Android SDK 36** if prompted.
3. Let Gradle sync, then select an Android phone/emulator running **Android 8.0 (API 26) or newer**.
4. Run the `app` configuration.

From PowerShell in this folder:

```powershell
.\gradlew.bat assembleDebug
.\gradlew.bat testDebugUnitTest lintDebug
```

The installable debug APK is generated at `app/build/outputs/apk/debug/app-debug.apk`.

The SDK path in `local.properties` is machine-specific and ignored by Git. Android Studio creates this file automatically on another machine.

## Try the app

On the first launch, choose **Create your shop** for your own account, or **Explore demo shop** for a ready-made account:

| Demo login | Value |
| --- | --- |
| Email | `demo@medistock.app` |
| Password | `Medi@12345` |

The app supports **one owner/shop on one device**. The demo button creates that device's owner account; it is only offered before account setup. After logout, log in with the same credentials. Login persists across app restarts until logout.

## Features

- **Login and logout:** local owner account, email validation, salted PBKDF2 password hashing, password visibility toggle.
- **Black + cyan theme:** dark backgrounds and cards, cyan buttons and accents, readable light text, and matching system bars and launcher icon across the app.
- **Home:** available stock, pending orders, delivered order value, stock alerts, recent orders.
- **Inventory:** 15 sample products (10 medicines, 5 injections), prices in rupees, search, category/stock filters, available/low/out-of-stock labels, manual stock updates.
- **Orders:** select products, adjust quantities, save customer name, optional phone and pickup note, inspect order details, search and filter by status.
- **Handover:** pending orders can be marked delivered or cancelled with confirmation.
- **Profile:** edit owner name, store name, and phone; view email; log out.
- **Offline persistence:** account, session, profile, inventory, and saved orders remain after closing the app.

## Example demo flow

1. Tap **Explore demo shop**.
2. Open **Inventory**, add two Paracetamol strips, and tap the green review bar.
3. Enter a customer name and save. The new pending order appears under **Orders**; Paracetamol stock changes from **42 to 40**.
4. Open that order and choose **Mark as delivered** when handing it over. Stock stays **40** because those units were reserved when the order was created.
5. Alternatively, cancel a pending order: its reserved units return to available inventory.
6. Edit your profile, log out, and log in again to see the saved records.

An unsaved cart survives switching tabs and screen rotation, but is cleared by logout or process death. Customer form state survives screen rotation. Saved orders always persist.

## Stock rules

- Each catalog quantity counts **selling units**: strips, vials, bottles, sachets, or ampoules. A strip is one unit, not one tablet.
- Stock is reserved when an order is saved. The entire order is validated before saving; an unavailable item rejects the order without partially deducting other items.
- Delivering an order does not deduct stock a second time.
- Cancelling a pending order restores its quantities exactly once. Delivered/cancelled orders are final.
- Manual stock edits set the quantity **available for new orders**, excluding units already reserved for pending orders.
- Prices are stored as integer paise and copied into each saved order.
- Products with **5 or fewer available units** show low stock. Zero is out of stock.
- Rx products require the owner to acknowledge checking a prescription before saving the order. No dosage or treatment advice is provided.

## Code structure

```text
app/src/main/java/com/medistock/app/
  MainActivity.kt            Android entry point
  ShopViewModel.kt           UI state, background saves, session and cart
  data/
    Models.kt                Products, owner, orders, sample catalog
    InventoryEngine.kt       Pure stock and order transactions
    ShopRepository.kt        Private JSON preferences and password hashing
  ui/
    Theme.kt                 Colors and typography
    Components.kt            Shared widgets and formatting
    MediStockApp.kt          App shell and bottom navigation
    AuthScreen.kt            Login and shop creation
    HomeScreen.kt            Dashboard
    InventoryScreen.kt       Catalog, search and stock editing
    CheckoutScreen.kt        Customer details and order creation
    OrdersScreen.kt          Order list, details and handover
    ProfileScreen.kt         Owner profile and logout
```

The architecture uses a ViewModel, repository, immutable data models, and a pure transaction engine. One private SharedPreferences file stores the shop as JSON; inventory and orders are committed together on an IO dispatcher before the displayed state changes. Credentials use a random 16-byte salt and PBKDF2-HMAC-SHA256 with 120,000 iterations. Passwords are not stored in plaintext. App backup is disabled.

## Project scope

This is an offline college demonstration. Accounts are local, and there is no cloud sync, remote login, password recovery, payment processing, GST invoice, batch/expiry tracking, or production pharmacy backend. Prices, product details, prescription flags, and initial quantities are illustrative catalog data. Clearing app data or uninstalling removes the account and records. Production deployment would need authenticated server storage, backups, and appropriate pharmacy workflows.

## Verification

`InventoryEngineTest` covers 12 transaction cases: catalog integrity, reservation and totals, atomic rejection of over-ordering, unavailable products, invalid carts, cancellation restoration, delivery without double deduction, order numbering, customer validation, stock editing, restocking with reservations, and invalid status transitions.

Verified on 28 October 2026: `assembleDebug`, `testDebugUnitTest` (12 passed, zero failures), and `lintDebug` completed successfully. Lint has no errors; remaining warnings concern pinned dependency/target versions and optional KTX syntax. Device UI testing has not been performed.

The `dist/` folder contains a copy of the debug APK and a source ZIP for sharing/submission. The source ZIP excludes build caches, local SDK paths, and generated APKs.

Build tool versions are pinned: AGP 8.12.0, Gradle 8.13, Kotlin 2.2.21, JDK 17, compile/target SDK 36. The Android team's [AGP 8.12 compatibility table](https://developer.android.com/build/releases/agp-8-12-0-release-notes) documents its Gradle, JDK, and SDK requirements.
