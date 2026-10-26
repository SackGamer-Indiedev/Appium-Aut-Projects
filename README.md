# Mobile Game UI Automation Framework (Appium + Pytest + OpenCV)

End-to-end UI automation for a **Unity-based Android puzzle game**, running on real devices
and Android emulators, with logcat crash/ANR detection and standard test reports
(JUnit XML, pytest-html, Allure).

Unity draws its whole UI into one surface, so the usual Appium element locators only exist
for the native layers (system dialogs, consent, ads). The framework combines both:

| Layer | Technique |
|---|---|
| Native Android dialogs, ad dismiss controls | Appium UiAutomator2 locators (resource-id) |
| Unity screens (home, settings, level select…) | OpenCV template matching on language-independent icons |
| Game board & pieces | Computer-vision board model: grid detected at runtime, cells and tray pieces read from pixels |
| Gameplay | Greedy placement strategy that plays levels, verifying every move against the predicted board |
| Settings toggles | Colour-based toggle detection (works in every language); vibration verified via Android `vibrator_manager` |
| Localization | In-game language picker, visual checks for untranslated text, label wrapping and stale strings; golden screenshots |
| Stability | Per-test logcat capture; tests fail on `FATAL EXCEPTION` / ANR; device pre-flight health check |

## Layout

```
config/config.yaml        device, timeouts, thresholds, logcat patterns
framework/                adb wrapper, Appium driver, logcat monitor, vision, board model, flows, health
pages/                    page objects: system dialogs, ads, screens, settings, menus, game
tests/                    smoke · onboarding · settings · localization · gameplay · navigation · lifecycle · performance
assets/templates/         image templates (icons, buttons, language labels)
assets/golden/            per-language golden screenshots (created on first run)
scripts/run_tests.sh      starts Appium, runs a suite, builds reports
.github/workflows/        CI: same suite on an Android emulator in GitHub Actions
```

## Running

```bash
python3 -m venv .venv && .venv/bin/pip install -r requirements.txt
npm i -g appium && appium driver install uiautomator2
scripts/run_tests.sh smoke        # ~2 min build gate
scripts/run_tests.sh full         # full regression
scripts/run_tests.sh localization # any pytest marker
```

`device.udid: auto` picks the single connected device; set `DEVICE_UDID` to choose one.
Tests that change device-wide settings (Wi-Fi, rotation lock) run on emulators only, or on a
real phone with `ALLOW_DEVICE_SETTINGS=1`.

## Reports

- `reports/junit.xml`: JUnit XML for CI dashboards
- `reports/report.html`: self-contained pytest-html report
- `reports/allure-results/`: Allure results with step screenshots, per-test logcat and failure page source (`allure serve reports/allure-results`)
- `reports/artifacts/<test>/`: screenshots + `logcat.txt` for every test

## Safety rules built in

- Live ads are only ever dismissed with their own close / "Continue to app" control; ad creatives are never tapped.
- No purchase, restore-purchase or account flows are exercised.
