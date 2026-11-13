# Arch Linux KDE Homelab

A personal homelab running on an Arch Linux laptop with KDE Plasma. It hosts Vaultwarden for password management, Samba for file sharing, KRDP for remote desktop access, and Wazuh for security monitoring. Tailscale provides private networking, with MagicDNS enabled and Mullvad Public DNS configured as a global nameserver.

Everything currently runs directly on the host. I don't use containers or virtual machines on this laptop.

## Hardware

| Component | Specification |
|---|---|
| Device | Laptop |
| CPU | Intel Core i5-1155G7 |
| RAM | 16 GB DDR4 |
| Storage | 512 GB SSD |
| Architecture | x86_64 |
| Operating System | Arch Linux |
| Desktop Environment | KDE Plasma |

The laptop originally had 12 GB of RAM. I upgraded it to 16 GB by opening the laptop and replacing a 4 GB memory stick with an 8 GB stick.

## Operating System

- **Distribution:** Arch Linux
- **Desktop Environment:** KDE Plasma
- **Kernel:** `7.2.15-arch1-1` (reported by `uname -a`; may change with updates)
- **Architecture:** x86_64
- **Package Manager:** pacman
- **Init System:** systemd
- **Firewall:** UFW
- **Containers:** None
- **Virtual Machines:** None currently installed

Arch is the host for my homelab services.

## Installation and Disk Encryption

The laptop originally ran Windows 11. I removed Windows and reclaimed the space previously occupied by its partitions for my Arch Linux installation.

I also configured LUKS2 encryption for the SSD and swap. Before making these changes, I backed up useful data to a client laptop and used ChatGPT to help work through the necessary cryptroot commands and configuration.

### Disk Security

- **Disk Encryption:** LUKS2
- **Swap Encryption:** Configured
- **Partition Layout:** Modified after removing Windows 11
- **Recovery Keys:** Excluded from this repository

Disk encryption helps protect data at rest, but it doesn't replace backups or protect data in the same way while the system is unlocked.

## Hosted Services

The following services currently make up the core of the homelab.

| Service | Purpose | Deployment |
|---|---|---|
| Vaultwarden | Self-hosted password management | Arch Linux host |
| Samba / SMB | Network file sharing | Arch Linux host |
| KRDP | Remote desktop access | Arch Linux host |
| Wazuh Indexer | Stores and indexes security data | Arch Linux host |
| Wazuh Manager | Processes security events and agent data | Arch Linux host |
| Wazuh Dashboard | Web interface for security monitoring | Arch Linux host |
| Wazuh Agent | Collects endpoint data | Client device |
| Tailscale | Private networking | Arch Linux and connected devices |
| Mullvad Public DNS | DNS resolution | Tailscale global nameserver |

All server-side services run directly on Arch Linux rather than inside containers or virtual machines.

### Vaultwarden

Vaultwarden provides a self-hosted, Bitwarden-compatible password management service.

- **Purpose:** Password and credential management
- **Deployment:** Native host installation
- **Access:** Master password
- **Data Location:** Omitted for privacy
- **Backups:** No regular backup routine currently configured

Vaultwarden stores sensitive credential data, making its database and associated files particularly important to protect. Establishing a reliable backup and recovery process is one of the main improvements I'd like to make.

### Samba / SMB File Sharing

Samba provides file sharing between compatible devices on the network.

- **Protocol:** SMB
- **Port:** TCP 445
- **Purpose:** Network file sharing
- **Remote Access:** Tailscale
- **Authentication:** Password-based
- **Shared Folders:** One shared folder
- **Backups:** No regular backup routine currently configured

SMB is intended for trusted devices rather than public internet access. Port 445 should not be exposed directly to the public internet.

### KRDP Remote Desktop

KRDP provides remote desktop access to my KDE Plasma system.

- **Purpose:** Remote desktop access
- **Deployment:** Arch Linux host
- **Authentication:** Username and password

Configuring and using KRDP provides practical experience with remote access to a Linux desktop and the basic administration of a remote-access service. Its setup was relatively straightforward, and I am still learning about its configuration and security considerations.

### Wazuh Security Monitoring

I run the Wazuh server-side components on the Arch Linux laptop, with a Wazuh Agent installed on one of my client devices.

| Component | Role |
|---|---|
| Wazuh Indexer | Stores and indexes security event data |
| Wazuh Manager | Processes agent data and security events |
| Wazuh Dashboard | Provides the monitoring and analysis interface |
| Wazuh Agent | Collects and forwards endpoint data |

The server-side components are installed directly on Arch Linux using packages from the AUR. The agent runs on a separate client device.

I encountered package conflicts and configuration issues during setup and used ChatGPT to help troubleshoot them. I haven't documented every configuration change, so this section describes the current architecture rather than a complete reproduction guide.

Wazuh is part of my ongoing exploration of Linux security monitoring and endpoint visibility. This deployment has given me hands-on experience installing server-side monitoring components and connecting a separate endpoint through an agent. I am still learning how to interpret alerts and understand the events collected by the platform, and I do not yet have substantial experience investigating security incidents.

The limited number of alerts that appear critical or clearly malicious does not establish that the configuration is correct or that the environment is free of threats.

Because the Indexer, Manager, and Dashboard share the same laptop, they also share its availability and hardware resources.

## Networking

### Tailscale

Tailscale provides private connectivity between authorized devices and allows me to access services without publishing their IP addresses in this repository.

- **Purpose:** Private networking and remote access
- **Device Name Resolution:** Tailscale MagicDNS
- **IP Addresses:** Intentionally omitted

I use Tailscale to access SMB remotely and connect devices within my tailnet.

### DNS

Mullvad Public DNS is configured as a global nameserver in the Tailscale admin console, and MagicDNS remains enabled.

- **Global Nameserver:** Mullvad Public DNS
- **Device Name Resolution:** Tailscale MagicDNS
- **DNS Configuration:** Tailscale admin console
- **Pi-hole:** No longer used
- **Unbound:** No longer used

MagicDNS provides names for devices on the tailnet, while Mullvad Public DNS serves as the configured global nameserver. Actual query handling depends on Tailscale's DNS settings and client behavior.

I previously experimented with other DNS tools, but the current setup is simpler and better suited to what I need.

### Network Architecture

The following diagram shows the general layout without exposing IP addresses or detailed network configuration.

```text
                 +----------------------+
                 |   Client Devices     |
                 |                      |
                 |   Wazuh Agent        |
                 |   Tailscale          |
                 +----------+-----------+
                            |
                       Tailscale
                            |
                 +----------v-----------+
                 | Arch Linux KDE       |
                 | Laptop               |
                 |                      |
                 | Vaultwarden          |
                 | Samba / SMB          |
                 | Wazuh Indexer        |
                 | Wazuh Manager        |
                 | Wazuh Dashboard      |
                 | UFW                  |
                 +----------------------+

                 DNS:
                 Tailscale MagicDNS
                 Mullvad Public DNS
                 (Global Nameserver)
```

This is a conceptual diagram, not a complete representation of packet flow or DNS resolution.

## Security

### Firewall

UFW manages the host firewall rules.

- **Firewall:** UFW
- **Purpose:** Control network access to the laptop
- **Services Requiring Access Controls:** Vaultwarden, SMB, and Wazuh
- **Public Port Forwarding:** Not documented

Firewall rules should permit only the traffic needed for the services in use. Remote access should be restricted appropriately, particularly for SMB and administrative interfaces.

### General Security Practices

- Use Tailscale for private remote connectivity.
- Restrict SMB access to trusted devices and networks.
- Keep Arch Linux and installed software updated.
- Protect Vaultwarden data and credentials.
- Review firewall rules and listening services periodically.
- Avoid committing passwords, API tokens, private keys, or encryption recovery material.
- Keep sensitive service configuration out of the public repository.

## Maintenance and Backups

I manage the system using Arch Linux's native tools, including pacman and systemd. Maintenance is performed as needed, and I still have work to do on documenting recovery procedures.

- **Package Management:** pacman
- **Service Management:** systemd
- **Firewall Management:** UFW
- **System Restore Utility:** Available but not regularly used
- **Regular Backups:** Not currently configured
- **Recovery Testing:** Not currently performed

### Backup Status

I don't currently have a regular backup routine for Vaultwarden, shared files, or system configuration.

This is one of the biggest weaknesses of the current setup. SSD failure, filesystem corruption, or accidental deletion could result in permanent data loss.

Planned improvements:

- [ ] Back up Vaultwarden's database and associated data.
- [ ] Back up important shared files.
- [ ] Keep copies of essential system configuration.
- [ ] Store backups on a separate device or storage destination.
- [ ] Test restoration instead of assuming a backup works.

The system restore utility is not treated as a verified backup solution for hosted services or personal data.

## Past Experiments and Lessons Learned

I've experimented with several DNS and virtualization tools while building this setup. Some were removed because they introduced unnecessary complexity, conflicts, or performance problems.

These experiments are documented for context; they are not part of the current configuration.

### Pi-hole

I previously installed Pi-hole to experiment with network-wide DNS filtering and ad blocking.

I eventually stopped using it and switched to Mullvad Public DNS through Tailscale's global nameserver configuration.

Pi-hole was removed during a later virtualization experiment after I ran into DNS-related conflicts involving dnsmasq. I didn't find Pi-hole particularly useful for my needs, so I didn't bring it back.

### Unbound

I also experimented with Unbound as part of my DNS setup.

I stopped using it after experiencing occasional performance issues and finding that some website elements were being blocked more aggressively than I wanted.

I didn't retain enough configuration details to determine the exact cause of those issues. Unbound is no longer part of the setup.

### KVM/QEMU, virt-manager, and ESET VPN

At one point, I experimented with running a Tiny10 virtual machine using KVM/QEMU and a graphical virtual machine manager.

I tried to route Tailscale exit-node traffic through an ESET VPN running inside the VM. The goal was to send traffic through the VPN inside the guest, but the resulting setup was too slow to be practical.

The experiment involved additional virtual networking and DNS configuration, and I eventually removed the virtualization software and related components.

During this process, I also encountered conflicts involving Pi-hole and dnsmasq, which contributed to removing Pi-hole.

I no longer use this setup, and I didn't preserve enough configuration details to recreate the original routing arrangement accurately.

### Arch Configuration and Troubleshooting

Some parts of this homelab took a fair amount of trial and error. I used ChatGPT to help troubleshoot Arch Linux package conflicts, disk encryption configuration, DNS behavior, and virtual networking.

I don't remember every command or configuration change from those experiments, so I avoid presenting them as reproducible instructions.

One lesson from the process is that adding more networking tools doesn't necessarily make a setup better. For my current needs, Tailscale, MagicDNS, Mullvad Public DNS, and UFW cover the main networking requirements without the additional DNS and VM components I previously experimented with.

## Known Limitations

- The homelab runs on a laptop rather than dedicated server hardware.
- Service availability depends on the laptop's power, uptime, and network connectivity.
- Wazuh's server-side components share the same host and its available resources.
- Services run directly on the host without container isolation.
- No regular backups are currently configured.
- Recovery procedures are not fully documented.
- Some historical networking configurations were not preserved.
- Wazuh alert interpretation and monitoring are still areas I'm learning.

## Future Improvements

- [ ] Establish a reliable backup and recovery strategy.
- [ ] Document Vaultwarden's data storage and restoration procedure.
- [ ] Improve my understanding of Wazuh alerts and monitoring capabilities.
- [ ] Review UFW rules and exposed services.

## Privacy

This repository intentionally excludes:

- Public, private, and Tailscale IP addresses
- Passwords, API tokens, and encryption recovery keys
- Sensitive service configuration
- Private device identifiers
- Personal files and sensitive filesystem details

## Notes

This homelab is an ongoing personal project built around a daily-use Arch Linux laptop. It combines self-hosted services, private networking, endpoint monitoring, and disk encryption.

The setup has evolved through experimentation and troubleshooting. Not every past configuration was retained, and some areas, especially backups and recovery, still need improvement.

This README documents the current architecture and known limitations rather than pretending the system is more polished or resilient than it is.
