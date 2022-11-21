---
title: "Pihole with Podman on Fedora"
date: 2021-01-07T12:42:20+01:00
url: /pihole-podman-fedora/
categories:
  - Linux
  - Networking
tags:
  - Pihole
  - Fedora
  - SELinux
draft: true
---
<!--more-->


podman pull pihole/pihole

podman volume create pihole_pihole

podman volume create pihole_dnsmasq

sudo sysctl net.ipv4.ip_unprivileged_port_start=53


sudo nvim /etc/sysctl.conf

net.ipv4.ip_unprivileged_port_start=53

sudo nvim /etc/systemd/resolved.conf
DNSStubListener=no
DNS=192.168.0.1

sudo systemctl status systemd-resolve
loginctl enable-linger $USER

podman create \
--name=pihole \
--hostname=pi.tokariew.xyz \
--label "io.containers.autoupdate=image" \
--cap-add=net_admin \
--cap-add=cap_net_bind_service \
--dns=127.0.0.1 \
--dns=1.1.1.1 \
-e tz='europe/warsaw' \
-e serverip=127.0.0.1 \
-e webpassword=f6y4ytfh \
-e interface=eth0 \
-e dnsmasq_listening=all \
-e webuiboxedlayout=traditional \
-e dns1=1.1.1.1 \
-e dns2=1.0.0.1 \
-e dnssec=true \
-e rev_server=true \
-e rev_server_target=192.168.0.1 \
-e rev_server_cidr=192.168.0.0/24 \
-e temperatureunit=c \
-v pihole_pihole:/etc/pihole:Z \
-v pihole_dnsmasq:/etc/dnsmasq.d:Z \
-p 80:80/tcp \
-p 443:443/tcp \
-p 67:67/udp \
-p 192.168.0.110:53:53/udp \
pihole/pihole

podman generate systemd --new pihole

