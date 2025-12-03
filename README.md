# Snort IDS Home Lab 🛡️

## Overview  
This repository documents my journey building a fully functional home lab using **Snort** — for intrusion detection (IDS) and intrusion prevention (IPS). The lab is designed for hands-on learning: you can simulate attacks, observe network traffic, and see real-time alerts.

I built this lab to climb deeper into network security and SOC-style defensive work — turning theory into practice.  

## Why Snort?  
- Snort is open-source & free — ideal for a personal lab with no licensing constraints. :contentReference[oaicite:1]{index=1}  
- It supports both IDS (detection) and IPS (prevention) modes. :contentReference[oaicite:2]{index=2}  
- Handles multiple protocols (TCP, UDP, HTTP, FTP, DNS, etc.), enabling diverse attack simulations. :contentReference[oaicite:3]{index=3}  
- Allows custom rule writing — perfect for learning and testing scanning & exploitation detection. :contentReference[oaicite:4]{index=4}  

## Lab Topology  

```

<img width="720" height="367" alt="image" src="https://github.com/user-attachments/assets/6af50525-2c37-43bd-8cf1-4e9ea3431632" />


````

- **Ubuntu Server** — runs Snort, monitors traffic between attacker and victim. :contentReference[oaicite:5]{index=5}  
- **Attacker (Kali)** — launches scanning & exploitation against victim. :contentReference[oaicite:6]{index=6}  
- **Victim (Metasploitable)** — hosts vulnerable services (HTTP, FTP, SSH, etc.) for testing. :contentReference[oaicite:7]{index=7}  

*All VMs are isolated in a host-only / NAT network — no risk of external interference.* :contentReference[oaicite:8]{index=8}  

## Requirements  

- **Hardware:** ≥ 8 GB RAM, ≥ 30 GB storage :contentReference[oaicite:9]{index=9}  
- **Software / VMs:**  
  - Ubuntu Server (for Snort)  
  - Kali Linux (attacker)  
  - Metasploitable (victim)  
  - VirtualBox (or any similar hypervisor) :contentReference[oaicite:10]{index=10}  

## Setup & Installation  

```bash
# Update system  
sudo apt update -y && sudo apt upgrade -y  

# Install Snort  
sudo apt-get install snort -y  
````

During installation, specify the network interface Snort should listen on (e.g. `enp0s8`). ([Medium][1])

### Configuration

1. Create a backup of the original config:

   ```bash
   sudo cp /etc/snort/snort.conf /etc/snort/test_snort.conf
   ```
2. Edit `test_snort.conf` — configure `HOME_NET` to your lab subnet (e.g. `192.168.56.0/24`). ([Medium][1])
3. Point Snort to use custom rules: ensure `local.rules` is included. ([Medium][1])
4. Disable or comment out default/community rules (optional — useful for custom learning). ([Medium][1])
5. Test configuration:

   ```bash
   sudo snort -T -c /etc/snort/test_snort.conf
   ```

## Example Custom Rules

```text
# Detect ICMP ping sweep (Nmap -sn)
alert icmp any any -> 192.168.56.0/24 any (msg:"Nmap ICMP Ping Sweep Detected"; icode:0; itype:8; sid:1000002; rev:1;)

# Detect Nmap ACK scan
alert tcp any any -> 192.168.56.0/24 any (msg:"Nmap ACK Scan Detected"; flags:A; sid:1000003; rev:1;)

# Detect TCP SYN scan
alert tcp any any -> 192.168.56.0/24 any (msg:"TCP SYN Scan Detected"; flags:S; threshold:type both, track by_src, count 20, seconds 60; sid:1000004; rev:1;)
```

These rules can detect basic scanning techniques such as ping sweeps, SYN scans, ACK scans, etc. ([Medium][1])

## Running the Lab

```bash
sudo snort -c /etc/snort/test_snort.conf -i <interface> -A console -l /var/log/snort
```

From your attacker VM (Kali), run scan commands:

```bash
nmap -sn -PE <victim_IP>        # ICMP ping sweep  
nmap -sS -Pn <victim_IP>        # SYN scan  
nmap -sA -Pn <victim_IP>        # ACK scan  
```

Watch Snort alerts in console or log files — confirm detection. ([Medium][1])

## What I Learned & Use Cases

* How to configure Snort from scratch, including custom network settings.
* Writing custom detection rules and understanding rule syntax.
* Detecting network reconnaissance and scanning by attackers.
* Building a realistic lab environment for SOC-style training.

This lab helped me turn theory into practical skills — a great first step toward real-world network defence.


[1]: https://medium.com/%40dhanushin787/building-my-snort-ids-home-lab-a4a425adc767 "Building My Snort IDS Home Lab. I often hear experts say that… | by Dhanush Dhayalan | Nov, 2025 | Medium"
