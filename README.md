# IAM Platform Setup on External SSD

---

## Step 1: Prepare the External SSD

Ensure your "T7" SSD is connected to your Mac.

### 1. Check the Format

For best performance and compatibility on macOS, the drive should be formatted as **APFS** or **Mac OS Extended (Journaled)**.

- Open **Disk Utility** (found in Applications > Utilities).
- Select your "T7" drive from the list on the left.
- Look at the "Format" field.
  - If it's **APFS**, you're good to go.
  - If it's **ExFAT**, **NTFS**, etc., reformat the drive (⚠️ This will erase all contents):
    - Click **Erase**.
    - Name it something like `T7`.
    - Choose **APFS** format and confirm.

### 2. Confirm the Path

After formatting, the drive should appear in Finder at:

```
/Volumes/T7
```

---

## Step 2: Create the Project Directory on the SSD

This directory will hold the **k0s cluster data** and **Kustomize configs**.

Open your terminal and run:

```bash
mkdir -p /Volumes/T7/iam-platform/k0s-data
```

This creates the structure:
```
/Volumes/T7/
└── iam-platform/
    └── k0s-data/
```

---

## Step 3: Install and Start the k0s Cluster on the SSD

We'll run `k0s` using the `--data-dir` flag to save everything on the SSD.

### 1. Install the Controller

```bash
k0s install controller --single --data-dir /Volumes/T7/iam-platform/k0s-data
```

This generates certificates and configs, but doesn't start the cluster yet.

### 2. Start the Cluster

```bash
sudo k0s start --data-dir /Volumes/T7/iam-platform/k0s-data
```

You’ll need to enter your Mac password for `sudo`.

---

## Step 4: Get Your Cluster Configuration (Kubeconfig)

Your cluster is running, but `kubectl` doesn’t know how to connect yet.

### 1. Generate the Kubeconfig

```bash
sudo k0s kubeconfig --data-dir /Volumes/T7/iam-platform/k0s-data > /Volumes/T7/iam-platform/kubeconfig.yaml
```

This writes the config file to your SSD.

### 2. Set the KUBECONFIG Environment Variable

```bash
export KUBECONFIG=/Volumes/T7/iam-platform/kubeconfig.yaml
```

> ✅ You'll need to run this **every time** you open a new terminal.

---

## Step 5: Verify Everything is Working

Test the setup:

```bash
kubectl get nodes
```

✅ You should see one node with status **Ready**.
