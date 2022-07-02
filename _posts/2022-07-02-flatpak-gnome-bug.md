---
layout: post
title: Flatpak Apps in Gnome 
subtitle: Install cross-distro apps in any Linux Distro
#cover-img: /assets/img/hyper.png
thumbnail-img: /assets/img/flatpak.jpg
share-img: /assets/img/flatpak.jpg
permalink: /blog/flatpak-bug/
tags: [ubuntu]
---

## Installation

Whether We might be an Ubuntu/CentOS/PopOS user, `Flatpak` is a rescuer for us. It is a packaging system, where we can install flatpak apps in our own distro and start using it.

Flatpak apps runs in a sandbox environment which separates from host system's runtime. It takes more space than Snaps or AppImages but runs faster than Snaps. Flatpak is installed by default on modern Linux distros. If that's not the case, we can install it using the following commands:

On Ubuntu/Debian:
```bash
$ sudo apt install flatpak
```

On Arch Linux:
```bash
$ sudo pacman -S flatpak
```

On Fedora, RHEL, and CentOS:
```bash
$ sudo dnf install flatpak
```

We can add the Flathub repo to our system using the below snippet:

```bash
$ flatpak remote-add --if-not-exists flathub \ https://flathub.org/repo/flathub.flatpakrepo
```
### Apps Installation

We can use the flatpak install command to install packages. The below command installs the VLC flatpak from Flathub:
```bash
$ flatpak install flathub org.videolan.VLC
```

`VLC` is the app name, `org.videolan` is the company which creates VLC. Wait how do I do know these details everytime to install other apps in flatpak?

I understand your pain.

<iframe src="https://giphy.com/embed/edYZN0n2CMQvRycFSV" width="480" height="480" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/confused-idk-ananya-birla-edYZN0n2CMQvRycFSV"></a></p>

To know these details:

- Visit [flatpak](https://www.flathub.org) website
- Select app which we want to install, say `Spotify`
- Use `search apps` box to search the app
- At the bottom of the app page, command line instructions are present to install the app. `flatpak install flathub com.spotify.Client`
- Run it on the terminal and we can access the app


## BUG Alert

Though flatpak is faster in loading apps, it also has some bugs. One of the bug below:

```bash
$ sudo apt-get update
[sudo] password for edward: 
Hit:1 https://download.docker.com/linux/ubuntu jammy InRelease
Hit:2 http://archive.ubuntu.com/ubuntu jammy InRelease                                                          
Get:3 http://archive.ubuntu.com/ubuntu jammy-updates InRelease [109 kB]                                         
Hit:4 https://ppa.launchpadcontent.net/cappelikan/ppa/ubuntu jammy InRelease
Get:5 http://archive.ubuntu.com/ubuntu jammy-backports InRelease [99.8 kB]                  
Hit:6 https://ppa.launchpadcontent.net/graphics-drivers/ppa/ubuntu jammy InRelease
Get:7 http://archive.ubuntu.com/ubuntu jammy-security InRelease [110 kB]
Get:8 http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 DEP-11 Metadata [91.1 kB]
Get:9 http://archive.ubuntu.com/ubuntu jammy-updates/universe amd64 DEP-11 Metadata [94.5 kB]
Get:10 http://archive.ubuntu.com/ubuntu jammy-backports/universe amd64 DEP-11 Metadata [12.5 kB]
Get:11 http://archive.ubuntu.com/ubuntu jammy-security/main amd64 DEP-11 Metadata [11.4 kB]
Fetched 529 kB in 4s (126 kB/s)                

** (appstreamcli:223895): WARNING **: 12:23:36.072: Found icon of unknown type 'unknown' in 'system/flatpak/flatpak/cc.nift.nsm/*', skipping it.

** (appstreamcli:223895): WARNING **: 12:23:36.072: Found icon of unknown type 'unknown' in 'system/flatpak/flatpak/cc.nift.nsm/*', skipping it.
Reading package lists... Done
```


<iframe src="https://giphy.com/embed/cZxpHI9dH4eqFtwRKv" width="480" height="480" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/originals-cZxpHI9dH4eqFtwRKv"></a></p>

The temporary way of solving this is by running below command

```bash
$ sudo flatpak repair
[93/93] Verifying flathub:runtime/org.kde.Platform.Locale/x86_64/5.15-21.08…
Checking remotes...
Pruning objects
Erasing .removed
```

If we run again `sudo apt-get update`, we won't get same error again for some time.

## Updating Apps

We can update flatpak apps just like our Snap apps. Just run,

```bash
$ sudo flatpak update
```