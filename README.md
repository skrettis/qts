# qts — Quick Test Script

**Quick Test Script** for checking the **QEMU Guest Agent** status in Proxmox VMs.

`qts` helps verify whether the guest agent is enabled, reachable, and returning useful VM information such as hostname, OS details, IP addresses, filesystems, guest time, and logged-in users.

## Features

- Checks if the VM exists on the Proxmox host
- Verifies Proxmox guest-agent configuration
- Tests guest-agent connectivity
- Shows VM hostname and OS information
- Displays guest IP addresses
- Shows mounted filesystems
- Converts guest-agent time from raw epoch values to readable 24-hour time
- Converts logged-in user login times to readable timestamps
- Pretty-prints JSON output when `jq` is installed
- Provides simple troubleshooting hints

## Requirements

Run this script on a **Proxmox host**.

Required:

```bash
qm
```

Recommended:

```bash
jq
```

Install `jq` on Proxmox:

```bash
apt update
apt install -y jq
```

Inside the VM, the QEMU Guest Agent should be installed and running.

For Ubuntu/Debian guests:

```bash
sudo apt update
sudo apt install -y qemu-guest-agent
sudo systemctl enable --now qemu-guest-agent
```

## Installation

Clone the repository:

```bash
git clone https://github.com/skrettis/qts.git
cd qts
```

Make the script executable:

```bash
chmod +x qts
```

Optional: install it system-wide:

```bash
sudo mv qts /usr/local/bin/qts
```

## Usage

Run:

```bash
qts <VMID>
```

Example:

```bash
qts 100
```

Or, if running from the repository folder:

```bash
./qts 100
```

## Example output

```text
============================================================
 qts — Quick Test Script
 Proxmox QEMU Guest Agent Test - VMID 100
============================================================

------------------------------------------------------------
VM status
------------------------------------------------------------
status: running

------------------------------------------------------------
Proxmox guest-agent config
------------------------------------------------------------
agent: enabled=1

------------------------------------------------------------
Ping
------------------------------------------------------------
Command: qm agent 100 ping

✓ PASS

------------------------------------------------------------
Guest time
------------------------------------------------------------
Command: qm agent 100 get-time

✓ PASS
Raw:   1777324503000000000
Local: 2026-04-27 23:15:03 CEST
UTC:   2026-04-27 21:15:03 UTC

------------------------------------------------------------
Network / IP addresses
------------------------------------------------------------
Command: qm agent 100 network-get-interfaces

✓ PASS

Detected addresses:
  ens18: ipv4 192.168.1.50/24
  ens18: ipv6 fe80::1234:abcd:5678:90ef/64

------------------------------------------------------------
Summary
------------------------------------------------------------
✓ Guest agent is responding.
```

## Enabling the Guest Agent in Proxmox

If `qts` says the guest agent is not configured or not responding, enable it on the Proxmox host:

```bash
qm set <VMID> --agent enabled=1
qm shutdown <VMID>
qm start <VMID>
```

Example:

```bash
qm set 100 --agent enabled=1
qm shutdown 100
qm start 100
```

A full shutdown/start is recommended. A normal reboot from inside the VM may not attach the guest-agent device correctly.

## Common fixes

### Guest agent is not responding

On the Proxmox host:

```bash
qm set <VMID> --agent enabled=1
qm shutdown <VMID>
qm start <VMID>
```

Inside the Ubuntu/Debian VM:

```bash
sudo apt update
sudo apt install -y qemu-guest-agent
sudo systemctl enable --now qemu-guest-agent
```

### IP addresses are not shown

Check that the guest agent is running inside the VM:

```bash
systemctl status qemu-guest-agent
```

Then from the Proxmox host:

```bash
qm agent <VMID> network-get-interfaces
```

### Output is hard to read

Install `jq` on the Proxmox host:

```bash
apt update
apt install -y jq
```

## Useful Proxmox commands

Ping the guest agent:

```bash
qm agent <VMID> ping
```

Show guest network interfaces:

```bash
qm agent <VMID> network-get-interfaces
```

Show guest OS info:

```bash
qm agent <VMID> get-osinfo
```

Show guest filesystems:

```bash
qm agent <VMID> get-fsinfo
```

Show guest time:

```bash
qm agent <VMID> get-time
```

Show logged-in users:

```bash
qm agent <VMID> get-users
```

## Notes

`qm agent get-time` and some `get-users` login timestamps may be returned as raw epoch values in seconds, milliseconds, microseconds, or nanoseconds. `qts` detects the unit and converts the value into readable 24-hour timestamps.

## License

MIT
