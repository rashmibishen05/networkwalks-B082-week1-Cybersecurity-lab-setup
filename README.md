# 🐉 Kali Linux Lab Setup

> **A clean, isolated Kali Linux lab environment using Oracle VirtualBox**

This guide walks through setting up a Kali Linux security lab with a **VirtualBox NAT Network**, configuring Kali's network connectivity, and creating a clean snapshot that can be restored whenever an experiment goes wrong.

---

## 🗺️ Lab Overview

```text
                         🌐 Internet
                             │
                             │
                    ┌────────▼────────┐
                    │  Host Machine   │
                    │    Windows      │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │    VirtualBox   │
                    │   NAT Network   │
                    │   Lab-Network   │
                    │  10.0.0.0/24    │
                    └────────┬────────┘
                             │
                      ┌──────▼──────┐
                      │ Kali Linux  │
                      │  10.0.0.3   │
                      └─────────────┘
```

### 🔧 Lab Configuration

| Component       | Configuration     |
| --------------- | ----------------- |
| 🖥️ Hypervisor  | Oracle VirtualBox |
| 🐉 OS           | Kali Linux 2026.2 |
| 🌐 Network Type | NAT Network       |
| 🔌 Network Name | `Lab-Network`     |
| 📡 Network      | `10.0.0.0/24`     |
| 🚪 Gateway      | `10.0.0.1`        |
| 📦 DHCP         | Enabled           |
| 💻 Kali IP      | `10.0.0.3`        |
| 🧠 RAM          | 2 GB              |
| ⚙️ CPUs         | 2                 |
| 💾 Disk         | ~80 GB            |

---

# 01 · 🌐 NAT Network Setup

The first step is to create an isolated VirtualBox network for the lab.

### Open VirtualBox Network Manager

In **Oracle VirtualBox Manager**, navigate to:

**Network → NAT Networks**

Create a new NAT Network with the following configuration:

```text
Name:          Lab-Network
IPv4 Prefix:   10.0.0.0/24
DHCP:          Enabled
```

Your network should look similar to:

```text
┌─────────────────────────────────────┐
│          Lab-Network                │
├─────────────────────────────────────┤
│ Network:     10.0.0.0/24            │
│ Gateway:     10.0.0.1               │
│ DHCP:        Enabled                │
└─────────────────────────────────────┘
```

### 💡 Why NAT Network?

A **NAT Network** allows lab VMs to:

* 🌐 Access the internet
* 🔄 Communicate with other VMs on the same lab network
* 🛡️ Remain separated from the physical LAN
* 🧪 Provide a controlled environment for security testing

> **Tip:** Use **NAT Network**, not the regular **NAT** adapter mode, when you want multiple lab VMs to communicate with one another.

---

# 02 · 🐉 Kali Linux Setup

## Create / Import the Kali VM

Create a new VM or import the official Kali VirtualBox image.

Recommended configuration:

```text
Name:        kali-linux-2026.2-virtualbox-amd64
OS Type:     Debian (64-bit)
Memory:      2048 MB
Processors:  2
Disk:        ~80 GB
```

### ⚙️ Configure the Network Adapter

Open:

**Kali VM → Settings → Network → Adapter 1**

Configure:

```text
Enable Adapter:     ✓
Attached to:        NAT Network
Network Name:       Lab-Network
```

The important part is:

```text
Adapter 1
   │
   └── NAT Network
          │
          └── Lab-Network
```

Start the Kali VM once the configuration is complete.

---

# 03 · 🔌 Kali Network Configuration

After Kali boots, open a terminal.

## Check the Network Interface

Run:

```bash
ip addr
```

or:

```bash
ip -4 addr
```

Look for an Ethernet interface such as:

```text
eth0
```

or:

```text
enp0s3
```

---

## Check the Kali IP

Run:

```bash
ip addr
```

The lab configuration should provide an address in:

```text
10.0.0.0/24
```

Example:

```text
10.0.0.3
```

---

## Check the Default Gateway

Run:

```bash
ip route
```

You should see something similar to:

```text
default via 10.0.0.1
```

This confirms that Kali is using the VirtualBox NAT Network gateway.

---

## Check DNS

Run:

```bash
cat /etc/resolv.conf
```

Confirm that a working DNS server is configured.

Example from this lab:

```text
192.168.1.1
```

---

## 🧪 Test Connectivity

### 1. Test the VirtualBox Gateway

```bash
ping -c 4 10.0.0.1
```

### 2. Test Internet Connectivity

```bash
ping -c 4 8.8.8.8
```

### 3. Test DNS Resolution

```bash
ping -c 4 google.com
```

A successful setup should pass all three tests:

```text
Gateway      ✓
Internet     ✓
DNS          ✓
```

---

## 🔎 Quick Network Verification

Use these commands whenever you need to troubleshoot the lab:

```bash
ip addr
ip route
cat /etc/resolv.conf
```

Expected configuration:

```text
┌─────────────────────────────────────┐
│       Kali Network Settings         │
├─────────────────────────────────────┤
│ IP Address : 10.0.0.3               │
│ Network    : 10.0.0.0/24            │
│ Gateway    : 10.0.0.1               │
│ DNS        : 192.168.1.1            │
└─────────────────────────────────────┘
```

> ⚠️ **Note:** With DHCP enabled, the Kali IP address may change after rebooting. If your lab requires a predictable IP, use a DHCP reservation or configure a static address.

---

# 04 · 📸 Kali Snapshots

Snapshots are one of the most useful features of a virtual security lab.

Before installing tools, changing network settings, or running potentially disruptive experiments, create a snapshot.

## 🧼 Create the Baseline Snapshot

Once Kali is completely configured:

1. Shut down Kali cleanly.
2. Open **Oracle VirtualBox Manager**.
3. Select the Kali VM.
4. Open **Snapshots**.
5. Create a new snapshot.

Use:

```text
Name:
Fresh Installation
```

Suggested description:

```text
Clean Kali Linux installation.
Lab-Network configured and network connectivity verified.
Use as the baseline recovery point.
```

Your snapshot tree should look like:

```text
📸 Fresh Installation
└── ▶ Current State
```

---

## 🔬 Recommended Snapshot Strategy

Create a new snapshot before major changes:

```text
📸 Fresh Installation
        │
        ├── 📸 Before Tool Installation
        │
        ├── 📸 Before Network Testing
        │
        └── 📸 Before Lab Exercise
```

Good snapshot names should describe **what state the VM is in**, not just the date.

Examples:

```text
Fresh Installation
Before Metasploit Lab
Before Network Configuration
Before Tool Installation
Clean Lab Baseline
```

---

## ♻️ Restore the Lab

If an experiment breaks the VM:

1. Power off Kali.
2. Open **VirtualBox → Snapshots**.
3. Select the snapshot you want.
4. Click **Restore**.
5. Start Kali again.

For example:

```text
Experiment
    │
    ▼
💥 Something breaks
    │
    ▼
♻️ Restore "Fresh Installation"
    │
    ▼
🐉 Clean Kali Environment
```

---

# ✅ Final Setup Checklist

### 🌐 Network

* [ ] Create `Lab-Network`
* [ ] Configure `10.0.0.0/24`
* [ ] Enable DHCP
* [ ] Verify gateway `10.0.0.1`

### 🐉 Kali

* [ ] Install/import Kali Linux
* [ ] Configure Adapter 1
* [ ] Attach Adapter 1 to `Lab-Network`
* [ ] Boot Kali successfully

### 🔌 Connectivity

* [ ] Verify Kali IP address
* [ ] Verify default gateway
* [ ] Verify DNS
* [ ] Ping `10.0.0.1`
* [ ] Ping `8.8.8.8`
* [ ] Test `google.com`

### 📸 Snapshots

* [ ] Shut down Kali
* [ ] Create `Fresh Installation`
* [ ] Add a useful snapshot description
* [ ] Create additional snapshots before major lab changes

---

# 🎯 Final Result

At the end of the setup, you should have:

```text
                 🌐 INTERNET
                     │
                     ▼
             ┌───────────────┐
             │   VirtualBox  │
             │ NAT Network   │
             └───────┬───────┘
                     │
              Lab-Network
               10.0.0.0/24
                     │
                     ▼
             ┌───────────────┐
             │  🐉 KALI LINUX │
             │   10.0.0.3    │
             └───────┬───────┘
                     │
                     ▼
             📸 Fresh Installation
                 Snapshot
```

> **🏁 Lab Ready:** Kali is configured, networking is verified, and a clean recovery point is available. You can now safely build additional lab machines and begin your security exercises.
