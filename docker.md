Docker + Docker Compose Documentation
Pi-hole, Unbound, and Nextcloud Stack on My Homelab Server

Author: Maddox Wise
Course: CYB 3353 – System Administration
Assignment: Docker Application Deployment
Date: 11-16-25

1. Overview

For this assignment, I deployed a Docker-based application stack on my Homelab Linux server using Docker and Docker Compose. I chose software applications that I personally use for my homelab networking setup:

Unbound – a validating, recursive DNS resolver

Pi-hole – a network-wide DNS sinkhole and ad-blocker

Nextcloud – a self-hosted cloud storage and productivity platform

I created a docker-compose.yml file, organized persistent data directories, and verified that each service runs successfully. This documentation is meant to allow another student to recreate my setup on their own system.

2. System Environment

Host System: Ubuntu Server (Homelab VM)

Architecture: x86_64

Working Directory: /home/maddox/

Files Created: docker-compose.yml plus persistent volumes in subfolders

3. Installing Docker & Docker Compose

Docker and Docker Compose were already installed on my homelab server.
For reference, installation typically looks like:

sudo apt update
sudo apt install docker.io docker-compose-plugin -y
sudo systemctl enable docker --now


Verify install:

docker --version
docker compose version

4. Folder Structure

Before deploying containers, I created directories to store persistent config and data:

/home/maddox/
 ├── unbound/
 ├── pihole/
 │    ├── etc-pihole/
 │    └── etc-dnsmasq.d/
 └── nextcloud/
      ├── config/
      └── data/


These folders hold container data so settings survive updates and reboots.

5. Docker Compose File

Here is the exact docker-compose.yml file used:

version: "3.9"

services:
  ###############################################################
  # UNBOUND – Local DNS Resolver
  ###############################################################
  unbound:
    image: mvance/unbound:latest
    container_name: unbound
    restart: unless-stopped
    ports:
      - "5335:53/udp"
    volumes:
      - /home/maddox/unbound:/etc/unbound

  ###############################################################
  # PI-HOLE – Network Ad Blocking
  ###############################################################
  pihole:
    image: pihole/pihole:latest
    container_name: pihole
    restart: unless-stopped
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "8081:80"
    environment:
      - TZ=America/Chicago
      - WEBPASSWORD=changeme
      - DNS1=127.0.0.1#5335   # Use Unbound
      - DNS2=1.1.1.1
    volumes:
      - /home/maddox/pihole/etc-pihole:/etc/pihole
      - /home/maddox/pihole/etc-dnsmasq.d:/etc/dnsmasq.d
    depends_on:
      - unbound
    cap_add:
      - NET_ADMIN

  ###############################################################
  # NEXTCLOUD – Self-Hosted Cloud Storage
  ###############################################################
  nextcloud:
    image: lscr.io/linuxserver/nextcloud:latest
    container_name: nextcloud
    restart: unless-stopped
    ports:
      - "8080:443"
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Chicago
    volumes:
      - /home/maddox/nextcloud/config:/config
      - /home/maddox/nextcloud/data:/data

networks:
  default:
    driver: bridge

6. Deploying the Stack

I navigated to the directory containing the docker-compose.yml:

cd /home/maddox
docker compose up -d


up -d starts everything in the background

Docker automatically created, pulled, and launched all containers

Check running containers:

docker ps

7. Service Verification
Unbound

Runs a recursive DNS resolver on port 5335

Command to test:

dig @127.0.0.1 -p 5335 google.com

Pi-hole

Accessible via:
http://your-server-ip:8081/admin

Confirmed it automatically uses Unbound for DNS resolution

Logged in with the password defined in the environment file

Nextcloud

Accessible at:
https://your-server-ip:8080

Completed the first-time web setup

Verified uploads, users, and apps work correctly

8. Stopping/Managing the Stack
Stop all services:
docker compose down

Restart
docker compose restart

View logs:
docker compose logs -f

Update containers:
docker compose pull
docker compose up -d

9. Troubleshooting Notes

Port 53 conflicts:
If another DNS service is running, Pi-hole will fail to start.
Solution: disable systemd-resolved or move ports.

Nextcloud port conflict:
Changing to another port (e.g. 8443:443) resolves this.

Permissions issues on volumes:
Fixed with:

sudo chown -R $USER:$USER /home/maddox/*


Unbound not resolving:
Usually caused by a bad config file — restarted container and verified folder contents.

10. Files Included in My Submission

I have included the following:

docker-compose.yml

This documentation file (docker-dns-nextcloud.md)

GitHub Pages link (submitted separately)

Discussion board post of this same documentation

11. GitHub Pages Link

Insert URL here:
https://yourgithubusername.github.io

If you want, I can also:
