# Snort Nmap Detection Lab

> Goal: run Snort on an Ubuntu VM (Defender) and detect Nmap scans and ICMP from a Kali attacker VM. 

---

## Prerequisites

* Virtualization: VirtualBox / VMware or physical machines on the same network
* If using Vagrant: Vagrant installed
* 2 VMs (or machines) on the same network:

  * **Ubuntu Server** (Defender) — runs Snort
  * **Kali Linux** (Attacker) — runs Nmap / ping
* Sufficient privileges to install packages and run packet capture (sudo)

---

## Quick setup (manual steps)

### 1. Update & install Snort (Ubuntu Defender)

```bash
sudo apt update && sudo apt upgrade -y
# Install Snort (simple apt method)
sudo DEBIAN_FRONTEND=noninteractive apt install -y snort
```

During the apt install interactive prompt, it will ask for the network interface — enter the primary interface (or configure later in `/etc/snort/snort.conf`).

If you prefer building Snort from source for the latest version, replace the apt steps with source build steps.

### 2. Put the interface into promiscuous mode (if needed)

Replace `ens33` with your interface name (find with `ip a`).

```bash
sudo ip link set ens33 promisc on
```

### 3. Verify or set `HOME_NET` in Snort config

Open `/etc/snort/snort.conf` and set a `HOME_NET` that matches your lab network. Examples:

```conf
# Single host:
ipvar HOME_NET 192.168.1.100

# Or whole subnet:
ipvar HOME_NET 192.168.1.0/24
```

> Note: In `test_snort.conf` included here we set `var HOME_NET` to a lab subnet for convenience.

### 4. Add local detection rules

Create or edit `/etc/snort/rules/local.rules` (or update your repo copy `snort/local.rules` and copy into the VM). Example contents:

```snort
# Detect ICMP ping (ping sweep / pings)
alert icmp any any -> $HOME_NET any (msg:"ICMP Packet Detected"; sid:1000001; rev:1;)

# Detect Nmap SYN scan (flags:S)
alert tcp any any -> $HOME_NET any (msg:"Nmap SYN Scan Detected"; flags:S; sid:1000002; rev:1;)

# Detect Nmap ACK scan (flags:A)
alert tcp any any -> $HOME_NET any (msg:"Nmap ACK Scan Detected"; flags:A; sid:1000003; rev:1;)
```

Ensure `snort.conf` (or `test_snort.conf`) includes:

```conf
var RULE_PATH /etc/snort/rules
include $RULE_PATH/local.rules
```

### 5. Test Snort configuration

```bash
sudo snort -T -c /etc/snort/snort.conf
# or for the repo test file:
sudo snort -T -c /etc/snort/test_snort.conf
```

You should see messages indicating parsing succeeded and no fatal errors.

### 6. Run Snort (console / sniffer mode)

Find the correct interface (e.g. `ip a`), then:

```bash
sudo snort -i ens33 -A console -c /etc/snort/snort.conf -l /var/log/snort
```

* `-i ens33` — capture on your network interface
* `-A console` — print alerts to console
* `-l /var/log/snort` — log directory

---

## Attack simulation (from Kali attacker)

SSH into or open a terminal in your Kali VM and run one (or all) of the following against the Defender IP:

```bash
# Replace 192.168.1.100 with your Defender (Ubuntu) IP

# SYN stealth scan
nmap -sS 192.168.1.100

# ACK scan
nmap -sA 192.168.1.100

# Aggressive scan (service detection + version)
nmap -A 192.168.1.100

# Ping/ICMP test
ping -c 4 192.168.1.100
```

---

## Expected Snort console output (example)

If Snort and rules are configured correctly, alerts will appear similar to:

```
[**] [1:1000002:1] Nmap SYN Scan Detected [**]
[Priority: 0] 
11/07-12:34:56.789012 192.168.2.200:12345 -> 192.168.1.100:80
TCP TTL:64 TOS:0x0 ID:54321 IpLen:20 DgmLen:40
***S
```

And for ICMP:

```
[**] [1:1000001:1] ICMP Packet Detected [**]
```

Additionally, logs are written to `/var/log/snort`.


---


## License

This README and accompanying repo files are released under the MIT License. See `LICENSE` for details.
