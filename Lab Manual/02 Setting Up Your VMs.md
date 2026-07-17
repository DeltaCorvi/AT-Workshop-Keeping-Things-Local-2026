---
author: Bronwen Aker
updated: 2026-07-13
presentation_type: Workshop
venue: Antisyphon AI Summit
---

```table-of-contents
title: # Table of Contents
minLevel: 0
maxLevel: 3
```

# Welcome

This course uses two virtual machines: 
* **HeartOfGold**, which acts as the server, hosting the local LLM that you will install and configure.
* **Marvin**, which you will use to access it.  

The connection between them is established and maintained using Tailscale, an encrypted mesh networking tool that lets two machines reach each other securely without manual VPN configuration or exposing services to the open internet. For penetration testers and red teamers, this is the same technique used to maintain persistent, encrypted access to internal services during an engagement. Here, we're using it to reach a private LLM instead.


## Setting Up VMware 

This lab is set up to use VMware Workstation Pro version 17, or VMware Fusion 13. Installers and course files, including both VM images, will be provided as a zip archive ahead of the workshop. Registered students will receive the download link via email.

VMware and VMware Fusion installation files are available for download. 

None of the software included in the VM requires a license for personal use. Some applications do have requirements for commercial or enterprise use. You should be able to make use of the VM and the software on it for a long time after this course is over. 

Here is the download link: TBD

> [!todo] Reminder to self: add the actual download link here before final delivery of course assets. Likely to be a personal Dropbox link again, but delivery method is not yet finalized.


### System Requirements

This workshop requires HeartOfGold and Marvin running at the same time. The two VMs are not symmetric, so host requirements are a sum of both, not a flat doubling:

- **Marvin (client, headless):** confirmed at 8 GB RAM, 2 processors, 20 GB disk.
- **HeartOfGold (server):** TBD. Depends on which model(s) it needs to run.

> [!todo] Reminder to self: fill in HeartOfGold's confirmed RAM/disk/cores once it's built and tested, then update the host minimums below (currently Marvin's 8 GB/20 GB plus a TBD placeholder for HeartOfGold, plus host OS overhead).

---
#### VMware Workstation 17 (Windows)

| Component    | Minimum                                      | Recommended                                |
| ------------ | -------------------------------------------- | ------------------------------------------ |
| **CPU**      | 64-bit, 4 cores (2 for Marvin + TBD for HeartOfGold), virtualization support (Intel VT-x / AMD-V) enabled in BIOS/UEFI | Quad-core+ 64-bit (Intel i5/i7 or Ryzen 5+) |
| **RAM**      | 8 GB (Marvin) + TBD (HeartOfGold) + host overhead | TBD once HeartOfGold is finalized  |
| **Disk**     | 20 GB (Marvin) + TBD (HeartOfGold)           | SSD, TBD once HeartOfGold is finalized |
| **Host OS**  | Windows 10/11 64-bit                         | Windows 10/11 64-bit                       |
| **Display**  | 1280×800                                     | 1920×1080 or higher                        |
| **GPU (3D)** | DirectX 11 / OpenGL 4.3 capable              | DirectX 11 capable GPU                     |

---
#### VMware Fusion 13 (macOS):

| Component    | Minimum                            | Recommended                                          |
| ------------ | ---------------------------------- | ---------------------------------------------------- |
| **CPU**      | Quad-core Intel (64-bit) or Apple Silicon, virtualization support enabled | Quad-core Intel i5/i7/i9 or Apple Silicon (M1/M2/M3) |
| **RAM**      | 8 GB (Marvin) + TBD (HeartOfGold) + host overhead | TBD once HeartOfGold is finalized |
| **Disk**     | 20 GB (Marvin) + TBD (HeartOfGold) | SSD, TBD once HeartOfGold is finalized        |
| **Host OS**  | macOS 12 (Monterey) or later       | Latest macOS (Ventura / Sonoma)                      |
| **Display**  | 1280×800                           | 1920×1080 or higher                                  |
| **GPU (3D)** | Metal-capable GPU (for Intel Macs) | Metal-capable GPU / Apple Silicon GPU                |

---

> [!warning] If you have Apple Silicon (M1/M2/M3):
>- Only **ARM-based guest OSes** are supported (ARM Linux, Windows 11 ARM).
>- Intel x86 guest OSes (e.g., older Windows/Linux ISOs) will not run under Fusion on Apple Silicon..

> [!info] About RAM... 
> Theoretically, RAM stands for "random access memory", but in reality it stands for "rarely adequate memory"! 😉 In short, if you have the system resources to increase the amount of RAM allocated to your VM, DO IT!

---

### Installing VMware

Included in the file bundle for this workshop you will find installer files for VMware Workstation and VMware Fusion. Sorry, no support for VirtualBox at this time. 

- Find the installer appropriate to the operating system for the host on which you will be running the virtual machine. 
- Copy the installer file to the host system. 
- Double click on the file and follow the prompts to complete the installation process.

### Installing the VM

Unlike when you install a new VM from scratch, this VM has already been built and customized to have all the software installed that you need to complete the activities in this workshop. 

> [!warning]
> The default unzipping utilities on both Macs and Windows systems are typically not able to handle large archive files like the ones we create when using VMs. It is recommended that you install either 7-Zip or Keka, depending on your host OS.

### Running a Linux Virtual Machine in VMware 

1. **Download the VM archive**    
    - In the files you downloaded, find `filename.zip`. This single archive contains both VMs.
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

> [!info] Repeat steps 5 and 6 for the second VM. Both HeartOfGold and Marvin need to be powered on at the same time for this workshop, and VMware Workstation can run them side by side without issue.

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


*That's it! We're ready to roll!*



---

< Previous - [[01 What is an LLM]] | [[03 Working with Ollama]] - Next >