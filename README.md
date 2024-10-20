# SAHS
Simple Alpine based home server, a.k.a DIY NAS

# Overview
## Features & Inspiration
This repository contains instructions on how to create a simple home server. Features:
- Lightweight & responsive linux basis (Alpine Distro)
- Local network shares for Windows, MacOs, Linux, using SMB
- Backups via a offsite machine outside (to be implemented)
- Docker environment for services
- Basic configuration that works
- customizable and many learning opportunities for the interested
  
The basic installation instructions are ~~stolen~~ inspired by the excellent guide 3.0 of [dither8](https://dither8.xyz/guide/alpine-linux-nas-3/ "Link to dither8's website"). This repository is meant as a backup of his instructions, and extends them with advanced docker services, and offsite backup.

## Quick links
| Part | Description & Link|
| ----------- | ----------- |
| 00 | [Overview and ReadMe](https://github.com/pubbelfiesch/SAHS/) |
| 01 | [Alpine installation until the server can be remotely managed via SSH](part_1.md) | 
| 02 | [Setup of local network shares via SMB, permissions, and file transfer from existing HDDs](part_2.md) |
| 03 | [Setup of a self hosted VPN for remote access to the files, and networking](part_3.md) |
| 04 | [Installation of Docker, NGINX reverse proxy, IMMICH, and Nextcloud (optional step)](part_4.md) |
| 05 | [Setup of a secondary machine for off-site backups](part_5.md) |
| 99 | [Philosophical considerations, and cursed knowledge, reflecting why the nas is build as it is](part_99)

## Architecture
![NAS architecture overview](/images/architecture/homeserver_architecture.drawio.svg)

# Status

This repository is currently being gradually build up. Part 1 and 2 are considered technically complete. Use at your own risk.
## Open topics:
- Implement Backup strategy, Project "No, there is another"
- Test backup strategy, Projekt "The Clone Wars"
- Find service to synchronise files between 4-5 devices without corruption, Projekt "Dont fall for the dark side *nextcloud*"
- update documentation, Project "Perhabs the archives are incomplete"
