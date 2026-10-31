---
layout: post
title: "Bringing a Bricked Echo Link Amp Back From the Dead"
date: 2026-10-31 01:30:00 +0000
icon: ti-reload
accent: "#E8A33D"
author: Darragh Crawley-Doyle
tags: Hardware, Repair, Right to Repair, MediaTek, Android, E-Waste
---

**Someone handed me an Amazon Echo Link Amp that didn't work, was still tied to an account nobody could log into, and had a locked bootloader that refused every reset I threw at it. It's now sitting here working perfectly, and here's the long version, because it's a good one.**

I'll split this in two. First the thought process, because that's the part I actually care about and the part that generalises. Then the bit nobody reads unless they're mid-repair at 1am with the same symptoms: the actual steps, every command, what to install, and how I confirmed each thing instead of hoping.

## Part one: the thought process

### The moment of hope

The thing was dead on arrival. Powered on, the light flashed blue and white forever and never went orange. A 25-second Action-button reset did nothing. It wouldn't show up for setup in the Alexa app. By every normal measure it was a paperweight.

I opened the case anyway, mostly to see what was in it, and that's where the hope came from: an internal micro USB port on the board. That little port is a door. It means the thing can probably talk to a computer, and if it can talk, it can usually be diagnosed, and if it can be diagnosed, it can often be saved.

I should be honest about why I bother with something like this. I had no particular plan for the device. I just can't stand throwing away hardware that isn't actually broken. E-waste feels like a waste in the most literal sense: a working computer, with a perfectly good chip and radio and amplifier in it, going in a bin because one piece of software got itself into a knot. The hardware deserves better than that, and frankly I find a real amount of joy in reviving something everyone else has written off. This one scratched that itch completely.

### "Won't boot" is a symptom, not a diagnosis

The single most useful instinct here is to refuse the word "bricked." A device stuck in a boot loop is telling you something. It's getting far enough to try, and failing at a specific point. The job is to find that point.

So the question stopped being "is it broken" and became "where, exactly, does it give up?" That reframes everything. You're no longer staring at a dead box. You're looking for one failing step in a sequence.

### The rule that keeps a half-dead device from becoming a fully dead one

Here's the discipline I held to the whole way through, and it's the thing I'd tattoo on a beginner. Read-only until proven otherwise. Back up everything before you change a single byte. And when you finally do have to write something, prove the write is safe on something worthless first.

The failure mode of device repair is impatience. You get a tool connected and immediately start flashing firmware or erasing partitions, and that's the step that turns a recoverable device into a genuinely dead one. I didn't write anything to this device for the first several hours of work. I made a complete read-only copy of its storage and did all the actual detective work on that copy, on my PC, where no mistake I made could reach the hardware.

Paired with that: every check needs a way to fail. If you run a test and it can only ever say "yes," you haven't tested anything. More than once that caught me, including on the single most important step of the whole repair, which I'll come to.

### What the detective work actually found

Once I had a full backup, the picture came together fast, and it was the opposite of what "bricked" implies. The storage was completely healthy: a valid partition table, valid boot images, the bootloader and secure-world firmware all intact. And the kernel's own crash log was empty. Whatever was rebooting the thing, it wasn't a kernel panic. The Linux underneath was fine.

That's a big deal. A healthy kernel with an empty panic log means the failure is happening up in the Android userspace, after everything low-level has started cleanly. The device was booting almost all the way and then falling over near the finish line.

The proof was in the crash logs Android keeps on its data partition. Ten native crash dumps, and every single one was the same process, Amazon's Whole Home Audio Daemon, segfaulting in the same place, over and over, every few seconds. The daemon starts, tries to reconcile the device's setup state, dereferences something that isn't there, dies, gets restarted, and dies again. Enough of those in a row and the device reboots. That's the loop. Not a dead board. One daemon in a crash loop.

And the reason it was crashing was almost poetic. The device was in a half-deregistered state. Its account credentials had been properly revoked (whoever owned it had removed it), but a load of stale references to that old account were left behind: cached account IDs, the old owner's saved Wi-Fi networks, the lot. The daemon kept tripping over this inconsistent leftover state. The cure for that is the one thing I'd been unable to do: a factory reset, to wipe the data partition clean.

### The lock that wasn't really a lock

The bootloader was locked, and a locked bootloader refuses `fastboot erase userdata`. That's the wall every normal recovery path hits.

But "locked" here is a policy enforced by one component, not a physical impossibility. MediaTek chips have a boot ROM baked into the silicon that runs beneath the bootloader. It's how they're recovered at the factory, and it was reachable even with everything locked down. Through that path I could do exactly the erase that `fastboot` refused.

I want to be careful about what that does and doesn't mean. It didn't unlock anything, it didn't defeat secure boot, and it only worked on hardware I physically own. The lock still does its job against casual tampering. It just isn't the same thing as "the data can never be wiped," and conflating the two is what sends recoverable devices to landfill.

That's the whole thought process. Don't trust the word "bricked," back up before you touch anything, find the one failing step, and know the difference between "locked" and "impossible."

## Part two: the actual steps

Right, the part nobody cares about until they need it. This is specific to an Amazon Echo Link Amp (MediaTek MT8167, codename KYANITE) on Windows 11, but the shape applies to a lot of locked MediaTek devices.

> **Fair warning.** This involves erasing a partition on a device over a low-level interface. Do it on hardware you own, understand that a wrong partition name can ruin your day, and make the read-only backup first so you can always see what was there.

### What you need

Software and downloads:

* **Python 3** for Windows.
* **[mtkclient](https://github.com/bkerler/mtkclient)**, the open-source tool that talks to MediaTek boot ROMs. Clone it and install its requirements into a virtual environment:

```powershell
git clone https://github.com/bkerler/mtkclient
cd mtkclient
py -3 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

I keep mine at `%LOCALAPPDATA%\smarthome-tools\mtkclient`, which is what the commands below assume.

* **[UsbDk](https://github.com/daynix/UsbDk/releases)**, the Red Hat USB filter-driver runtime. On Windows this is the USB backend mtkclient uses to reach the boot ROM reliably. Install it, then tell mtkclient to use it with an environment variable (shown in each command below). Optional fallback: [Zadig](https://zadig.akeo.ie/), to bind a WinUSB driver to the port. I did not end up needing it, because the MediaTek serial (VCOM) driver Windows installs automatically was enough, but keep it in your back pocket.

Hardware:

* A **data-capable micro USB cable**. This bit me: my first cable was charge-only and the device simply didn't enumerate. Swap cables before you assume anything's wrong.
* The device opened enough to reach the internal micro USB port.

For the offline analysis (reading your backup without touching the device), I used three small read-only scripts I wrote along the way: one that parses the partition dumps, one that reads an ext4 image without mounting it, and one that carves the ramdisk out of a boot image. I wrote the ext4 reader because neither 7-Zip nor WSL would read the data-partition image cleanly on this machine, and I wanted something that only ever reads. They're all dependency-free Python and they're on GitHub as [mtk-dump-tools](https://github.com/WeetabixsJNR/mtk-dump-tools) if they're any use to you.

### Step 1: identify the USB modes

Plug in and watch what Windows enumerates in each state. For this device:

| State | How you get it | What appears (USB VID:PID) |
|---|---|---|
| Normal boot | Just power on | MTP device `0e8d:2008`, ADB off |
| Fastboot (green light) | Hold the rear button while powering on | `0bb4:0c01`, but the bootloader is locked |
| Boot ROM (BROM) | Button-held cold boot (below) | `0e8d:0003` for a few seconds, then preloader `0e8d:2000` |

Confirming it with a control: I didn't guess at the trigger. I watched Windows' own device-arrival timestamps across two deliberate cold boots. A plain cold boot produced only the preloader (`0e8d:2000`) for a couple of seconds and then booted normally. A button-held cold boot produced the BROM (`0e8d:0003`) first. That contrast is the negative control. The thing I thought triggered the BROM was the only difference between the run that showed it and the run that didn't.

### Step 2: get into the boot ROM

The BROM window is only a few seconds, but you don't have to race it. Start mtkclient first so it's already waiting, then trigger the device.

The "button-held cold boot": with the device fully powered off, hold the rear button down, and while holding it, apply power. Keep holding. That's what exposes the boot ROM.

Start with a harmless, read-only question to confirm you're actually talking to it:

```powershell
$env:MTK_USE_USBDK = "1"
& "$env:LOCALAPPDATA\smarthome-tools\mtkclient\.venv\Scripts\python.exe" `
  "$env:LOCALAPPDATA\smarthome-tools\mtkclient\mtk.py" printgpt
```

If it connects, it'll load a download agent into the device's RAM and print the partition table. On this device it also confirmed the chip and its security state: SoC MT8167 (hardware code `0x8167`), read straight from the chip rather than guessed from the codename, with secure boot on and download-agent auth on. Not encouraging on paper, but the MT8167 is old enough that mtkclient could still load a patched agent. Nothing above writes to the device. If it connects and prints the partition table, you're in.

### Step 3: back up everything, read-only

Before changing anything, copy the flash out. This is the step that makes everything after it safe, because you can always look at what was there.

```powershell
# Every partition except the big data one:
$env:MTK_USE_USBDK = "1"
& "$env:LOCALAPPDATA\smarthome-tools\mtkclient\.venv\Scripts\python.exe" `
  "$env:LOCALAPPDATA\smarthome-tools\mtkclient\mtk.py" `
  rl "$env:LOCALAPPDATA\smarthome-tools\dumps\partitions" --skip userdata

# Then the data partition on its own:
& "$env:LOCALAPPDATA\smarthome-tools\mtkclient\.venv\Scripts\python.exe" `
  "$env:LOCALAPPDATA\smarthome-tools\mtkclient\mtk.py" `
  r userdata "$env:LOCALAPPDATA\smarthome-tools\dumps\partitions\userdata.bin"
```

(`rl` reads a list of partitions to a folder; `r <name> <file>` reads one partition. Each run needs its own button-held cold boot, because the boot ROM drops off after it's done.)

### Step 4: diagnose from the copy, not the device

All of this happens on the PC against the backup. Nothing here can hurt the hardware.

Parsing the partition table, boot images, bootloader-control block and kernel crash log is what told me the storage was healthy. A valid partition table, two valid Android boot images, a bootloader-control block holding no stuck recovery command, and a MediaTek exception log that was completely empty. No kernel panic. That empty-versus-full distinction matters, and it's a real reading rather than a dead tool, because the same scan found plenty of data in other partitions.

The data partition's filesystem was plain, unencrypted ext4 (the fstab mounted it `encryptable`, not `forceencrypt`), so it was readable directly. That's where it cracked open: ten crash dumps, every one the same Amazon audio daemon, same segfault, timestamps from that very day. A tight crash loop. Reading the device's account database showed zero valid tokens but stale cached references to the previous owner's account and Wi-Fi, which is the half-deregistered state the daemon kept choking on.

So: healthy hardware, one daemon crash-looping on leftover account state. The fix is a factory reset, which means wiping the data partition.

### Step 5: prove you can write, on something worthless

The boot ROM could read. Whether it could write or erase under secure boot was the open question, and I was not going to find out by erasing the data partition.

So I tested it on `swdl`, Amazon's software-download staging partition, which was already empty and holds no boot code. Formatting it couldn't hurt anything:

```powershell
$env:MTK_USE_USBDK = "1"
& "$env:LOCALAPPDATA\smarthome-tools\mtkclient\.venv\Scripts\python.exe" `
  "$env:LOCALAPPDATA\smarthome-tools\mtkclient\mtk.py" e swdl
```

```
DAXFlash - Successfully formatted addr 0x4980000 with length 33554432.
Formatted sector 150528 with sector count 65536.
```

Those sector numbers matched `swdl`'s exact position in the partition table, so I knew it had formatted the right thing. The boot ROM will erase under secure boot. Capability proven, zero risk taken.

> A control I nearly got wrong: my first check was to read `swdl` back and compare it to the original. But `swdl` was already all-zeros, so "zeros before, zeros after" would look identical whether the format worked or was silently refused. That test couldn't fail, so it proved nothing. The console output above is what actually settled it. If your check can't tell success from failure, it isn't a check.

### Step 6: the actual fix

Same command, real target. The data partition holds no boot code, so even in the worst case this can't brick the device, and there's a full backup of it.

```powershell
$env:MTK_USE_USBDK = "1"
& "$env:LOCALAPPDATA\smarthome-tools\mtkclient\.venv\Scripts\python.exe" `
  "$env:LOCALAPPDATA\smarthome-tools\mtkclient\mtk.py" e userdata
```

```
DAXFlash - Successfully formatted addr 0x88980000 with length 1667743232.
Formatted sector 4475904 with sector count 3257311.
```

Sector range matched `userdata` exactly. Nothing outside it touched.

### Step 7: boot it

Normal power-on, no button held. Give it a few minutes the first time, because it has to rebuild the data partition from empty before it does anything visible. Don't pull power during that.

The light went to the orange setup state. The loop was gone. Two writes total: one to a throwaway partition to prove it was safe, one to do the job.

## The ending: happy days

Then the satisfying part. I registered it to an account, and it just came back. It picked up exactly where it left off, pulled down its old name, remembered what it was, and worked immediately. No anti-theft gate, no drama, no customer-service phone call. One moment it was a boot-looping paperweight that every official tool had given up on, the next it was a working speaker amplifier behaving as if nothing had ever happened.

That's the whole reason I do this. A device that everyone had written off as e-waste is back in service, and it took understanding rather than brute force to get there. None of the clever bits were really that clever. It was mostly just refusing to skip steps: back up before touching, diagnose from the copy, and prove the dangerous thing is safe before you do it for real. That's the discipline that turns "it's bricked, bin it" into "happy days."

## Tools and scripts

* [mtkclient](https://github.com/bkerler/mtkclient): MediaTek boot ROM and download-agent access.
* [UsbDk](https://github.com/daynix/UsbDk/releases): the USB backend it uses on Windows.
* [mtk-dump-tools](https://github.com/WeetabixsJNR/mtk-dump-tools): the three small read-only scripts I used for the offline analysis (an ext4 reader, a dump-forensics parser, and a boot-image ramdisk carver), dependency-free Python.
