# Smart Farm "Gaston": IoT plant monitoring system

Team project, Master EEA (Université de Bourgogne). An end-to-end IoT system that monitors a thyme plant:
an Arduino reads the sensors, a Raspberry Pi running Node-RED collects the data, stores it in MySQL,
shows it on a live dashboard, waters the plant automatically and sends alerts by e-mail.

> **My role:** _describe your part here (e.g. Arduino firmware, Node-RED dashboard, database, wiring/hardware)._

![Node-RED dashboard](docs/images/dashboard.png)
<!-- Add a screenshot of the THYM Dashboard and a photo of the setup -->

## Architecture

```
 DHT11, light, soil moisture, gas (MQ), 2× HC-SR04
        │
        ▼
 Arduino Uno ── 16x2 I2C LCD (local display)
   │  ▲          LED / buzzer / pump relay (local alarms + watering)
   │  │  serial 9600 baud, one JSON line every 2 s
   ▼  │  pump commands back to the Arduino
 Raspberry Pi ─ Node-RED ──► MySQL (measurements, pump log, alert log)
                   │──────► Dashboard (gauges, 1 h charts, logs, weather)
                   │──────► E-mail alerts (cat detected, pump on too long)
                   └──────► Python weather script (city forecast, JSON)
```

## Features
- **7 measurements every 2 s:** air temperature and humidity, soil moisture, light, gas level, plant height (ultrasonic), cat presence (ultrasonic, checked every 100 ms)
- **Automatic watering:** the pump starts when the soil is dry, with manual override from the dashboard
- **Local alarms:** LED for any anomaly (temperature out of 0–35 °C, dry soil, gas ≥ 33 %, cat), buzzer when a cat comes close
- **Dashboard:** 6 gauges, 1-hour history charts, pump and alert logs, weather forecast cards, pop-up notifications
- **Data logging:** all measurements and events stored in MySQL
- **Alerts:** e-mail notifications, with a 10-minute cooldown on cat alerts
- **Weather:** Python script (current weather + 5-day forecast, JSON output) called from Node-RED

## Tech stack
| Layer | Tools |
|---|---|
| Embedded | Arduino Uno (C++), DHT11, HC-SR04, MQ gas sensor, I2C LCD, relay |
| Gateway | Raspberry Pi, Node-RED (serial, function, MySQL, e-mail, dashboard nodes) |
| Data | MySQL (phpMyAdmin / MySQL Workbench) |
| Scripts | Python 3 (`requests`, `argparse`) |

## Source code (team organisation)
| Part | Repository |
|---|---|
| Arduino firmware | [gaston_uno](https://github.com/IOT-Master-EEA-TSI/gaston_uno) |
| Raspberry Pi version (Python, MCP3008, dissolved-oxygen sensor) | [gaston_project](https://github.com/IOT-Master-EEA-TSI/gaston_project) |
| Node-RED dashboard | [interface](https://github.com/IOT-Master-EEA-TSI/interface) |
| Database schema | [Base-de-donnees](https://github.com/IOT-Master-EEA-TSI/Base-de-donnees) |
| Weather script | [script-API-meteo](https://github.com/IOT-Master-EEA-TSI/script-API-meteo) |

## Run it
1. Flash `gaston_v1.ino` to the Arduino Uno (libraries: `DHT`, `LiquidCrystal_I2C`).
2. Import `gaston_data_base.sql` into MySQL.
3. Import `flows.json` into Node-RED on the Raspberry Pi, set the serial port (`/dev/ttyACM0`) and the MySQL host.
4. Open the dashboard at `http://<raspberry-pi>:1880/ui`.
