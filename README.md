# RIT / IoT — ESP8266 NodeMCU Android app

A GitHub-ready Capacitor 7 Android project rebuilt from the supplied RIT Control reference files. The UI is refreshed, not a pixel-identical copy. ESP8266 NodeMCU 1.0 (ESP-12E Module) is the only supported microcontroller. All control and telemetry use local Wi-Fi. No board simulation is included in the application.

## Quick start: upload to GitHub and get your APK

**Important: extract the source ZIP first. Uploading the ZIP itself as one repository file does not start a build.**

1. Extract `RIT-IoT-GitHub-Source.zip` on your computer.
2. Create a GitHub repository (for example `rit-iot-esp8266`). Do not add another template app on top of this one.
3. Upload the **contents** of the extracted folder into the repository root. You should see `package.json`, `www`, `firmware`, `android` and `.github` at the top level—not inside an extra nested folder.
4. Make sure `.github/workflows/build-apk.yml` exists in the repository. Some file pickers hide dot-folders; turn on “show hidden files” or use the Git commands below.
5. Commit to `main`. Open the repository's **Actions** tab. If asked, enable workflows.
6. Choose **Build Android APK**. A push to `main` or `master` starts it automatically, or use **Run workflow**. Allow several minutes.
7. Open the successful run and scroll to **Artifacts**. Download **RIT-IoT-ESP8266-debug-APK**.
8. Extract that artifact ZIP. Install `app-debug.apk` on your Android phone. This is an installable debug-signed APK, not a Play Store release.
9. Permit installation from your browser/file manager when Android asks. Only install files from sources you trust. Grant camera/microphone permissions when using those features.

Git alternative (run inside the extracted folder):

```sh
git init
git add .
git commit -m "Add RIT IoT ESP8266 app and APK build"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPOSITORY.git
git push -u origin main
```

You do not need to add secrets, a signing keystore, Android Studio or a paid build service for the debug workflow. Actions availability/usage depends on your account. A stable release signing key and additional release setup are required for publishing and predictable future app upgrades. A pre-existing `com.rit.control` app signed by another key may need to be uninstalled first (which clears its saved settings). The included locally built debug APK and a GitHub-built debug APK can also have different debug keys.

## What is included

- Home top-bar App Information icon and a dedicated beginner guide.
- Separate information icons on every project card, project interface and Sensor Status section.
- Every project guide includes introduction, components, a five-column connection table, logical wiring diagram, complete standalone ESP8266 sketch, copy-code/download buttons, operating instructions and troubleshooting.
- Native UDP discovery on port 4210 (`RIT_DISCOVER`), numeric IP connection on port 80, firmware identity check, live status, profile mismatch prevention and disconnect controls.
- Car drive/light/horn, four relays, positional servo, LCD text, OLED text/drawing, Switi voice control, camera motion-zone control, five sensor interfaces.
- All board communication in Android uses Capacitor's native HTTP layer. `https://localhost` provides the local app origin needed for camera features; the native HTTP layer handles the board's `http://<IP>` communication.
- No external CDN, remote fonts, downloaded AI model or external web asset is required by the app UI.
- GitHub workflows build the APK and independently compile all ESP8266 sketches.

### Firmware selection

Upload **one** sketch at a time. These are separate projects, not an unsafe all-in-one pin assignment.

| App project | Sketch directory | Main pins |
|---|---|---|
| Car Control | `firmware/car/car.ino` | Motor IN1/2 D5/D6; IN3/4 D7/D0; light D1; buzzer D2 |
| Home Relay Control | `firmware/relay/relay.ino` | Relays D5, D6, D7, D0 |
| Servo Control | `firmware/servo/servo.ino` | Signal D5; external 5 V power |
| LCD 16×2 | `firmware/lcd/lcd.ino` | SDA D2, SCL D1, mandatory level shifter |
| OLED 128×64 | `firmware/oled/oled.ino` | SDA D2, SCL D1, 3.3 V module |
| Temperature / humidity | `firmware/dht/dht.ino` | DHT22 data D5 |
| Light sensor | `firmware/ldr/ldr.ino` | A0 on a board with verified 3.3 V-range input divider |
| Motion sensor | `firmware/pir/pir.ino` | HC-SR501 OUT D5 |
| Ultrasonic distance | `firmware/ultrasonic/ultrasonic.ino` | TRIG D5, divided ECHO D6 |
| Parking assistant | `firmware/parking/parking.ino` | TRIG D5, divided ECHO D6, occupied LED D1 |
| Camera Gesture Control | Same `car` sketch and wiring | Phone camera, not an ESP camera |
| Switi Voice Assistant | `relay` OR `car`, matching selected mode | Phone microphone |

**Camera limitation:** this implementation measures image motion in left/right/top zones. It is not AI finger counting or hand-pose recognition. Start camera, keep the phone fixed, then arm; left/right motion turns and top-centre motion drives forward. No motion stops. It auto-disarms after 30 seconds. Background/phone movement can trigger it; test with raised wheels and supervision. It does not reverse from camera gestures.

**Voice:** recognition depends on the phone's speech service and may require internet. Car voice commands move for approximately 700 ms; stop is always available. The firmware's independent 700 ms command watchdog also stops motors when commands cease. Firmware relay state can remain ON after a network failure; it is not a safety controller.

## Upload firmware and connect the real board

1. Install Arduino IDE 2.x.
2. Add `https://arduino.esp8266.com/stable/package_esp8266com_index.json` in Preferences → Additional Boards Manager URLs.
3. In Boards Manager install **esp8266 by ESP8266 Community 3.1.2**. Select **NodeMCU 1.0 (ESP-12E Module)** and the USB port.
4. Open the selected sketch from the table above. Each `.ino` lives in a same-named folder as required by Arduino IDE.
5. Set `WIFI_SSID` / `WIFI_PASS` for your router's 2.4 GHz network. Original reference credentials have deliberately not been retained.
6. Install the libraries listed below for that project, wire with power disconnected, then upload.
7. Open Serial Monitor at **115200 baud**. Note `Connect IP`.
8. If router connection fails after about 15 seconds, the board creates **RIT-<chip ID>**, password **rit8266demo**, address **192.168.4.1**. Change the demo password in firmware before deployment.
9. Connect the phone to the same local network as the board. For the fallback hotspot, select “stay connected” despite no internet. Disable VPN, guest/client isolation and automatic mobile-data switching if they interfere. Phone and board cannot be on separate isolated networks.
10. Open the Android app. Scan Wi-Fi and tap the board, or enter its numeric IP and tap Connect IP.
11. Check the green online badge and firmware profile, then open the matching project. Sensors poll every 2 seconds; DHT22 hardware samples every 2.5 seconds. No echo/invalid readings are unavailable, not zero.

Read `docs/PROJECTS.md` or each project's ⓘ page for full component lists and wiring. The included diagrams are labelled logical connection maps, not breadboard illustrations.

### Libraries

The ESP8266 core includes `ESP8266WiFi`, `ESP8266WebServer`, `WiFiUDP`, `Wire` and ESP8266 `Servo`. **Do not install an AVR-only Servo library over the ESP8266 core version.**

| Project | Additional Library Manager packages tested |
|---|---|
| LCD | LiquidCrystal I2C 1.1.2 (Frank de Brabander) |
| OLED | Adafruit SSD1306 2.5.13; Adafruit GFX Library 1.11.11; Adafruit BusIO 1.16.2 |
| DHT22 | DHT sensor library 1.4.6; Adafruit Unified Sensor 1.1.15 |
| Other projects | None beyond ESP8266 core |

The LiquidCrystal library emits an AVR architecture compatibility warning, but the supplied LCD sketch compiles successfully for ESP8266 using its Wire API. An actual LCD still needs the correct address, contrast setting and voltage interface.

## Electrical safety and pin accuracy

- Digital I/O is **3.3 V only**, not 5 V tolerant. GPIO numbers and NodeMCU D labels are not interchangeable.
- D0=GPIO16, D1=GPIO5, D2=GPIO4, D3=GPIO0, D4=GPIO2, D5=GPIO14, D6=GPIO12, D7=GPIO13, D8=GPIO15.
- These sketches avoid external use of boot-strapping D3, D4 and D8. Do not use flash GPIO6–11. RX/TX remain available for Serial diagnostics.
- Motor, servo and relay loads need suitable external supplies and common grounds. Do not power motors from GPIO or the NodeMCU 3V3 regulator. Keep a physical motor-power disconnect.
- Car wiring differs from the old reference: light is now D1 and horn D2. Do not reuse the old D8/D3 wiring.
- Use a transistor-input, low-current 3.3 V buzzer module; a bare high-current buzzer needs a driver transistor and suitable protection.
- HC-SR04 ECHO: **ECHO → 2.2 kΩ → D6 junction → 3.3 kΩ → GND**. This reduces a 5 V echo to about 3.0 V. Never attach raw ECHO to GPIO. Verify the sensor accepts a 3.3 V TRIG; add an appropriate AHCT buffer if your module requires a higher trigger.
- LCD: use a bidirectional I2C level shifter. LV=3.3 V, HV=5 V, common GND; ESP8266 on LV side and 5 V backpack on HV side. A 5 V LCD backpack can pull SDA/SCL to 5 V and must not connect directly.
- LDR: 3V3 → 3.3 kΩ limiter → LDR → junction → 10 kΩ → GND; junction through 1 kΩ to A0. This limits the divider voltage to below approximately 2.5 V. This wiring is only for a NodeMCU with a **verified ~3.2/3.3 V-range A0 input divider**. The bare chip's ADC maximum is 1.0 V and clones vary; if unknown, do not connect until measured/verified and adapted. Raw reading is not lux.
- For extra I/O use a 3.3 V MCP23017 (digital) or ADS1115 (analog), normally on D1/D2 I2C. These need new pin planning and firmware—not simply more wires. Car uses D1/D2 already, so it cannot add that bus unchanged.
- Demonstrate relay switching with low-voltage DC loads only. No household mains project instructions are supplied.
- Parking is an educational occupied/free indicator, not a vehicle collision-avoidance system.

## Build locally

Requirements: Node.js 22, Python 3, JDK 21, Android SDK platform 35/build-tools 35.0.0, Android SDK environment variables.

```sh
npm ci
npm run generate
npm test
npm run android:prepare
cd android
# macOS/Linux
chmod +x gradlew
./gradlew assembleDebug
# Windows: gradlew.bat assembleDebug
```

Output: `android/app/build/outputs/apk/debug/app-debug.apk`.

`android/` is supplied as source, including its Gradle wrapper. The prepare script is idempotent; it can also create the Android platform if absent. It copies the reference-derived native discovery activity, adds network/camera/microphone permissions and creates a self-contained vector launcher icon. The original icon-generation script required PNGs that were not attached, so it is not used.

Edit `scripts/generate.py` to change firmware or project hardware docs; run `npm run generate` to synchronize the ten `.ino` files and the app's embedded code. Edit `www/app.js` and `www/index.html` for the interface. Always run `npm run android:prepare` after changes before building the APK.

### Validation performed

- All **10 sketches compiled** for `esp8266:esp8266:nodemcuv2` using ESP8266 core 3.1.2. Logs: `docs/firmware-build.log`.
- Android debug APK **built successfully** using JDK 21 and SDK 35. Log: `docs/android-build.log`.
- JavaScript syntax and embedded-code consistency checks passed (`npm test`).
- Browser smoke test covered all 12 project guides/interfaces, IP handshake, drive/stop endpoint requests and all 5 sensor views with mocked board replies; no JS runtime errors. Log: `docs/browser-test.log`.
- **Not physically tested:** actual NodeMCU electronics, Wi-Fi radio/broadcast behavior on your router, Android camera/microphone permissions and speech service, or a specific phone installation. Compiling does not prove wiring is safe or sensors operate correctly. Carry out the bench checklist below before using moving parts.

Optional browser smoke test (development only): `npm install --no-save playwright`, `npx playwright install --with-deps chromium`, then `node tests/smoke.cjs`. This test's mocks exist only inside the test runner, not in the shipped app.

### Bench acceptance checklist

- [ ] Inspect supply voltages, all grounds and every required divider/level shifter with power disconnected.
- [ ] Upload one sketch; confirm Serial IP and valid `/ping` / `/status` JSON.
- [ ] Confirm manual IP connection, then UDP scan on the actual phone/network.
- [ ] Confirm the app refuses controls for a mismatched profile.
- [ ] Test car with wheels lifted; release controls, leave the screen and disable Wi-Fi to confirm automatic stop. Check physical power disconnect.
- [ ] Test each relay with a safe low-voltage load and verify off polarity/boot behavior.
- [ ] Test sensors with known stimuli and verify “unavailable” on sensor/no-echo errors.
- [ ] Verify info icons open the correct guide and copy-code matches the corresponding `.ino`.
- [ ] Check camera permission, arm/disarm, false motion and 30 second timeout; do not use near people/vehicles.
- [ ] Check speech service/microphone permission, mode selection and timed car movement.

## Protocol and security

See `docs/PROTOCOL.md`. HTTP and UDP are unauthenticated and unencrypted. Use a trusted isolated learning LAN. Never expose the device to the internet or port-forward it. Camera images are processed locally in app JavaScript; the phone's speech provider may process voice remotely. No analytics or cloud backend is included.

Debug artifacts are intended for testing and expire after 30 days in GitHub Actions. For public distribution, set up a private persistent signing key, release configuration, security hardening, device testing and the relevant store requirements.
