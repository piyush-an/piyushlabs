---
title: "Buying vs Building an Ambient Sensor"
date: 2026-08-15
draft: false
author: "Piyush Anand"
description: "I bought a Govee hygrometer and built an ESP32 + DHT22 sensor for Home Assistant. Here is how they compare."
tags: ["home-assistant", "esp32", "3d-printing"]
categories: ["home-automation"]
ShowToc: true
TocOpen: false
---

**TL;DR:** I wanted temperature and humidity monitoring in two rooms. Rather than buy two sensors, I bought one (the Govee H5075) and built the other myself using an ESP32 and DHT22. The bought sensor was working in fifteen minutes. The built one took days of prototyping, CAD work, and print iterations. Both ended up on my Home Assistant dashboard side by side. One path was fast and cheap. The other was a rabbit hole I thoroughly enjoyed falling into.

[![Buy Vs Build](/images/temp_sensor_cover.jpg)](/images/temp_sensor_cover.jpg)

---

## Why I Did Both

I already run [Home Assistant](https://www.home-assistant.io/) as my unified home automation setup. Adding temperature and humidity monitoring to two rooms was my next step, both for comfort tracking and for automations.

When picking sensors, my constraint was simple: WiFi or Bluetooth only. I don't have a Zigbee hub. I had two spaces to cover, and instead of buying two, I thought this was a good opportunity to actually build one myself and see how the experience compared.

---

## The Bought Path: Govee H5075

### Picking the sensor

My selection criteria were pretty minimal: Bluetooth or WiFi connectivity, a reasonable price, and solid Home Assistant support. The [Govee Indoor Hygrometer Thermometer H5075](https://us.govee.com/products/govee-bluetooth-hygrometer-thermometer-h5075) checked all those boxes at around $10. It's a compact unit with a display, runs on AAA batteries, and communicates over Bluetooth.

### Getting it into Home Assistant

I added the device through Home Assistant's built-in Bluetooth integration and the H5075 showed up automatically once it was powered on and in range. No custom integration, no YAML editing.

Once paired, it exposed four sensors immediately:

- Temperature
- Humidity
- Battery level
- Bluetooth signal strength

From unboxing to a live reading on my dashboard, it took roughly fifteen minutes.

[![Govee](/images/govee_config.png)](/images/govee_config.png)


---

## The Build Path: ESP32 + DHT22 + ESPHome

This is where things got interesting.

### Parts

I ordered everything from AliExpress to keep costs low. Here's what I used:

| Part | Notes |
|---|---|
| ESP32-D dev board | WiFi built in, plenty of GPIO |
| DHT22 sensor | More accurate than the DHT11, worth the small price difference |
| Perfboard | For the final soldered assembly |
| Jumper wires | For prototyping on the breadboard |
| Soldering iron + solder | Essential for the final build |
| M2 brass heat-set inserts | For a clean screw-in lid on the enclosure |
| M2 screws | To close it all up |

The total came to more than the Govee, though part of that cost is tools that now live in my drawer for future projects.

### Prototyping on the breadboard

The circuit itself is simple: the DHT22 data pin connects to GPIO4, with 3.3V and GND completing the circuit. Nothing exotic, but it's satisfying when you actually see the sensor start reporting data.

[![ESP32 Config](/images/pin_out.png)](/images/pin_out.png)


### ESPHome configuration

I used the web-based [ESPHome](https://esphome.io/) Device Builder to create the config, which it flashed directly to the ESP32 over USB. After that, all future updates happen wirelessly over OTA. Here's the configuration:

<details>
<summary>View full ESPHome configuration</summary>

```yaml
esphome:
  name: ambientsensor
  friendly_name: AmbientSensor

esp32:
  board: esp32dev
  framework:
    type: esp-idf

logger:

api:
  encryption:
    key: !secret api_encryption_key  # ESPHome generates this on first setup

ota:
  - platform: esphome
    password: !secret ota_password

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  ap:
    ssid: "Ambientsensor Fallback Hotspot"
    password: !secret ap_password  # Used only if WiFi connection fails

captive_portal:

sensor:
  - platform: wifi_signal
    name: "AmbientSensor WiFi Signal"
    update_interval: 60s

  - platform: uptime
    name: "AmbientSensor Uptime"

  - platform: dht
    pin: GPIO4
    model: DHT22
    temperature:
      name: "Ambient Temperature"
    humidity:
      name: "Ambient Humidity"
    update_interval: 30s
```

</details>


Once flashed, Home Assistant discovered the device automatically through the ESPHome integration, and the same four sensor types appeared: temperature, humidity, WiFi signal strength, and uptime.

[![ESP32 Config](/images/esp32_config.png)](/images/esp32_config.png)

---

## Designing and Printing an Enclosure

Getting the sensor working was one thing. Leaving a bare ESP32 and a DHT22 dangling off a breadboard on a shelf felt wrong.

I wanted a proper enclosure, which meant revisiting CAD skills I hadn't touched since undergrad, roughly a decade ago. I was surprised how quickly it came back. I designed a two-part case: a body to hold the perfboard and a lid with a honeycomb cutout pattern for airflow, since the DHT22 needs to breathe to give accurate readings.

The design is shared on [Thingiverse](https://www.thingiverse.com/thing:7399138)

<video width="100%" controls autoplay muted loop playsinline>
  <source src="/images/6x8_perfboard_animation.mp4" type="video/mp4">
</video>

### Four print iterations

Getting to a finished enclosure took four attempts. The body and lid fit came together reasonably quickly, but the micro-USB slot refused to cooperate. I kept getting the dimensions wrong. The tolerance was just slightly off each time. After the fourth print I gave up trying to dial it in perfectly and used a knife to open the slot by two millimetres. Not elegant, but it works.

The brass heat-set inserts screw into the perfboard mounting posts and hold the board securely in place. The lid snaps on cleanly and can be removed for any future rework.

<video width="100%" controls autoplay muted loop playsinline>
  <source src="/images/esp32_movie.mov" type="video/mp4">
</video>

---

## Both Sensors on the Dashboard

With everything assembled and flashed, I added both devices to my Home Assistant dashboard as side-by-side cards. Both sensors are now live and reading roughly the same values, which was a good sanity check that the DHT22 is calibrated in the right ballpark. After all the iterations, seeing both sensors finally sitting in the same dashboard felt oddly satisfying.

[![Home Assistant dashboard with both sensors](/images/both_final.jpg)](/images/both_final.jpg)


[![Home Assistant dashboard with both sensors](/images/ipad_sensor.png)](/images/ipad_sensor.png)

---

## What I Learned

Here's the comparison:

| | Bought (Govee H5075) | Built (ESP32 + DHT22) |
|---|---|---|
| Cost | ~$10 | More (ESP32 + DHT22 + materials) |
| Time to working | ~15 minutes | Days (prototype, CAD, prints, solder) |
| Connectivity | Bluetooth | WiFi |
| Update interval | Govee-controlled | Configurable (I use 30s) |
| Skills required | None | Soldering, ESPHome YAML, basic CAD |
| Skills gained | Minimal | Soldering, enclosure design, print tolerancing |
| Satisfaction level | Plug and play | Strong sense of accomplishment |

The buy-vs-build framing undersells what building actually gives you. Yes, it took longer and cost more. But along the way I re-learned CAD, picked up soldering confidence, got comfortable with ESPHome, and came away with a handful of ideas for what to build next.

A few things I want to improve in a future revision:

- Get that USB slot tolerance right from the start. I now know to add at least 1-2mm clearance
- Explore running the ESP32 on a LiPo battery so it can sit anywhere without needing a USB cable

If you're debating whether to build your first ESP32 project, my honest take is: do it. The Govee is the better sensor for someone who just wants data fast, but the build is the better project for someone who wants to learn.


---
