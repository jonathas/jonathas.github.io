---
layout: post
title: "How I replaced Dropbox Camera Uploads with an automated photo sync pipeline"
excerpt: How I rebuilt the best part of Dropbox Camera Uploads with Apple Photos, osxphotos, ADB, and rsync to keep my photos available as normal files.
date: 2026-03-08T17:39:19+01:00
comments: true
tags: [apple-photos, photo-backup, automation, adb, fotoo, osxphotos, rsync, digital-photo-frame, filesystem, macos]
---

I've always liked to have my photos and videos organized in proper directories instead of leaving everything in a messy state, with duplicates and many very similar photos and videos just occupying space. In order to keep my file list clean, I run a photo workflow process on a weekly basis (so they don't accumulate over time), where I move the files to the correct directories and back them up. I also want to have full control of my files, which means I want to have them in my hard-drive so I can export them to external drives instead of only keeping them in iCloud.

For a long time, my photo workflow was technically working, but it was wasting too much of my time.

For years, one important part of that workflow was handled by Dropbox Camera Uploads.

That feature solved a very practical problem really well:

> It gave me automatic, local-first access to my camera photos as normal files.

However, I've started thinking about maybe replacing Dropbox with another service, like for example Proton Drive. Unfortunately, Proton Drive doesn't have a Camera Uploads feature and the photos it syncs are not available as files in your hard-drive.

Once Dropbox Camera Uploads stopped being the center of my workflow, I had to rebuild that behavior myself.

---

## Before

Every week I had to do the same sequence again:

- Wait for Dropbox Camera Uploads to sync photos and videos from my iPhone to my MacBook
- Check for duplicates and clean them up
- Run the [pbc-organizer](https://github.com/jonathas/pbc-organizer) script I created to rename and move the files into directories separated by date, according to their exif info.
- Move files into the right folders for backup
- Choose a few good photos for my Digital frame
- Convert and resize these selected images since I don't need huge files in a Digital frame.
- Transfer them to the Digital frame/Android tablet with [LocalSend](https://localsend.org/)
- Run backups to external drives using custom rsync scripts

None of these tasks were individually difficult, but too many small manual steps meant too many chances to forget something, postpone it, or leave the whole process half-finished.

This became even more obvious after I [replaced my old Frameo digital frame with an Android tablet running Fotoo](https://jonathas.com/how-i-replaced-my-frameo-digital-frame-with-an-android-tablet/).

That change gave me a better display device and faster system, but it did not solve the larger problem:

> My photos were still trapped inside a mostly manual workflow.

Dropbox Camera Uploads had set a very high baseline for me.

It was not sophisticated, but it did one thing extremely well:

- Camera photos appeared as files
- The files were easy to inspect
- The files were easy to back up
- The filesystem stayed central

Once I no longer wanted to depend on Dropbox anymore for that, I discovered that replacing this behavior was not trivial.

This newer workflow had a few annoying properties:

- Apple Photos was already useful for weekly curation, but it did not give me the same filesystem-level result as Dropbox Camera Uploads
- The digital frame wanted plain JPEG files in a normal folder
- LocalSend still required a manual transfer step every time I wanted to update the tablet
- Backups wanted deterministic paths and filenames
- Other cloud storage services did not automatically replace the Dropbox Camera Uploads workflow. Proton Drive's photos feature doesn't sync the real files to your MacBook.
- My time was being spent on glue work between tools instead of on actual curation

The curation part is valuable, while glue work is not.

## What I wanted instead

I was already using Apple Photos for the part it is actually good at:

- Capturing and syncing from devices
- Browsing photos quickly
- Deleting duplicates
- Selecting favorites

My weekly routine was already based on curating photos there:

- Remove the photos I do not want to keep
- Leave only the selected ones in iCloud and the rest on my physical backups (Dropbox + Macbook hard-drive + external hard-drives)

The idea was:

> I am already paying for iCloud, and Apple Photos already handles the syncing part well enough, so I can leverage that and rebuild the filesystem part around it.

In other words, I wanted back the useful part of Dropbox Camera Uploads:

- Photos arriving on my MacBook through sync
- Photos available as normal files on disk

Without depending on Dropbox to do it.

This also makes the setup more resilient if I change cloud providers again.

So instead of looking for one big replacement product, I built a small pipeline around the tools that already do each part well.

The core pieces are:

- `osxphotos` to export files from Apple Photos
- `sips` to resize and convert images on macOS
- ADB to push photos to the Android tablet over Wi-Fi
- `rsync` to copy the archive to external drives

---

## The part I automated

The result is a small project I called [photo-sync-pipeline](https://github.com/jonathas/photo-sync-pipeline).

It automates the repetitive parts of my workflow while keeping the human decisions manual.

I still manually decide:

- Which duplicates to delete
- Which photos are worth keeping
- Which photos should go to the Digital frame

But once those decisions are made, the rest can be scripted.

The pipeline now does this for me:

- Export photos and videos from Apple Photos since a given date
- Place them into predictable directories on disk
- Export the curated "Digital frame" album
- Normalize file extensions
- Convert HEIC files to JPG
- Resize and compress images for the tablet
- Push them to the Android device over ADB via Wi-Fi
- Trigger a media scan
- Restart Fotoo
- Clear the Digital frame album after the transfer
- Run backups to external drives

## What the weekly flow looks like now

I still keep one lightweight weekly routine:

1. Open Photos and remove obvious duplicates.
2. Wait until iCloud sync is complete.
3. Export new photos since the last run.
4. Select the photos I want on the digital frame by adding them to the `Digital frame` album.
5. Connect the backup drives.
6. Run the export command for the digital frame.

In practice, the two commands I care about are:

```bash
export-photos-since 2026-01-25
export-digital-frame
```

The first command exports recent photos from Apple Photos into my archive folders.

Conceptually, that command is my replacement for Dropbox Camera Uploads.

It turns "photos inside Apple Photos" back into "photos as files on disk".

The second command handles the device-specific work:

- Export the album
- Resize and compress images
- Convert HEIC to JPG
- Send files to the tablet
- Refresh the media database
- Restart Fotoo
- Clear the Photos album
- Run the backup scripts

That removed most of the boring operational work from the process.

---

## Why this saves me time

The biggest improvement is not that the commands are fast.

The biggest improvement is that they reduce mental overhead.

Before, I had to remember the exact sequence and mentally verify each step:

- Did I export the right date range?
- Did I convert the HEIC files?
- Did I transfer the resized versions or the originals?
- Did I run backups afterward?

Now the process is encoded once in scripts instead of being re-created from memory every week.

This is the kind of automation I like most: Not flashy, just removing repeated friction from a process I already know I will keep doing.

---

## The digital frame became part of the pipeline

This also completed the setup I started when I moved away from Frameo.

In my earlier post about the Android tablet, I described the device change itself.

This pipeline is what made that setup truly practical long-term.

At first I was transferring the selected photos from my MacBook to the tablet with LocalSend, which was already much better than using Frameo. But once I wanted a repeatable weekly process, manual transfer was still too much friction.

So LocalSend was good as a transition step, while ADB over Wi-Fi turned it into a proper automation.

Instead of manually preparing a batch of photos and transferring them each time, the "Digital frame" album in Apple Photos now acts as a signal:

> These are the photos I want on the frame next.

Once the pipeline consumes that album, it exports the files, sends them to the tablet, refreshes Fotoo, and clears the album again.

So the album is not storage, but intent.

That small shift made the whole workflow much cleaner.

---

## Final thoughts

I did not build this because I wanted an elaborate photo management system, but because I was tired of doing the same boring photo chores over and over again.

The final result is intentionally simple:

- Apple Photos for capture and curation
- exported files as the archive
- ADB for the tablet
- rsync for backups

No single tool owns the whole process.

That is exactly why it works better for me.

If you also use Apple Photos but want your files, backups, and digital frame updates to behave like normal, inspectable Unix-style workflows, building a small pipeline around them is absolutely worth it.
