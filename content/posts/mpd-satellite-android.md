---
title: "MPD Satellite Android"
date: 2022-05-20T19:17:03+02:00
url: /mpd-satellite-android/
tags:
  - Linux
  - Networking
  - Android
draft: false
---

I will show how, I setup my NAS for streaming music to my Android phone, and
another computers on my local network. Why such setup? It allows me to stream
music without transcoding to my devices.

<!--more-->

## NAS Side

My NAS is running Fedora Server Edition, so all instruction will be provided for this distro.

So let start with installing the MPD[^1] which will
serve as my music database, and NFS server which will allow me to stream music.

[^1]: https://www.musicpd.org/


```bash
sudo dnf install mpd nfs-utils
```

Next, I created base config for my MPD with:

``` BashSession
mkdir -p ~/.config/mpd
nvim ~/.config/mpd/mpd.conf
```

I tried to keep my config simple. With this config, we will never use it to
play music, only have shared database between computers and Android.

``` INI
music_directory "/media/btrfs/Media/Music"
playlist_directory "~/.local/share/mpd/playlists"
db_file "~/.local/share/mpd/mpd.db"
state_file "~/.local/share/mpd/mpdstate"
auto_update "yes"
max_output_buffer_size "32768"

input {
        plugin "curl"
}
```

For another devices to be able to connect with our MPD server, we will need to add firewall rule.

```BashSession
sudo firewall-cmd --permanent --add-service mpd
```

Next, I created folders where MPD database and files will be stored, and run it, as systemd service.

```bash
mkdir -p ~/.local/share/mpd/playlists
systemctl --user start --now mpd.service
```

We are nearly done with our setup on the NAS, now we need to make Music
directory accessible from other devices. For it, I used the NFS share. I edited
the `/etc/exports` file.

```bash
sudoedit /etc/exports
```

In the file, I added single a share with read only permissions.

```
/media/btrfs/Media/Music 192.168.0.0/24(ro,insecure)
```

Next, export NFS shares, and run it:

```bash
sudo exportfs -rav
sudo firewall-cmd --permanent --add-service={nfs3,mountd,rpc-bind}
sudo systemctl enable --now rpcbind nfs-server
```

## Android Side

For Android, we need two pieces of software which can be downloaded using
F-Droid[^2] or Google Play Store. First is just MPD version
for Android available here[^3] and
second is M.A.L.P[^4].

[^2]: https://f-droid.org/
[^3]: https://f-droid.org/en/packages/org.musicpd/
[^4]:  https://gitlab.com/gateship-one/malp/



So let's start with configuring MPD on Android.

We need to create `mpd.conf` file in the base of Android folder. My NAS have
static IP, which is `192.168.0.113`, so if you follow these guide, you will
need to change it to IP of server with MPD.

```
music_directory "nfs://192.168.0.113/media/btrfs/Media/Music"
restore_paused "no"
max_output_buffer_size "32768"
replaygain "auto"

database {
    plugin "proxy"
    host "192.168.0.113"
}
audio_output {
    type "sles"
    name "Base Output"
}
```

I will advice to check following checkbox in MPD app on Android `Prevent
suspend when MPD is running (Wakelock)`. After that, click on button with `MPD
is not running`, button should change it text to `MPD is running` and the
notification will pop up.

So now we will configure `M.A.L.P` to connect with. Basic configuration is
quite simple, just adding name to the profile, and providing IP of server, in
this case `127.0.0.1`.

![add-profile](img/malp-profile.webp "Click on plus to create new profile")
![add-profile](img/malp-edit-profile.webp "Simple profile for local running version")
![add-profile](img/malp-folder.webp "We can browse music base on folder structure")

Few quirks with this setup:
 * Can't access files from outside local network -- can be bypassed by
   running VPN on phone, to connect to home network, I run wireguard as such,
   and will explain my setup in next tutorial.
 * In case that music collection is in lossless format, we will stream a lot of
   data to phone. So you can quickly run out of your data plan. In such case,
   you can run transcoding server[^5] on NAS, and
   slightly modify `music_directory` variable on `mpd.conf` on Android.

[^5]: https://github.com/jorams/transcoding-music-server

## Other computers on network

To access music on another computers, we will run MPD on them too, with following config:

```
music_directory "nfs://192.168.0.113/media/btrfs/Media/Music"
playlist_directory "~/.local/share/mpd/playlist"
log_file "~/.local/share/mpd/mpd.log"
pid_file "~/.local/share/mpd/pid"
state_file "~/.local/share/mpd/mpdstate"
restore_paused "no"
max_output_buffer_size "32768"
replaygain "auto"

database {
    plugin "proxy"
    host "192.168.0.113"
}

audio_output {
    type "pipewire"
    name "Pipewire"
}
```

And then start mpd.service

```bash
systemctl --user start --now mpd.service
```

As for client to control music playback you can use something like mpc[^6] ncmpcpp[^7] or cantata[^8]
mpc is great if you try to script, or remap your media keys to control MPD playback.

[^6]: https://musicpd.org/doc/mpc/html/
[^7]: https://rybczak.net/ncmpcpp/
[^8]: https://github.com/cdrummond/cantata
