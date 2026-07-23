---
author: Bronwen Aker
updated: 2026-07-22
presentation_type: Workshop
venue: Antisyphon AI Summit
---

```table-of-contents
title: # Table of Contents
minLevel: 0
maxLevel: 3
```

# Meet HeartOfGold and Marvin

This course uses two virtual machines:
* **HeartOfGold**, which acts as the server, hosting the local [[10 Glossary#Large Language Model (LLM)|LLM]] that you will install and configure.
* **Marvin**, which you will use to access it.

The connection between them is established and maintained using [[10 Glossary#Tailscale|Tailscale]], an encrypted mesh networking tool that lets two machines reach each other securely without manual VPN configuration or exposing services to the open internet. For penetration testers and red teamers, this is the same technique used to maintain persistent, encrypted access to internal services during an engagement. Here, we're using it to reach a private LLM instead.

## Setting Up VMware

This lab is set up to use VMware Workstation Pro or VMware Fusion Pro. The two VM images are provided as a zip archive ahead of the workshop, and registered students receive that download link via email.

**[Download the workshop files from Dropbox](https://www.dropbox.com/scl/fo/ws0q6m0ex4zs7aij9158c/APeV-FmjEznsiUrv2jkdek8?rlkey=a96mi9cgq4hj1686h6fpzsyt9&st=pqt2x9c4&dl=0)**

VMware itself you download from Broadcom. Both Workstation Pro and Fusion Pro are now free for personal, educational, and commercial use, with no license key required, but the download sits behind a free Broadcom Support Portal account:

[support.broadcom.com/group/ecx/productdownloads](https://support.broadcom.com/group/ecx/productdownloads?subfamily=VMware+Workstation+Pro&freeDownloads=true)

None of the software included in the VMs requires a license for personal use. Some applications do have requirements for commercial or enterprise use. You should be able to make use of the VMs and the software on them for a long time after this course is over.

> [!todo] Reminder to self: the Dropbox download link is in place above. Still open: whether hosted VMs become the primary delivery path, which would make most of this section moot.

### System Requirements

This workshop requires HeartOfGold and Marvin running at the same time, so host requirements are the sum of both VMs plus overhead for your host OS and VMware. Both VMs are set to 8 GB RAM each, which puts the VM total at 16 GB and the practical host minimum at 24 GB.

- **Marvin (client, Ubuntu desktop):** 8 GB RAM, 2 processors, 20 GB disk.
- **HeartOfGold (server, headless):** 8 GB RAM, 2 processors, 30 GB disk.

---
#### VMware Workstation Pro (Windows)

| Component    | Minimum                                      | Recommended                                |
| ------------ | -------------------------------------------- | ------------------------------------------ |
| **CPU**      | 64-bit, 4 cores (2 for Marvin + 2 for HeartOfGold), virtualization support (Intel VT-x / AMD-V) enabled in BIOS/UEFI | Quad-core+ 64-bit (Intel i5/i7 or Ryzen 5+) |
| **RAM**      | 24 GB (8 GB Marvin + 8 GB HeartOfGold + host overhead) | 32 GB or more |
| **Disk**     | 20 GB (Marvin) + 30 GB (HeartOfGold)           | SSD, 60 GB or more free |
| **Host OS**  | Windows 10/11 64-bit                         | Windows 10/11 64-bit                       |
| **Display**  | 1280×800                                     | 1920×1080 or higher                        |
| **GPU (3D)** | DirectX 11 / OpenGL 4.3 capable              | DirectX 11 capable GPU                     |

---
#### VMware Fusion Pro (macOS)

| Component    | Minimum                            | Recommended                                          |
| ------------ | ---------------------------------- | ---------------------------------------------------- |
| **CPU**      | Quad-core Intel (64-bit) or Apple Silicon, virtualization support enabled | Quad-core Intel i5/i7/i9 or Apple Silicon (M1/M2/M3) |
| **RAM**      | 24 GB (8 GB Marvin + 8 GB HeartOfGold + host overhead) | 32 GB or more |
| **Disk**     | 20 GB (Marvin) + 30 GB (HeartOfGold) | SSD, 60 GB or more free        |
| **Host OS**  | macOS 12 (Monterey) or later       | Latest macOS (Ventura / Sonoma)                      |
| **Display**  | 1280×800                           | 1920×1080 or higher                                  |
| **GPU (3D)** | Metal-capable GPU (for Intel Macs) | Metal-capable GPU / Apple Silicon GPU                |

---

> [!warning] If you have Apple Silicon (M1/M2/M3):
> - Only **ARM-based guest OSes** are supported (ARM Linux, Windows 11 ARM).
> - Intel x86 guest OSes (e.g., older Windows/Linux ISOs) will not run under Fusion on Apple Silicon.

> [!info] About RAM...
> Theoretically, RAM stands for "random access memory", but in reality it stands for "rarely adequate memory"! 😉 In short, if you have the system resources to increase the amount of RAM allocated to your VM, DO IT!

---

### Installing VMware

Download the installer from Broadcom using the link above. Sorry, no support for VirtualBox at this time.

- Sign in to the Broadcom Support Portal, creating a free account if you do not have one.
- Choose the installer appropriate to the operating system of the host you will run the virtual machines on: Workstation Pro for Windows or Linux, Fusion Pro for macOS.
- Run the installer and follow the prompts to complete the installation.

### Installing the VM

Unlike when you install a new VM from scratch, this VM has already been built and customized to have all the software installed that you need to complete the activities in this workshop.

> [!warning]
> The default unzipping utilities on both Macs and Windows systems are typically not able to handle large archive files like the ones we create when using VMs. It is recommended that you install either 7-Zip or Keka, depending on your host OS.

### Running a Linux Virtual Machine in VMware

1. **Download the VM archive**
    - In the files you downloaded, find the archive, a single `.zip` that contains both VMs.
2. **Install or verify extraction software**
    - **Windows:** Install [7-Zip](https://www.7-zip.org/) if it’s not already on your system.
    - **Mac:** Install [Keka](https://www.keka.io/) if it’s not already on your system.

3. **Extract the archive**
    - Choose a location where you want the VMs to live and run from.
        - Example: a dedicated folder such as `C:\VMs\`, or your **Desktop** if that’s easiest.
        - You can move it later, but it’s best to pick the permanent location now.
    - **Windows:** Right-click the `.zip` file → **7-Zip → Extract Here** (or extract to your chosen folder).
    - **Mac:** Right-click the `.zip` file → **Open With → Keka** → extract to your chosen folder.
    - You should end up with two VM folders side by side, one for **HeartOfGold** and one for **Marvin**. Keep them together under the same parent folder.

4. **Open VMware Workstation**
    - Start **VMware Workstation**. You'll use this same window to open both VMs.

5. **Open a VM**
    - In VMware, go to **File → Open…**
    - Browse to the VM folder you extracted, either **HeartOfGold** or **Marvin**.
    - Select the file ending in `.vmx`

6. **Start the VM**
    - Click **Power on this virtual machine**.
    - Ubuntu will boot and be ready to use.

> [!info] Do This for Both VMs
> Repeat steps 5 and 6 for the second VM. Both HeartOfGold and Marvin need to be powered on at the same time for this workshop, and VMware Workstation can run them side by side without issue.

---

### Troubleshooting Tips

- **If you accidentally open the wrong file**:
    - Always select the `.vmx` file, not `.vmdk` or other files.

- **If VMware asks “Did you move or copy this VM?”**:
    - Choose **“I copied it”**.
    - This ensures VMware assigns new IDs so the VM won’t conflict with others.

- **If you want to move the VM later**:
    - Shut down the VM first.
    - Move the entire folder (not just some files) to the new location.
    - Open VMware → **File → Open…** → select the `.vmx` file in its new location.

## Logging into the VMs

**HeartOfGold** (server):
- Username: `frankie`
- Password: `KeepS3rversS3cur3`

**Marvin** (client):
- Username: `benjy`
- Password: `LLMs4evr`

## Reading the Manual on Marvin

You do not have to keep this manual open on your own computer. A copy of it is already on Marvin, and reading it there is easier once the labs get going, because you can copy a command straight from the manual into the terminal beside it.

Log in to Marvin and look at the desktop. **Obsidian** is the app that displays the manual. Open it and it lands on lesson 00, with the whole manual in the sidebar. Everything renders the way it was meant to: the colored command boxes that tell you which machine a command belongs to, the checkpoints at the end of each lesson, and the Previous and Next links at the bottom of every page.

The manual itself lives at `~/Documents/Antisyphon/AT-Workshop-Keeping-Things-Local-2026` if you ever want to find the files directly.

> [!tip] Two Windows, Side by Side
> The setup most people settle on is Obsidian on one half of Marvin's screen and a terminal on the other. From [[05 Tailscale Mesh Networking]] onward you will also have a second terminal holding an SSH session to HeartOfGold, so the manual, the Marvin shell, and the HeartOfGold shell are all visible at once.

### Getting the Latest Version

The manual gets corrections. There is a **Update Lab Manual** icon on Marvin's desktop that fetches the newest version from GitHub. Double-click it, and a terminal window opens and reports what it did.

You do not need to run it to start the workshop. Reach for it if you are told a lesson has been fixed, or if something in the manual clearly disagrees with what you are seeing on screen.

> [!warning] Updating clears your ticked checkpoints
> Ticking a checkpoint box actually edits the manual file it sits in, so updating replaces those edits along with everything else. If you have been ticking boxes as you go, an update will clear them.
>
> This only affects the manual. Your VMs, your models, and everything you have built in the labs are untouched. The updater tells you before it does anything and gives you the chance to back out, so nothing is lost by accident.

*That's it! We're ready to roll!*

> [!checkpoint] Checkpoint
> You have finished this lesson when all of the boxes below are ticked. Work through them in order, and if one does not hold, go back to the section it came from before moving on. Tick each box as you confirm it.
>
> - [ ] Both VMs extract from the archive into their own folders, side by side
> - [ ] HeartOfGold and Marvin both power on and boot to Ubuntu in VMware
> - [ ] You can log in to HeartOfGold as `frankie` and to Marvin as `benjy`
> - [ ] Both VMs are running at the same time
> - [ ] Obsidian opens on Marvin and shows this manual

---

> [!nav]
> [[01 What is an LLM]]
>
> [[03 Working with Ollama]]
