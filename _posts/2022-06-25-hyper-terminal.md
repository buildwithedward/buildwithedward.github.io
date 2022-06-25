---
layout: post
title: Hyper - Give life to your default terminal
subtitle: Terminal for those who love UI
#cover-img: /assets/img/hyper.png
thumbnail-img: /assets/img/hyper.png
share-img: /assets/img/hyper.png
permalink: /blog/hyper-terminal/
tags: [misc,ubuntu]
---

{: .box-note}
**Note:** This article covers the installation in Ubuntu 22.04. For other OS, kindly check the [Website](https://hyper.is/)

# Table of contents

- [Background](#background)
- [Problem](#problem)
- [Solution](#solution)
  - [Step 1: Download Hyper terminal File](#step1)
  - [Step 2: Update the system after the installation](#step2)
  - [Step 3: Install Hyper Terminal](#step3)
  - [Step 4: Running the Hyper Terminal on Ubuntu 22.04](#step4)
  - [Step 5: Customizing the Hyper Terminal](#step5)
  - [Step 6: Making hyper as default terminal](#step6)
- [Limitations](#limitations)


## Background {#background}

I am using Ubuntu for the past 4+ years after a bad experience with Windows (*Of Course, in a different blog*).

<iframe src="https://giphy.com/embed/l1yjLsiHMQdWQmN2xY" width="480" height="245" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/wolfentertainment-svu-law-and-order-svu21-l1yjLsiHMQdWQmN2xY"></a></p>


I have been using [Gnome Terminal](https://wiki.gnome.org/Apps/Terminal) for years but I always look around for other [terminal emulators](https://en.wikipedia.org/wiki/Terminal_emulator) for the love of UI. It doesn't mean I hate default terminal in Ubuntu.

## Problem {#problem}

The issue with Gnome terminal is the UI. Yes I can understand, what you think. We can improve UI by customization. But the font sharpness in default terminal seems bad.

## Solution {#solution}

Finally, I find the impressive terminal emulator - [hyper.is](https://hyper.is/) which is an Electron based terminal built on `HTML/CSS/JS`. It is available for all OS platforms.

## Step 1: Download Hyper terminal File {#step1}

First, we need to download the Hyper terminal `deb` file through the below mentioned command. For OS installations, check the [website](https://hyper.is/)

{: .box-note}
**Note:** We can run hyper terminal through `.AppImage` in Linux distros if you want to try it out without installing.

```bash
$ wget -O hyper_3.2.3_amd64.deb https://releases.hyper.is/download/deb
```

## Step 2: Update the system after the installation {#step2}

It is the best practice to update the system before installing any package.

```bash
$ sudo apt update
```

## Step 3: Install Hyper Terminal {#step3}

Before Installing, move into the directory where `hyper` is downloaded. We can install our `.deb` in either of ways below.

```bash
# One-way
$ sudo apt install ./hyper_3.2.3_amd64.deb
```

```bash
# Another-way
$ sudo dpkg -i hyper_3.2.3_amd64.deb
```

## Step 4: Running the Hyper Terminal on Ubuntu 22.04 {#step4}

Once the Hyper terminal installation is completed, we can then search the application on the Ubuntu applications search bar or we can use the `hyper` command in the terminal to run the application on the desktop.

## Step 5: Customizing the Hyper Terminal {#step5}

We can customize the hyper terminal by the following command

```bash
$ sudo nano ~/.hyper.js
```

![](https://raw.githubusercontent.com/edwardcodes/edwardcodes.github.io/main/assets/img/hyper-settings.png)

## Step 6: Making hyper as default terminal {#step6}

We have set our hyper terminal, but if we try to open terminal using hotkeys `CTRL + ALT + T`, it will open default gnome terminal instead of hyper. Because, our linux distro does not consider hyper as an alternative terminal. We can change this by below commands.

Find where your `hyper` is located
```bash
(base) ➜  ~ which hyper 
/usr/local/bin/hyper
(base) ➜  ~ 
```

Set your hyper terminal with `priority` to identify the terminal later with choice. Here, I set the priority for hyper as `1`

```bash
(base) ➜  ~ update-alternatives --install /usr/bin/x-terminal-emulator x-terminal-emulator /usr/local/bin/hyper 1
(base) ➜  ~ update-alternatives --set x-terminal-emulator /usr/local/bin/hyper
```

Select the choice with the below command to make hyper as default terminal.
```bash
(base) ➜  ~ sudo update-alternatives --config x-terminal-emulator
There are 2 choices for the alternative x-terminal-emulator (providing /usr/bin/x-terminal-emulator).

  Selection    Path                             Priority   Status
------------------------------------------------------------
  0            /usr/bin/gnome-terminal.wrapper   40        auto mode
  1            /usr/bin/gnome-terminal.wrapper   40        manual mode
* 2            /usr/local/bin/hyper              1         manual mode

Press <enter> to keep the current choice[*], or type selection number: 2
(base) ➜  ~ 
```

If we want to make your gnome terminal as default again, repeat the last step and select either `0` or `1`

## Limitations {#limitations}

- **Slow.** While preparing for this blog, I came to know about a [blog](https://medium.com/@brianhague/why-i-switched-my-terminal-to-hyper-then-switched-back-f0bd06af4d7d#:~:text=It%E2%80%99s%20slow.,interrupt%20my%20workflow.) where the author describes his issues with hyper being slow, when he used multiple hyper windows. I never faced this. Might be in future.

- **Issue with Nautilus.** Nautilus is the file manager for Ubuntu. Even after making hyper as default terminal, if we try to open the terminal via `right click` mouse via `desktop`, either it redirects to gnome terminal or terminal won't open. 