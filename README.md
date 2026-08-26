# ESP32 E‑Paper Stock Ticker (Finnhub + GxEPD2)

The stock ticker is a low power “glanceable” stock quote display built on an ESP32 and a Waveshare 2.9 tri‑color e‑paper panel. It fetches live quote data from Finnhub over HTTPS, then parses JSON, and finally renders a price and daily change with color cues.

## Demo
- Photo/video: https://drive.google.com/file/d/1Ch2R4mWahz-3-0Cq4hb9GrInZbsJ_bEy/view?usp=sharing    

## Why this project
I wanted an embedded firmware project that allowed me to bridge the gap between software and hardware interface. I have always been curious about these things but have never done any individual work on them. I also wanted to learn how to manage constraints like networking in constrained environments, TLS tradeoffs, peripheral timing, and a clean UI on a slow refresh display which will be useful later on.

## Hardware
- MCU: ESP32 dev board
- Display: Waveshare 2.9" tri‑color e‑paper (GxEPD2_3C)
- Wiring (Waveshare → ESP32):
  - DIN → GPIO23
  - CLK → GPIO18
  - CS  → GPIO5
  - DC  → GPIO15
  - RST → GPIO2
  - BUSY→ GPIO4

## Software stack
- Tooling: PlatformIO + Arduino framework (C++)
- Networking: `WiFi`, `WiFiClientSecure` (HTTPS)
- Data: `ArduinoJson` (parse Finnhub quote JSON)
- Graphics: `Adafruit_GFX` + `GxEPD2_3C`
 
## Build & run
1. Clone the repo.
2. Create `include/secrets.h` (see `include/secrets.h.example`) and set:
   - `FINNHUB_TOKEN`
3. Flash with PlatformIO.
4. The device connects to Wi‑Fi, fetches `AAPL`, and updates the display.

## Key engineering decisions
- HTTPS: Used `WiFiClientSecure` for TLS so the API token isn’t sent in cleartext.
- Certificate handling: Used an “insecure” TLS mode during bring-up to reduce friction; next step is proper CA cert/pinning.
- E‑paper UI: Chose simple typography and a 2-line information hierarchy because refresh is slow and partial refresh behavior varies by panel.

## Debugging journey (what I learned)
- University Wi‑Fi constraints: Campus networks (IllinoisNet/eduroam) are WPA2‑Enterprise (802.1X), so a typical `WiFi.begin(ssid, password)` flow won’t work.
- Hotspot pitfalls: Phone hotspots can be inconsistent for embedded clients due to band/compatibility/power-saving behavior.
- Final solution: Registered the device MAC and used the campus guest/device workflow with `IllinoisNet_Guest` (open SSID). Verified end-to-end connectivity with an HTTP 200 test before reintroducing Finnhub + display rendering.
- Takeaway: Reduce the problem to layers (RF visibility → association → DHCP → DNS → HTTPS) and validate each layer with a minimal test.

## Results
- Stable Wi‑Fi connectivity on `IllinoisNet_Guest`
- Successful outbound HTTP and Finnhub HTTPS requests
- Clean e‑paper rendering: ticker, price, and day change (red for negative, green for positive)

## Next improvements
- Deep sleep + timed refresh for battery operation
- Proper TLS validation (CA bundle or certificate pinning)
- Multiple tickers + simple menu (buttons) or rotating display
- Caching + backoff on API errors; more robust JSON/body parsing
- Unit-testable parsing layer; separate “drivers” vs “app logic” modules

## Repo structure
- `src/main.cpp` – application firmware
- `include/secrets.h.example` – template (no real secrets)
- `docs/` – photos, wiring diagram, demo video link
