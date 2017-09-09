---
layout: post
title: "Linux and me: Why I fell in love with Arch Linux"
excerpt: "How I started using Linux and the things I love about Arch Linux"
tags: [linux, archlinux, arch linux, arch, debian, windows, kiss, pacman, abs, aur, rolling release, slackware, fedora, manjaro, apricityos, chakra, ubuntu]
date: 2015-10-29T23:07:08-01:00
comments: true
image:
  feature: posts/cover-archlinux.jpg
---

In this post I'd like to talk about one of the things I love the most in tech, which is my favorite Linux distribution, called Arch Linux and why I chose it as my main OS.

About Arch Linux
----------------

Arch was created in 2002 by Judd Vinet in Canada and is now led by Aaron Griffin and other developers. It was inspired by CRUX and aims to be a simple and lightweight distribution with optimized packages for the i686 and x86-64 architectures.

According to the Arch wiki:
The design approach of the development team follows the KISS principle ("keep it simple, stupid") as the general guideline, and focuses on elegance, code correctness, minimalism and simplicity, and expects the user to be willing to make some effort to understand the system's operation. A package manager written specifically for Arch Linux, pacman, is used to install, remove and update software packages.

Arch Linux uses a rolling release model, such that a regular system update is all that is needed to obtain the latest Arch software; the installation images released by the Arch team are simply up-to-date snapshots of the main system components


A little bit of my history with Linux
-------------------------------------

My history with Linux started a little bit before December 2006 with a Brazilian distribution based on Debian that no longer exists, called Kurumin. I just knew that I had to learn something different from Windows, because come on, I was about to start my Computer Science course, so I decided I had to know about technology in a deeper level. It should be more than just clicking and dragging a mouse on Windows. 

Everybody was using Windows, but this idea of Linux really attracted me. It was an easy migration for me, as KDE doesn't and didn't use to look so different from Windows at that time either. After a while I started reading more about Linux distributions and got interested in Debian, as Kurumin was based on Debian. I thought: Why not go for the source, the father (or mother?), instead of using something based on it? 

I found Debian harder to install and configure for the beginner that I was at the time, but kept digging deeper and asking questions on forums, slowly learning more about it. The more I learned, the more motivated I'd get to know more about it, because I was not only learning how Linux works, but about Unix and operational systems in general. When I realised I could assemble my own OS with the packages I wanted, curiosity spoke louder and I was already completely involved in it. 

It was amazing to discover I could choose among many desktop environments instead of being trapped in just one of them, like I was when using Windows. It was very insteresting to discover too that they were made of packages that would run on the X server, so even if the graphic part would crash, I could just kill it and run it again, unlike MS Windows that doesn't have that separation between graphic and text mode and I'd have to Ctrl+alt+del it and restart the whole system. Another thing I found really great was the apt package manager. I didn't need to go to many websites and download the softwares I wanted, because I could just do an 'apt-get install nameofsoftware' and it would download and install it for me. No more next next finish, no more clicking and that was so practical. In order to update the whole system, I just needed to use the package manager as well, so that was great.

My curiosity led me to try Slackware for some time, but the lack of a package manager with repositories and automatic dependency solving made me lose so much time that I didn't stick with it for so long, even though it was interesting to learn about a new distribution and how it worked differently from Debian. I felt it was just not practical, though I could configure it just as I wanted. I learned about the KISS philosophy because of Slackware and started to adopt it.

I went back to Debian and used it as my main distribution until 2008, when I decided to try Arch Linux after reading a lot of good things about it. It was just love at first sight, after I discovered how customizable it is and that I could do anything I'd like to with it in a very simple way. Since 2008, I've tried many other distributions, most notably Fedora for some months, but wasn't able to stay away from Arch Linux for too long. I had already been spoiled by Arch and its simplicity, its KISS philosophy. Fedora just had so many wizards for everything and I understood that those are layers of unnecessary complexity when you can just change some values in a text file to make things work (Arch way) instead. Here are some of the things I love about Arch Linux and that made me choose it and until today not even think about replacing it for any other distribution:

Pacman
------

This is the name of Arch's package manager. Since I started using Linux, I've used apt (Kurumin, Debian, Ubuntu), yum (Fedora) and others, but by far pacman is the best package manager for me. It's fast, you need just one command to update the repositories and upgrade de system (pacman -Syu), instead of two (apt-get update && apt-get upgrade), it installs packages that are in the hard drive as well instead of in repositories, no need for a different command for that (like dpkg), and have I mentioned that it's fast? It also handles dependencies very well and the repositories have a plethora of packages. I broke many things with apt and dpkg, but never with pacman.

[Read more about it on Arch Wiki](https://wiki.archlinux.org/index.php/Pacman)

ABS (Arch Build System)
-----------------------

ABS is a ports-like system. Ports is a system used by *BSD to automate the process of building software from source code. So for example, if you don't wanna use the Firefox binary package that is easily available on Arch's repositories via pacman because you wanna change something on it to enable or disable a feature, you have access through ABS to the PKGBUILD file with the instructions used for compiling the Firefox package, so you can change them, compile it and install your own Firefox package instead, optimized the way you want. I've configured the compiler once to work in an optimized way for my specific CPU, then compiled all the LXDE packages from ABS and installed them. I noticed they were running a little bit faster after that.


AUR (Arch User Repository)
--------------------------

Arch Linux packages are infinite, because if you don't have a package in the official repositores to install via pacman, you may almost certainly find it in the AUR, which stands for Arch User Repository. Even if you can't find it there because nobody added it yet, you can do it yourself and become the maintainer for that package. You just need to check other packages and how their PKGBUILD file with instructions was created, then create one for the package you want and upload it in the AUR website. Recently AUR started using git for its packages, so it's even better now. You just need to push the changes.
The packages are usually compiled from source code, so it works similar to ABS in that sense, but in the AUR there are also some proprietary software that don't have the source code available. In that case, the binary will be downloaded and a package will be created with it.

According to the wiki: A good number of new packages that enter the official repositories start in the AUR. In the AUR, users are able to contribute their own package builds (PKGBUILD and related files). The AUR community has the ability to vote for or against packages in the AUR. If a package becomes popular enough — provided it has a compatible license and good packaging technique — it may be entered into the community repository (directly accessible by pacman or abs). 

[Read more about it on Arch Wiki](https://wiki.archlinux.org/index.php/Arch_User_Repository)

Rolling release
---------------

The rolling release model is certainly one of my favorite things about Arch Linux. Instead of having to do an apt-get dist-upgrade or something similar every 6 months or so (and risk breaking everything), in order to get the most recent version of the distribution, Arch just doesn't release any version. That means that after I update the entire system with one single 'pacman -Syu' command, I have the most recent version of Arch. Why not have it this way? It just works and during all these years my system was never left broken after an update, even though the packages are usually the latest available (bleeding edge).

[Read more about Arch compared to other distributions](https://wiki.archlinux.org/index.php/Arch_compared_to_other_distributions)


The Arch Wiki
-------------

That's the most complete documentation I've ever seen about a distribution. The wiki is so complete that I never had to ask anything in the forums. Whenever the developers decide to change something about how Arch will work, all the steps the user needs to take will be written in the wiki and even the problems that might happen in the process. It's really complete.


Distros that are based on Arch Linux
------------------------------------

Due to the success of the distribution, many based ones started to appear. They usually come with an installer that does everything for the user and also some software by default. That's useful for when you just wanna install it as fast as possible and already start using it having to configure the least possible. At work for example, I decided to give Manjaro a try instead of using Ubuntu. Besides Manjaro, other ones worth trying are Chackra (heavily geared towards KDE) and more recently Apricity OS, which I still haven't tried on the date I'm writing this post, but it looks interesting. 

Read more about Arch based distributions:

- [Top 5 Arch Linux Derivatives](https://www.maketecheasier.com/best-arch-linux-derivatives)

- [Arch based distributions (Arch wiki)](https://wiki.archlinux.org/index.php/Arch_based_distributions_%28active%29)

