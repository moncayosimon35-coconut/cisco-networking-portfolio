# Lab 01: IPv6 SLAAC & EUI-64 Auto-Configuration in a Diamond Topology

## 📌 Project Overview
This repository documents the design, configuration, and verification of an IPv6 network arranged in a diamond/ring topology using **4 Cisco 2911 routers** and connected host PCs.

The main objective of this lab is to demonstrate **Stateless Address Autoconfiguration (SLAAC)** using the **EUI-64** algorithm to dynamically derive 64-bit host identifiers from each device's 48-bit physical MAC address.

---

## 📐 Network Topology
The network consists of 4 routers interconnected via point-to-point serial links (red cables), with end PCs connected directly to GigabitEthernet interfaces using crossover cables:

