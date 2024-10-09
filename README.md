# SAHS
Simple Alpine based home server, a.k.a DIY NAS

# Overview
This repository contains instructions on how to create a simple home server. Features:
- Lightweight & responsive linux basis (Alpine Distro)
- Local network shares for Windows, MacOs, Linux, using SMB
- Backups via a offsite machine outside (to be implemented)
- Docker environment for services
- Basic configuration that works, and offers many learning opportunities for those interested
  
Most of them are ~~stolen~~ inspired from the excellent guide 3.0 of [dither8](https://dither8.xyz/guide/alpine-linux-nas-3/ "Link to dither8's website"). This repository is meant as a backup, and documents the changes specific to my setup and use-case.

| Part | Description & Link|
| ----------- | ----------- |
| 00 | [Motivation and other considerations for this project](part_0.md) |
| 01 | [Alpine installation until the server can be remotely managed via SSH](part_1.md) | 
| 02 | [Setup of local network shares via SMB, permissions, and file transfer from existing HDDs](part_2.md) |
| 03 | [Setup of a self hosted VPN for remote access to the files, and networking](part_3.md) |
| 04 | [Installation of Docker, NGINX reverse proxy, IMMICH, and Nextcloud (optional step)](part_4.md) |
| 05 | [Setup of a secondary machine for off-site backups](part_5.md) |
