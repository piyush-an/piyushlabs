---
title: "Never Miss a Delivery Again: Building a Package Alert"
date: 2026-08-23
draft: false
author: "Piyush Anand"
description: "After getting a package stolen from my apartment lobby, I built a delivery alert system using Change Detection, MQTT, Node-RED, and Google Home speakers to announce arrivals"
tags: ["home-assistant", "homelab", "node-red", "mqtt"]
categories: ["home-automation"]
ShowToc: true
TocOpen: false
---


**TL;DR:** A package got stolen from my apartment lobby. No camera, no delivery API, no tracking webhook. So I built my own. Change Detection polls the carrier's public tracking page every five minutes, publishes a message over MQTT when the status changes, and Node-RED routes that message to every Google Home speaker in my apartment. The moment a package is marked delivered, the speakers announce it.

---

## The Problem

It finally happened to me.

<video width="60%" controls playsinline>
  <source src="/images/package_stolen.mp4" type="video/mp4">
</video>


A package from AliExpress was delivered in my apartment lobby and was gone by the time I got home. Had it been from Amazon, I might have had a shot at a refund. This one? No chance. I had already waited two weeks for it to arrive. Reordering would mean another full month of waiting, only to end up paying more than the Amazon price would have been.

I decided right then that I was not going to let it happen again.

<img src="/images/smart.gif" alt="Alt text" width="100%">

Living in an apartment rules out the obvious fix. I cannot mount a camera in the lobby. That's a problem for when I eventually get a house. For now, I needed a software solution that could tell me the moment a package hit my doorstep.

---

## Exploring Options

My packages come from five places primarily: Amazon, FedEx, USPS, GoFo, and SpeedX. The first thing I looked into was whether any of these carriers offered a tracking API for retail customers. A webhook that fired when a package status changed would be perfect. Turns out, none of them expose a public API at the consumer level. That door was closed before I even knocked.

Next, I explored services like [Global Package Tracking](https://parcelsapp.com/) and [17TRACK](https://www.17track.net/en) that aggregate tracking across carriers into a single dashboard. I got genuinely excited for a moment. One portal for everything sounded ideal. But after testing a few, the delay killed it. These services poll the carriers and cache the results. By the time the status appears in the aggregator, it's already old news. I needed the freshest possible update, ideally right from the carrier's own tracking page.

Back to square one.

---

## The Solution: Change Detection

While browsing for alternatives, I came across [Change Detection](https://changedetection.io/). The idea is simple: give it a URL, and it watches that page for any change. Common uses include price alerts on product pages, monitoring news sites, or tracking availability of sold-out items. For my use case, it was exactly the right tool: I wanted to know the moment a carrier's tracking page updated from "out for delivery" to "delivered."

Change Detection is open source and self-hostable. I already run a homelab on Proxmox, so spinning it up as a container took a few minutes.

---

## Setting It Up

### Amazon's Shareable Tracking Link

Amazon has one feature that made this particularly clean: the app lets you generate a public, shareable tracking link for any order. No login required to view it, which means I could point Change Detection at it without any authentication headaches or session management.

I grabbed the link, pasted it into Change Detection, and used the browser inspect tool to identify the specific HTML element that contains the delivery status text. By targeting just that section, ignoring the header, footer, and ads, I reduced the noise significantly and made the change detection more precise.

The polling interval is set to every five minutes, which felt like the right balance between responsiveness and not hammering the page.

<!-- [![Change Detection tracking setup](/images/delivery_changedetection_setup.png)](/images/delivery_changedetection_setup.png) -->

### Sending a Notification via MQTT

For the notification, I didn't want a push notification to my phone. I wanted something ambient, something I'd hear even if my phone was across the room or I was in the middle of something. I have Google Home Mini speakers in a few rooms, and I wanted to use them.

The bridge between Change Detection and the speakers is MQTT. When Change Detection detects a change on the tracking page, it fires a webhook to a Home Assistant MQTT publish endpoint. That drops a message onto a topic, a simple pub/sub queue that the rest of the system listens to.

---

## The Notification Pipeline

Here's the full flow from carrier website to my ears:

[![Flow Chart](/images/alert_flow.png)](/images/alert_flow.png)

### Node-RED Flow

[Node-RED](https://nodered.org/) handles the routing. It's a low-code, visual flow builder that runs as a service in my homelab. My flow is straightforward: an MQTT subscriber node listens on the topic, and when a message arrives, it fans out to three Google Home media player nodes, one for each speaker.

[![Node-RED delivery alert flow](/images/node_red_flow.png)](/images/node_red_flow.png)

Each Google Home node is configured to announce the message text out loud. The announcement plays on all three speakers simultaneously so I hear it wherever I am in the apartment.

---

## The First Real Test

The real test came on the next Amazon delivery day.

[![Tracking page showing out for delivery](/images/out_for_delivery.png)](/images/out_for_delivery.png)

Change Detection was polling every five minutes. The package went out for delivery in the morning. A little while later, the carrier updated the tracking page to "Delivered", and within minutes, I heard this:

<video width="100%" controls playsinline>
  <source src="/images/delivery_alert.mp4" type="video/mp4">
</video>

The speakers announced the delivery across the apartment. I ran downstairs immediately. I don't think I move that fast when the fire alarm goes off. And there it was, package collected within five minutes of it landing in the lobby.

[![Tracking page showing delivered status](/images/delivered.png)](/images/delivered.png)

<a href="/images/delivery_timeline.png"><img src="/images/delivery_timeline.png" alt="Tracking page showing time of delivery" width="50%"></a>

Change detected in 3 minutes.

[![Change Detector Tracks](/images/change_detected.png)](/images/change_detected.png)

---

## What Works and What Doesn't

After testing across all five carriers, the results were mixed but still useful:

| Carrier | Works? | Notes |
|---|---|---|
| Amazon | Yes | Works |
| GoFo | Yes | Works |
| SpeedX | Yes | Works |
| FedEx | No | Bot detection blocks Change Detection |
| USPS | No | Bot detection blocks Change Detection |

FedEx and USPS both have more sophisticated bot mitigation on their tracking pages. Change Detection isn't able to reliably get through those. Honestly, credit where it's due: their engineering is better. It's a bit ironic that Amazon, with all its resources, leaves the tracking page open enough for this to work.

For my day-to-day deliveries, Amazon and GoFo cover the majority, so the setup solves the problem it was built for.

---

## Closing Thoughts

When I look back at what I built, the whole thing is a chain of off-the-shelf tools stitched together with a bit of configuration.

The bigger takeaway is something older than any of these tools: **necessity is the mother of invention**. It took losing a package to actually sit down and build one. Now the whole system runs in the background and I don't think about it on delivery days, until I hear the speakers announce a delivery and remember why I built it.


---
