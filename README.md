Google Chrome (macOS) – Intune Custom Configuration Profiles
This repository contains Apple configuration profile (.mobileconfig / XML) files used to manage Google Chrome on macOS via Microsoft Intune, using device‑scoped Custom profiles.
The profiles are intentionally split by concern (UX, Updates, Extensions, Profiles/Sync, Security) to:

Minimise blast radius
Simplify troubleshooting
Align with Open Intune Baseline (OIB)–style layering
Support incremental hardening (v0.x → v1.x)

All policies are enforced via macOS Managed Preferences (MCX) and apply at the device (System) scope.

/
├── ChromeBase.xml        # User Experience (UX baseline)
├── ChromeUpdates.xml     # Chrome update & maintenance behaviour
├── ChromeExtensions.xml  # Extension control (default deny / allowlist)
├── ChromeProfiles.xml    # Profiles, sign-in, sync controls
├── ChromeCIS.xml         # Security controls (CIS L1 aligned)
└── README.md

General Design Principles

Platform: macOS
Deployment method: Microsoft Intune → Configuration Profiles → Custom
Scope: Device (PayloadScope = System)
Policy source in Chrome: Platform
Verification point: chrome://policy

Chrome policies are delivered via:
/Library/Managed Preferences/com.google.Chrome.plist

Chrome does not read directly from Intune — Intune delivers MCX, and Chrome consumes it.

Critical MCX Rule (Read This First)

All Chrome policies MUST be flat under mcx_preference_settings.
Never nest mcx_preference_settings inside another policy key.
Never nest policies inside each other unless explicitly documented (e.g. ExtensionSettings).

Violating this rule silently corrupts the MCX tree, causing:

Later profiles to be ignored
Policies to disappear without errors
Extremely confusing behaviour

This repo reflects the corrected structure after hard‑won debugging.

Profile Details
1. ChromeBase.xml – User Experience
Purpose:
Provide a predictable, low‑noise Chrome UX without security impact.
Examples:

Homepage / startup behaviour
Home button visibility
Suppress promo tabs
Disable telemetry (non‑security)

Notes:

No security enforcement
Safe to deploy broadly
Usually invisible to end users


2. ChromeUpdates.xml – Updates
Purpose:
Ensure Chrome remains updated while avoiding disruptive update prompts.
Controls:

Auto‑update check cadence
Update suppression windows

Notes:

Chrome updates on macOS are handled by Google Keystone
These policies guide behaviour but do not “block” updates


3. ChromeExtensions.xml – Extensions (Default Deny)
Purpose:
Full extension governance using default‑deny with explicit allow / force rules.
Implementation:

Uses ExtensionSettings
Blocks all extensions by default ("*" rule)
Allows or force‑installs approved extensions only

Key points:

ExtensionSettings is the only place extension controls should live
Do not duplicate extension controls in other profiles
force_installed extensions include a Chrome Web Store update URL


4. ChromeProfiles.xml – Profiles, Sign‑In & Sync
Purpose:
Prevent data egress and shadow profiles by controlling identity features.
Controls:

Disable Chrome sign‑in
Disable sync
Prevent adding new browser profiles
Suppress profile picker UX

Important:

This profile must not contain ExtensionSettings
All keys are flat under mcx_preference_settings


5. ChromeCIS.xml – Security (CIS L1 Aligned)
Purpose:
Baseline Chrome hardening aligned to CIS Level 1 intent without breaking UX.
v0.1 controls include:

Safe Browsing enabled (standard level)
Dangerous download restrictions
Disable password manager
Disable autofill (address & credit card)
Block insecure content
Disable network prediction / prefetch

Notes:

Labelled “CIS L1 Aligned”, not “Enforced”
Designed for safe, incremental rollout
Higher‑impact controls belong in later revisions


