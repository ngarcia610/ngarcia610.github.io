---
title: "vSphere 7"
description: "Notes for the VCP Exam"
date: 2023-09-11T18:04:05-04:00
lang: en
categories: ["Notes"]
tags: ["VMWare"]
draft: True
---

## VCSA Deployment

Stage 1

- Check DNS, and verify all ESXI hosts have forward and reverse lookup records, as well as the VCSA
- Open the installer on the DNS server (you will specify the host and all settings in the wizard)
- Go through and enter your settings. Remember that the disk must be larger than 30 GB.
- Go to the esxi host and create a new data store if you need additional space to install the vcsa
- Configure your network and FQDN
- Click next, finish, and install

Stage 2

- Setup NTP server
- Create logical datacenter
- Add hosts
- Remove from maintenance mode
- Suppress initial warnings
