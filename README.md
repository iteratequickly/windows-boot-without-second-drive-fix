# Fix: Windows Will Not Boot Without Second Hard Drive | EFI System Partition Recovery

> **Problem:** Windows fails to boot when secondary hard drive is disconnected? This guide shows how to fix Windows boot dependency on a second drive by relocating the EFI System Partition to your primary SSD.

## The Problem: Windows Requires Second Drive to Boot

Many users encounter a frustrating issue where **Windows will not boot without the second hard drive connected**. This happens when the EFI System Partition (ESP) is located on a secondary drive (HDD) while Windows itself is installed on the primary drive (SSD). When you disconnect the secondary drive, the BIOS cannot find the "Windows Boot Manager" entry, and your system fails to boot.

### Common Symptoms
- Windows boots fine with both drives connected
- Removing the secondary HDD causes boot failure
- BIOS shows "No bootable device" or "Windows Boot Manager not found"

## System Configuration

- **Boot Drive:** 1TB NVMe SSD (GPT)
- **Storage/Backup Drive:** 1TB HDD (GPT)
- **BIOS Mode:** UEFI with CSM Disabled

## Solution: Move EFI Partition to Primary SSD

This guide will help you fix Windows boot issues by creating a new EFI System Partition on your primary drive and rebuilding the Windows bootloader, eliminating the dependency on the secondary drive.

### Prerequisites
- Windows installation media or recovery environment
- Administrator access
- Both drives currently connected

### Step 1: Create New EFI Partition on SSD

First, we will shrink your Windows partition and create a new 100MB EFI System Partition on the primary SSD.
```cmd
diskpart
list disk
select disk 0
list partition
select partition 1
shrink desired=100
create partition efi size=100
format fs=fat32 quick
assign letter=S
exit
```

**What this does:** Creates a FAT32-formatted EFI partition at the beginning of your primary drive.

### Step 2: Rebuild Windows Bootloader

Copy the Windows boot files to the new EFI partition on your SSD.
```cmd
bcdboot W:\Windows /s S: /f UEFI
```

**Important:** Replace `W:\Windows` with your actual Windows installation path and `S:` with the letter you assigned to your new EFI partition.

### Step 3: Create Fail-Safe Boot Path

This step ensures UEFI can boot Windows even if the Boot Manager entry is missing.
```cmd
mkdir S:\EFI\Boot
copy S:\EFI\Microsoft\Boot\bootmgfw.efi S:\EFI\Boot\bootx64.efi
```

**Why this matters:** The `bootx64.efi` file is the default UEFI boot file that all UEFI systems look for when no other boot entry is found.

### Step 4: Remove Old EFI Partition from Secondary Drive

Now that Windows boots from the SSD, remove the old EFI partition from your HDD to prevent confusion.
```cmd
diskpart
list disk
select disk 1
list partition
select partition 1
delete partition override
exit
```

> **Note:** This will leave 101 MB of unallocated space at the start of the HDD. This space is too small to merge with the main partition and can safely be left as-is.

## Verification and Testing

1. **Test boot with both drives:** Restart and ensure Windows boots normally
2. **Test boot with SSD only:** Shut down, disconnect the secondary HDD, and boot again
3. **Check BIOS boot order:** Ensure your SSD is listed first in the boot priority

✅ **Success:** Your system should now boot independently from the SSD without requiring the secondary drive.

## Troubleshooting

### Windows Boot Manager Not Found After Following Steps
- Verify the EFI partition was created correctly: Use `diskpart` and `list partition` to check
- Ensure the bootloader was copied: Check that `S:\EFI\Microsoft\Boot\` contains files
- Try the fail-safe boot path again: Make sure `bootx64.efi` exists in `S:\EFI\Boot\`

### Cannot Create EFI Partition (Not Enough Space)
- Shrink your Windows partition further using Disk Management
- Consider using third-party tools like GParted for more flexible resizing

### BIOS Still Boots from HDD
- Check BIOS boot order and move SSD to first position
- Disable any legacy/CSM boot options
- Remove the old Windows Boot Manager entry in BIOS (if present)

## Maintenance and Recommendations

### Monitor Windows Updates
After major Windows updates, occasionally check that your boot configuration remains intact. If issues arise, re-run:
```cmd
bcdboot C:\Windows /s S: /f UEFI
```

### Optional: Reclaim HDD Space
The 101 MB unallocated space on your HDD is negligible but can be reclaimed using third-party tools like:
- MiniTool Partition Wizard
- AOMEI Partition Assistant
- GParted (Linux-based)

### UEFI Boot Priority
Always ensure your primary SSD is first in the BIOS boot order to prevent boot delays or fallback to the old HDD entry.

## Technical Background

### Why Does This Happen?
This issue commonly occurs when:
1. **Clone/migration tools** copy Windows but not the boot partition
2. **Multiple drives** are present during Windows installation
3. **UEFI firmware** defaults to the first available EFI partition (often on the secondary drive)

### Understanding EFI System Partition (ESP)
The ESP is a small FAT32 partition that stores boot loaders and boot configuration files for UEFI-based systems. Every bootable Windows drive should have its own ESP to be independent.

## Related Issues and Keywords
- Windows will not boot after removing second hard drive
- Windows Boot Manager on wrong drive
- How to move EFI partition to another drive
- Fix Windows boot dependency on HDD
- Windows requires secondary drive to boot
- UEFI boot partition on wrong disk
- Windows bootloader not found after disk removal

## Additional Resources

- [Microsoft: BCDboot Command-Line Options](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/bcdboot-command-line-options-techref-di)
- [Understanding GPT and UEFI Boot](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/configure-uefigpt-based-hard-drive-partitions?view=windows-11)

---

**Keywords:** windows will not boot without second hard drive, windows boot manager on wrong drive, EFI partition recovery, fix windows boot dependency, windows requires secondary drive, UEFI boot fix, move EFI system partition, windows boot repair SSD HDD
