# Media Server

Simple Media Server from an Old Laptop.

---

## Table of Contents

- [Hardware](#hardware)
- [Network Topology](#network-topology)
- [Installation](#installation)
- [Initial System Checks](#initial-system-checks)
- [Firewall and Network Validation](#firewall-and-network-validation)
- [Network Setup](#network-setup)
- [Server Configuration (Lid Behavior)](#server-configuration-lid-behavior)
- [Storage Layout](#storage-layout)
- [Samba Setup (File Sharing)](#samba-setup-file-sharing)
- [SMB Troubleshooting](#smb-troubleshooting-what-actually-fixed-it)
- [Jellyfin Streaming](#jellyfin-streaming)
- [Remote Access with Tailscale](#remote-access-with-tailscale)
- [Mobile Access](#mobile-access)
- [Takeaways](#takeaways)
- [Future Plans](#future-plans)

---

## Hardware

The server runs on an old Dell Vostro laptop. It's nothing fancy just repurposed hardware that would otherwise collect dust.

![Hardware Setup - Dell Vostro Laptop](images/laptops.jpg)

---

## Network Topology

Before diving into the setup, here's the overall network architecture showing how the media server fits into the home network.

![Home Lab Network Diagram](images/homelabdiagram.png)

---

## Installation

Before installation, I entered the BIOS, verified the boot mode, and booted from a USB installer.

![Ubuntu Server Installer Boot Screen](images/choosing%20ubunutu%20boot%20.jpg)

During installation:

- The entire disk was wiped and reused for Ubuntu Server

![Ubuntu Entire Disk Setup](images/ubuntu_entire-disk.jpg)

- Wi-Fi was configured (Ethernet wasn't practical in my room)

![Wi-Fi Configuration During Installation](images/wifi.jpg)

- A minimal installation was selected
- Username and password were created

![Setting Username and Password](images/setting%20username%20and%20password.jpg)

- The installation proceeded after completing all configuration steps

![Installing Ubuntu Server](images/installing%20ubuntu.jpg)

After installation completed, the system rebooted into Ubuntu Server.

At this point, I unplugged the keyboard and display. From here on, the laptop was managed entirely over the network. This was the moment the laptop stopped behaving like a personal computer and started behaving like a server.

---

## Initial System Checks

Once connected to the server, the first thing I did was check for updates and bring the system fully up to date.

![Ubuntu Package Update](images/updating%20ubuntu.png)

I also checked the IP address assigned to the server so I could confirm network connectivity from my main laptop.

![IP Address Output](images/sambaipaddress.png)

---

## Firewall and Network Validation

Before exposing any services, I verified the basics:

- Firewall (UFW) status
- Listening ports
- Network connectivity

UFW was checked to confirm the firewall state with appropriate rules in place.

![UFW Status Output](images/checking%20active%20ufw%20status.png)

---

## Network Setup

The server connects wirelessly through a secondary router.

**Why Wi-Fi?** The primary router sits in an awkward spot, and running Ethernet across the room wasn't practical. Wi-Fi was good enough to get started.

**Why a dynamic IP instead of static?** I don't have access to the primary router to configure static leases. Tailscale makes this irrelevant by assigning consistent internal addresses through its mesh network.

The secondary router keeps server traffic isolated and reduces the risk of disrupting the main network.

---

## Server Configuration (Lid Behavior)

Because this is still laptop hardware, power behavior needed adjustment. Closing the lid should not suspend the system.

The configuration was changed so the screen blanks while the server continues running normally.

![Lid Configuration - logind.conf](images/lidconfig.png)

This single change made the setup feel intentional rather than temporary.

With the lid closed, the server continues running quietly in the background, fully accessible over the network.

![Laptop with Lid Closed - Server Running](images/picture%20of%20laptop%20lid%20close.jpg)

---

## Storage Layout

Before deploying any services, storage was organized into a simple structure:

- One location for general files and backups
- One location dedicated to media content

Separating these early avoided permission issues later when services needed access to specific directories.

![Folder Structure Output](images/folderstructure.png)

---

## Samba Setup (File Sharing)

To enable file access from my main laptop, Samba was installed and configured.

After configuration, the server appeared like a normal shared drive on the network.

![Samba Configuration File (smb.conf)](images/smbconf.png)

The Samba configuration was set to restrict access to only one specific user for security.

![Samba Access Restricted to One User](images/giving%20accer%20to%20only%20one%20user%20in%20smbconfig.png)

After editing configuration files, the Samba service was restarted and its status verified.

![Samba Service Running](images/sambaserver.png)

From Windows File Explorer on the main laptop, I connected using the server's local IP and share name.

![SMB Connected on Main Laptop](images/smbdconnect.png)

Only the main laptop was allowed access, keeping the share restricted.

---

## SMB Troubleshooting (What Actually Fixed It)

Samba was the most time-consuming part of the setup. Multiple "Access Denied" errors occurred before everything finally worked.

**What fixed it:**

- Explicitly adding a Samba password for the Linux user
- Clearing cached Windows credentials
- Fixing directory ownership
- Using the correct username format when logging in

Each issue on its own was small, but together they made Samba feel deceptively difficult.

![Setting Samba Password](images/after%20editing%20samba%20putting%20password.png)

---

## Jellyfin Streaming

With file sharing working, Jellyfin was installed.

The service was enabled, status verified, and accessed from the main laptop through a browser.

![Jellyfin Authentication Page](images/authencationpage.png)

![Jellyfin Dashboard](images/dashboardjelly.png)

After pointing Jellyfin to the media directory and allowing it to scan, media became available almost immediately.

![Jellyfin Home Page](images/homejelly.png)

![Jellyfin Home Pane View](images/homepane.png)

Even over Wi-Fi, playback was smooth on both laptop and phone. The Celeron handled the workload better than expected.

![Jellyfin Video Streaming](images/streaming%20in%20jellyfin.png)

---

## Remote Access with Tailscale

Everything worked locally but remote access was the next challenge.

Instead of port forwarding or dynamic DNS, I chose Tailscale.

Tailscale was installed, enabled, and authenticated. Once connected, the server appeared as part of a private mesh network.

![Tailscale Downloaded and Active](images/tailscale%20downlaod%20and%20active.png)

Using Tailscale's MagicDNS, I could access Jellyfin from other devices on the internet as if I were at home.

No open ports. No firewall gymnastics. No exposed services.

![Jellyfin Accessed Remotely via Tailscale](images/remoteaccess.png)

---

## Mobile Access

To allow mobile access to Jellyfin, I used Tailscale's sharing feature with a shared account. This provides secure remote access while limiting functionality.

**Key Setup:**

The mobile device is connected through a **shared Tailscale account** that has access restricted to only port **8096** (Jellyfin's default port).

![Sharing Tailscale Access](images/sharing%20tailwind.png)

![Checking Shared Account Configuration](images/chekcing%20shared%20account%20homeserver%20do%20i%20share%20.png)

**Access Control Lists (ACLs)** were configured to ensure the shared user can only access the Jellyfin streaming port, nothing else on the server.

![Tailscale Access Control List](images/accesscorntrol%20list%20in%20tailwind.png)

![Adding Port-Specific Access Control](images/adding%20new%20access%20control%20to%20give%20only%20one%20port%20to%20shared%20users.png)

**Mobile Setup:**

Tailscale was downloaded and set up on the mobile device.

![Downloading Tailscale on Mobile](images/downloading%20tailscale%20in%20mobile.jpg)

Once connected, the Jellyfin dashboard became accessible from the phone.

![Mobile Login Success](images/mobile%20login%20sucess.jpg)

![Mobile Dashboard](images/mobile%20dashboard.jpg)

![Jellyfin Dashboard on Mobile](images/jellyfin%20dashboard%20with%20mobile.jpg)

This setup allows streaming from anywhere while keeping the server secure. The shared account can only see port 8096 no file shares, no SSH, nothing else.

---

## Takeaways

This project was never about building something impressive. It was about understanding:

- How systems behave when they run continuously
- How small configuration mistakes break stable services
- How much value can be extracted from old hardware

The Dell Vostro now runs quietly in the background with the lid closed and the screen off. Samba handles file sharing. Jellyfin handles media streaming. Tailscale handles secure remote access.

What started as curiosity turned into a home server that actually gets used daily.

---

## Future Plans

- Add stricter firewall rules with UFW
- Flash OpenWRT on the router
- Upgrade hardware and migrate to Proxmox
- Move to Cloudflare Tunnels with a custom domain
- Add monitoring and automated backups

The setup will continue to evolve but for now, it does exactly what it was built to do.

---

## Currently Included Images (34 total)

| Image File                                                            | Description                          | Section Used          |
| --------------------------------------------------------------------- | ------------------------------------ | --------------------- |
| `laptops.jpg`                                                         | Hardware - Dell Vostro laptop        | Hardware              |
| `homelabdiagram.png`                                                  | Network topology diagram             | Network Topology      |
| `choosing ubunutu boot .jpg`                                          | BIOS/boot menu with Ubuntu selected  | Installation          |
| `ubuntu_entire-disk.jpg`                                              | Disk partitioning during install     | Installation          |
| `wifi.jpg`                                                            | Wi-Fi configuration screen           | Installation          |
| `setting username and password.jpg`                                   | User creation during install         | Installation          |
| `installing ubuntu.jpg`                                               | Ubuntu installation progress         | Installation          |
| `updating ubuntu.png`                                                 | Package update output                | Initial System Checks |
| `sambaipaddress.png`                                                  | IP address output                    | Initial System Checks |
| `checking active ufw status.png`                                      | UFW firewall status                  | Firewall Validation   |
| `lidconfig.png`                                                       | logind.conf lid behavior config      | Server Configuration  |
| `picture of laptop lid close.jpg`                                     | Laptop with lid closed, running      | Server Configuration  |
| `folderstructure.png`                                                 | Directory structure output           | Storage Layout        |
| `smbconf.png`                                                         | Samba smb.conf configuration         | Samba Setup           |
| `giving accer to only one user in smbconfig.png`                      | Samba user access restriction        | Samba Setup           |
| `sambaserver.png`                                                     | Samba service status                 | Samba Setup           |
| `smbdconnect.png`                                                     | Windows File Explorer SMB connection | Samba Setup           |
| `after editing samba putting password.png`                            | Setting Samba password               | SMB Troubleshooting   |
| `authencationpage.png`                                                | Jellyfin login/auth page             | Jellyfin Streaming    |
| `dashboardjelly.png`                                                  | Jellyfin dashboard view              | Jellyfin Streaming    |
| `homejelly.png`                                                       | Jellyfin home page                   | Jellyfin Streaming    |
| `homepane.png`                                                        | Jellyfin home pane view              | Jellyfin Streaming    |
| `streaming in jellyfin.png`                                           | Jellyfin video playback              | Jellyfin Streaming    |
| `tailscale downlaod and active.png`                                   | Tailscale installation and status    | Remote Access         |
| `remoteaccess.png`                                                    | Jellyfin accessed via Tailscale      | Remote Access         |
| `sharing tailwind.png`                                                | Tailscale sharing configuration      | Mobile Access         |
| `chekcing shared account homeserver do i share .png`                  | Shared account verification          | Mobile Access         |
| `accesscorntrol list in tailwind.png`                                 | Tailscale ACL configuration          | Mobile Access         |
| `adding new access control to give only one port to shared users.png` | Port-specific ACL for shared users   | Mobile Access         |
| `downloading tailscale in mobile.jpg`                                 | Tailscale mobile app download        | Mobile Access         |
| `mobile login sucess.jpg`                                             | Successful mobile login              | Mobile Access         |
| `mobile dashboard.jpg`                                                | Jellyfin on mobile                   | Mobile Access         |
| `jellyfin dashboard with mobile.jpg`                                  | Jellyfin dashboard mobile view       | Mobile Access         |
| `sshtoserver.png`                                                     | (Not used in documentation)          | -                     |
