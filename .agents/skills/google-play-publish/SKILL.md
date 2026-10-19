---
name: google-play-publish
description: >-
  Build, sign, inspect, and publish Android App Bundles (.aab) and store listings
  to Google Play Console via the Google Play Developer API (androidpublisher v3).
  Use whenever the user asks to check Play Store tracks, list bundles, update store
  listings, or build and publish an Android release without using the browser UI.
---

# Google Play Publishing Skill (`androidpublisher` v3)

This skill enables token-efficient, direct API operations against Google Play Console (`io.williamlagos.pomodorino` or any Android package) without launching a browser session.

## 1. Credential Verification (Safe Protocol)

Before running any Google Play API command, verify that `GOOGLE_PLAY_SERVICE_ACCOUNT_JSON` is configured in `~/.env` **without** printing its contents:

```bash
grep -sq "^GOOGLE_PLAY_SERVICE_ACCOUNT_JSON=" ~/.env
```

If this check fails (non-zero exit code), ask the user to link a Service Account JSON key using:

```bash
printf "Enter path to Service Account JSON (typing hidden): " && read -s val && echo && echo "GOOGLE_PLAY_SERVICE_ACCOUNT_JSON=$val" >> "/Users/william/.env" && echo "Saved."
```

### One-Time Service Account Setup (If the user does not have one yet)
1. In Google Cloud Console (e.g., project `device-streaming-e385b6a3`), enable the **Google Play Android Developer API** (`androidpublisher.googleapis.com`).
2. Create a Service Account and download a JSON key file (e.g., `~/.config/gcloud/play-publisher.json`).
3. In **Google Play Console → Users and permissions → Invite new users**, invite the Service Account email and grant **Release to testing tracks** and **Release to production** permissions.

---

## 2. Building & Signing the Release `.aab`

The Android project in `android/Pomodorino` is configured to read `KEYSTORE_PASSWORD` and `KEY_PASSWORD` automatically from `~/.env` and sign with `/Users/william/Developer/PlayStore.jks`.

To build a signed release bundle:
```bash
cd /Users/william/Developer/pomodorino/android/Pomodorino && \
JAVA_HOME=$(/usr/libexec/java_home -v 21) ./gradlew bundleRelease && \
cp app/build/outputs/bundle/release/app-release.aab app/release/app-release.aab
```

---

## 3. Publishing & Inspecting via MCP Tools or CLI

You can either call the **`google-play-console` MCP tools** directly:
- `play_list_tracks(package_name="io.williamlagos.pomodorino")`
- `play_list_bundles(package_name="io.williamlagos.pomodorino")`
- `play_get_listing(package_name="io.williamlagos.pomodorino", language="en-US")`
- `play_update_listing(title, short_description, full_description, package_name, language)`
- `play_upload_bundle(aab_path, track="alpha", release_notes="...", release_name="...")`

Or run the CLI directly via `uv`:
```bash
# List tracks & active releases
/opt/homebrew/bin/uv run ~/.gemini/config/mcp_servers/google_play_mcp.py tracks

# List uploaded bundles
/opt/homebrew/bin/uv run ~/.gemini/config/mcp_servers/google_play_mcp.py bundles

# Upload and roll out a signed .aab to Closed Testing - Alpha
/opt/homebrew/bin/uv run ~/.gemini/config/mcp_servers/google_play_mcp.py upload \
  --aab /Users/william/Developer/pomodorino/android/Pomodorino/app/release/app-release.aab \
  --track alpha \
  --name "102 (1.0.1)" \
  --notes "Updated target Android SDK to API level 36 (Android 16)."
```
