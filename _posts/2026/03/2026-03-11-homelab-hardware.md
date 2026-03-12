---
layout: post
title: 'My Homelab Hardware'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
pub_date: '2026-03-11'
---

I don't have a lot of hardware for my homelab. I think that is a good thing.

<!--more-->

## What Hardware Do I Need?

For any homelab in its infancy, I think the fewer pieces of hardware to manage, the better. I have
felt overwhelmed on more than one occasion so keeping things simple is a priority for me. My main goal
is to setup an environment where I can safely play with new things. For that I probably only need
a single server. Let's see where I am starting.

## Networking Equipment

As I mentioned in my previous post, I have a Unifi controller/router and it is connected directly
to my Internet service provider. From there I have a Unifi switch that distributes my LAN to
all the computers in my home office. I have 2 VLANs that I use. One for my work PC and one for everything
else in the room (My IoT devices, wifi, etc. are all on separate VLANs).

The "Work" VLAN has only my work PC on it. It is isolated from the rest of the network. No one can access
it and it can't access anything else. The other VLAN I call the "Trusted" VLAN is where my homelab
devices exist today (including my Home PC I'll be using to access the homelab). I plan on creating
a "Homelab" VLAN to connect the devices, providing some isolation for the rest of my network.

## Devices

I have the following devices I plan on using in my homelab.

- A Mini PC I run a Minecraft server on. It has 8C/16T, 32 GB RAM and 1TB of NVME SSD storage.
- A Raspberry Pi (Pi 5, 8GB RAM, POE+ hat, 256 GB NVME SSD).
- A Unifi managed 2.5 GbE switch
- A Unifi 2-bay NAS
- A CyberPower UPS (EC650LCD)

 ```mermaid
graph TD
    ISP["🌐 ISP"]                    
    ROUTER["Unifi Router / Controller"]        
    subgraph WORK_VLAN["Work VLAN (Isolated)"]
        WORKPC["🖥️ Work PC"]
    end

    subgraph TRUSTED_VLAN["Trusted VLAN (Trusted)"]
        HOMEPC["🖥️ Home PC"]
    end

    subgraph HOMELAB_VLAN["Homelab VLAN"]
        SW["🔀 Unifi Switch (2.5 GbE)"]
        MINIPC["🖥️ Mini PC (Proxmox)"]
        PI["🍓 Raspberry Pi 5"]
        NAS["💾 Unifi NAS"]

        SW -- "2.5 Gbps" --- MINIPC
        SW -- "1 Gbps" --- PI
        SW -- "2.5 Gbps" --- NAS
    end

    ISP --> ROUTER
    ROUTER --> TRUSTED_VLAN
    ROUTER --> HOMELAB_VLAN
    ROUTER --> WORK_VLAN
```

## How I (Currently) Plan to Use These Devices

The Mini PC was overkill for just running a Minecraft server. I plan on wiping it clean and installing
Proxmox as a virtualization hypervisor. I will run my Minecraft server as a VM. I plan on using this
server to host VMs for experiments with containers, AI tools and more.

The Raspberry Pi could be used for a whole host of things. Initially I will run Docker containers
on this device. Lightweight containers like NGINX Proxy Manager, Homepage, and Portainer. I might
also use it to host Home Assistant, when the time comes.

I already have a NAS in my home network, but I rely on it for backing up my family's files. I don't want
to start playing with it in my homelab and risking loosing any important files. I have a second Unifi
2 bay NAS I will use for my homelab. It will be where I backup Proxmox to, and all the Docker files
from the Raspberry Pi.

While I do have a 1 GbE switch currently connecting the devices to my LAN, I plan on using a separate
2.5 GbE switch to connect all the Homelab devices. Except for the Raspberry Pi, all the devices support
2.5 GbE, which should provide a fast, stable network for the devices to communicate with each other.

## Good Enough

I feel that for my situation, this initial setup is more than adequate. I have my homelab equipment
isolated from my home and work equipment. This gives me the safety I am looking for. I have a place
to experiment with virtual environments and host containers and AI tools. And there is room to grow in the
future. Let's have some fun.
