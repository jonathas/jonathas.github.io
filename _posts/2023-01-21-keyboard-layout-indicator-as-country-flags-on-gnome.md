---
layout: post
title: "Bringing Back the Flags! Keyboard Layout Indicator as Country Flags on Gnome"
excerpt: "A script for converting the keyboard layout indicator on Gnome from text to country flags"
tags: [gnome, linux, keyboard layout, country flags, python, x11, x11 config, arch linux, aur, customization, open-source]
date: 2023-01-21T13:10:08+01:00
comments: true
image:
  feature: posts/keyboard-layout-indicator/cover-keyboard-layout.jpg
  credit: PngTree
  creditlink: https://pngtree.com/free-backgrounds
---

[Gnome](https://www.gnome.org/) is a free and open-source desktop environment for Linux and Unix-like operating systems. It is one of the most popular desktop environments for Linux, and is the default desktop environment for many Linux distributions. Gnome is known for its simplicity, ease of use and a user-friendly interface. It is designed to be easy to use for people of all ages and skill levels and it's also fully themeable and customizable.

One of the features that Gnome used to offer was the ability to display country flags as the keyboard layout indicator. This feature was present in earlier versions of Gnome, but was removed in later versions. The thing is, users don't always agree with the decisions a product takes, so in this case as the product is open-source and highly configurable, it's much easier to customize it to our liking.

A few months ago I implemented a Python script which updates the config files related to X11, where Gnome fetches the information for the keyboard layout indicator, and changes the country codes to their flags instead.

## Current state

On Gnome, the keyboard layout is presented as the code of the language which is currently selected. In the example below, the English (US) keyboard layout is selected:

![Original Keyboard Layout indicator](/images/posts/keyboard-layout-indicator/gnome-keyboard-original.jpg "Original Keyboard Layout indicator")

## After the script

A country flag there looks nicer to me instead of the language. After this script runs, this is what that would look like for the English (US) keyboard layout:

![Modified Keyboard Layout indicator](/images/posts/keyboard-layout-indicator/gnome-keyboard-modified.jpg "Modified Keyboard Layout indicator")

Ps: Other keyboard layouts/languages are also available

### The switcher now looks like this

![Keyboard Layout switcher](/images/posts/keyboard-layout-indicator/keyboard-layout-switcher.jpg "Keyboard Layout switcher")

### In context

![Keyboard Layout screenshot](/images/posts/keyboard-layout-indicator/keyboard-layout-screenshot.jpg "Keyboard Layout screenshot")

## Installing

If you use [Arch Linux](https://archlinux.org/) or any distribution based on it, you can get it from [AUR](https://wiki.archlinux.org/title/Arch_User_Repository) using [yay](https://github.com/Jguer/yay):

```bash
yay -S x11-keyboard-flags
```

On other distributions which have nothing to do with Arch Linux, you can [get the source code](https://github.com/jonathas/x11-keyboard-flags) and install it manually.

## Using

Run it with sudo:

```bash
sudo x11-keyboard-flags
```

Then reset gnome-shell by pressing Alt+F2 and entering the "r" command.

The script makes a backup of the original evdev.xml file from X11 before running and then sets the flags in the original evdev.xml file

And that's it! After these steps you'll see the flags in your keyboard layout selector.

## Conclusion

This customization can enhance the visual representation of the keyboard layout and make the language switching process more intuitive. As a further step, you can include this script in the startup process of your OS so that even if the package related to the file that was changed gets updated, the country flags will still be there.
