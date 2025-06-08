---
layout: post
title: "How I Replaced My Frameo Digital Frame with an Android Tablet"
excerpt: "My digital photo frame kept crashing and limiting uploads, so I replaced it with an Android tablet running Fotoo—free cloud backup, no slowdowns."
tags: [android-tablet, frameo-alternative, fotoo, digital-photo-frame, lenovo-yoga-tab-11, localsend, google-photos, frameo-issues, photo-backup, diy-tech]
date: 2025-06-08T10:10:08+02:00
comments: true
image:
  feature: posts/frameo-vs-fotoo/cover-frames.jpg
---

Having a digital photo frame at home has always been a nice way to keep our memories alive. Over the years, I've collected thousands of photos from family trips, special events, and everyday moments. This turned our digital frame into a slideshow of our best experiences. However, recently, I had issues with my Frameo-powered digital frame, leading me to look for a better solution. Here's how I moved to a Lenovo Yoga Tab 11 running Fotoo and why it’s been a much better choice.

### The Issues with Frameo

<img src="/images/posts/frameo-vs-fotoo/frameo.jpg" alt="Frameo" title="Frameo" style="margin-top: 1.5em;margin-bottom: 1.5em;" />

Initially, [Frameo](https://www.frameo.com/) seemed like a good idea—sending photos directly from an iPhone app to the frame. But in reality, it wasn't very good. First, the Frameo iOS app took up a lot of space, sometimes more than 4GB on my phone, which is too much for just sending photos.

Even worse, after uploading a few thousand photos, my digital frame became slow and unresponsive. Navigating photos caused it to crash frequently, needing regular restarts. The touch screen became slow, and the overall experience was frustrating. This made me realize the Frameo digital frame wasn't reliable at its basic job—showing photos smoothly.

### The Difficult Frameo Workflow

My workflow with Frameo was annoying:

* Selecting and sending photos in small groups (only 10 photos at a time unless paying extra).
* Manually resizing and positioning each photo on the frame to fit the display.
* Regularly inserting an SD card to manually back up the photos.

Not exactly easy!

### Moving to Fotoo on Lenovo Yoga Tab 11

![Lenovo Yoga Tab 11](/images/posts/frameo-vs-fotoo/tablet.jpg "Lenovo Yoga Tab 11")

Looking for something better, I decided to use a used Lenovo Yoga Tab 11 as my new digital frame. The tablet might be a bit overkill for this task, but it already has a built-in stand, perfect for displaying photos.

My new setup includes:

* Installing [Fotoo](https://play.google.com/store/apps/details?id=com.bo.fotoo), a simple digital frame app for Android.
* Using [LocalSend](https://localsend.org/) to easily transfer photos directly from my MacBook to the tablet.
* Automatic backup through Google Photos using the tablet's built-in syncing feature, so all photos are safely stored in Google Drive for free.

### Optimizing Images with a Custom Script

The Frameo app on iOS automatically reduced photo quality to save storage space. LocalSend doesn't have this feature, so I made a simple script to resize and convert my images before sending them:

```bash
#!/bin/bash

for f in *.heic; do
  sips -Z 2000 -s format jpeg -s formatOptions 75 "$f" --out "${f%.*}.jpg"
done
```

This script resizes images to save space without losing much quality, so after selecting the photos, I run the script to resize them and then can just transfer all at once to the tablet.

### Keeping the Same Screen Schedule

In Frameo, I had my frame set to turn off at 11 pm and turn on again at 7 am. Fotoo allows me to easily set up the tablet to do exactly the same, matching my previous settings perfectly.

### Free Cloud Backup Compared to Frameo

One great benefit of using an Android tablet is automatic free backup with Google Photos. In Frameo, cloud backup is only available with a paid subscription, but with my tablet setup, backups happen automatically for free using Google Drive.

### Conclusion

Switching from a Frameo digital frame to an Android tablet running Fotoo has greatly improved my experience. Using Fotoo, LocalSend, and Google Photos is an efficient and simple way to display our family's favorite moments. If you're frustrated with slow or limited digital frames, this setup could save you headaches and storage space!
