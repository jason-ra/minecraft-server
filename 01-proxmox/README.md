# Proxmox
## Summary
Running Proxmox allows the PC to be used for multiple purposes. Running one virtual machine with Ubuntu and Minecraft Server, and other future VMs doing different things.

I want the Minecraft Server to operate in its own, isolated network so it can be exposed to the Internet for friends to join without risking the rest of the network.

```
  +-------------------------------------+
  | PC                                  |
  |    +-----------+                    |
  |    |   VM      |                    |
  |    | Minecraft |                    |
  |    +-----T-----+                    |
  |          | vNIC (VLAN 30)           |
  |          | 192.168.30.30            |
  |  +-------------------------------+  |
  |  |      Proxmox Hypervisor       |  |
  |  +--------------T----------------+  |
  |                 | Mgmt (VLAN 20)    |
  |                 | 192.168.20.20     |
  +-------------------------------------+
                    | Physical LAN port
                    | 802.1Q Tagged VLAN 20,30
                    |
         +----------------------+
         |       Network        |
         +----------------------+
```

## Install Guide
### Prepare bootable USB
- Download ISO https://www.proxmox.com/en/downloads
- Download Rufus to create bootable USB from ISO https://rufus.ie/en/
- Open Rufus and create bootable USB from the Proxmox ISO

### Prepare network
- Create new VLANs
  - VLAN 20 - Mgmt - (192.168.20.0/24)
  - VLAN 30 - DMZ - (192.168.30.0/24)
- Add VLAN tags to designated port of switch
- Temporarily add untagged VLAN of the LAN network

### Install Proxmox
- Plug in USB
- Boot PC and load boot menu (F7 for me, maybe F11 for you?)
- Follow prompts
  - Set locale
  - Set admin password
  - Set management IP address
- After reboot, switch to another computer and open web browser to Proxmox management IP e.g. https​://192.168.20.20:8006

### Configure updates
- Select the node/host
- From Updates, open Repositories
- Disable the two enterprise repositories
- Add repository: No-Subscription

### Configure network
In my case, physical port is "eno1"
- Select the node/host
- From System, open Network
- Set vmbr0 Linux Bridge to VLAN Aware = checked
- Create Linux VLAN
  - Name: vmbr0.20
  - IP: 192.168.20.20/24
  - Gateway: 192.168.20.1
  - Comment: Mgmt
- Create Linux VLAN #2
  - Name: vmbr0.30
  - IP: 192.168.30.0/24
  - Comment: DMZ

Example result in /etc/network/interfaces:

```
auto lo
iface lo inet loopback

iface eno1 inet manual

auto vmbr0
iface vmbr0 inet manual
        bridge-ports eno1
        bridge-stp off
        bridge-fd 0
        bridge-vlan-aware yes
        bridge-vids 2-4094

auto vmbr0.30
iface vmbr0.30 inet static
        address 192.168.30.0/24
#dmz

auto vmbr0.20
iface vmbr0.20 inet static
        address 192.168.20.20/24
        gateway 192.168.20.1
#mgmt

source /etc/network/interfaces.d/*
```

## References
Thanks to the following inspiration:

- [Digital Mirror](https://www.youtube.com/@DigitalMirrorComputing): Let's install Proxmox 8.3 in 2025: From Scratch. Spelled out. https://www.youtube.com/watch?v=kqZNFD0JNBc

- [Tech Tutorials - David McKone](https://www.youtube.com/@TechTutorialsDavidMcKone): How To Create VLANs in Proxmox For a Single NIC https://www.youtube.com/watch?v=ljq6wlzn4qo
