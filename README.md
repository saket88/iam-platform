# iam-platform

## Step 1: Prepare the External SSD

First, ensure your **"T7" SSD** is connected to your Mac.

### 1. Check the Format

For best performance and compatibility on macOS, the drive should be formatted as **APFS** (Apple File System) or **Mac OS Extended (Journaled)**.

- Open **Disk Utility** (you can find it in `Applications > Utilities`).
- Select your "T7" drive from the list on the left.
- Look at the **Format** information.
  - If it's already **APFS**, you're good to go.
  - If it's **not APFS** (e.g., it's **ExFAT** or **NTFS**), you must reformat it. **Be aware that reformatting will erase everything on the drive.**
    - Click the **Erase** button in Disk Utility
    - Give it a name (like `T7`)
    - Choose the `APFS` format

### 2. Confirm the Path

After formatting, the drive should appear in Finder and be available at the path:

