---
layout: post
title:  "Homelab Aspirations (July 2024)"
categories: linux iot homelab 
excerpt_separator: <!--more-->
---

I have all these ideas for projects, not much time to execute. Getting them out of my head to start prioritising.
<!--more-->

Working environment

- Install Linux/Unix on MacBook
- Docker and KVM for MacBook

Networking enhancements

- Pi-hole (HA, with redundancy eg. using keepalived)
- Ethernet cabling through the house
  - Server room to Tim's office x2
  - Server room to Anja's office x2
- Fibre cable to the garage
- Rack for server room

Homelab expansion

- Setup mini PC with Debian or other headless OS
  - shellinabox
  - docker
- Turn old ISP routers into productive members of society
  - k3s
- Google Cloud free tier
- Oracle Cloud free tier
- NAS for deep freeze backups

Services

- Data collection:
  - timeseries DB (eg. InfluxDB)
  - data sources:
    - solar power generation, whole house power usage
    - rain gauges
    - BoM weather forecast
    - electricity usage
    - temperature/humidity sensors
  - backup to NAS
- iCloud backup to S3
- Strava data injector
- dashboard
  - weather
  - solar
  - energy
