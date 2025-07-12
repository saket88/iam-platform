# iam-platform
Step 1: Prepare the External SSD

  First, ensure your "T7" SSD is connected to your Mac.

   1. Check the Format: For best performance and compatibility on macOS, the drive should be formatted as APFS (Apple File System) or Mac 
      OS Extended (Journaled).
       * Open Disk Utility (you can find it in Applications/Utilities).
       * Select your "T7" drive from the list on the left.
       * Look at the "Format" information. If it's already APFS, you're good to go.
       * If it's not APFS (e.g., it's ExFAT or NTFS), you must reformat it. Be aware that reformatting will erase everything on the drive.
         To do this, click the "Erase" button in Disk Utility, give it a name (like "T7"), and choose the "APFS" format.

   2. Confirm the Path: After formatting, the drive should appear in Finder and be available at the path /Volumes/T7.

  Step 2: Create the Project Directory on the SSD

  This is where we will store everything for our IAM platform: the k0s cluster data and all our Kustomize configuration files. Open
  your terminal and run this command:

   1 mkdir -p /Volumes/T7/iam-platform/k0s-data

  This creates a main folder iam-platform and a subfolder k0s-data which will hold the actual cluster state.

  Step 3: Install and Start the k0s Cluster on the SSD

  Now, we'll use the k0s command you installed, but we'll use the --data-dir flag to force it to save everything on your SSD instead of
   your main computer.

  Run these two commands in your terminal, one after the other:

   1. Install the k0s controller: This command sets up the control plane but doesn't start it yet. All the necessary certificates and
      configuration files will be generated inside the directory we just made.

   1     k0s install controller --single --data-dir /Volumes/T7/iam-platform/k0s-data

   2. Start the k0s service: This command actually starts the Kubernetes cluster as a background process.

   1     sudo k0s start --data-dir /Volumes/T7/iam-platform/k0s-data
      (You will likely need to enter your Mac's password for the `sudo` command here.)

  Step 4: Get Your Cluster Configuration (Kubeconfig)

  Your cluster is now running, but your kubectl command-line tool doesn't know how to connect to it yet. We need to get the connection
  configuration file, called a "kubeconfig".

   1. Generate the Kubeconfig: Use this command to get the configuration from your new k0s cluster:

   1     sudo k0s kubeconfig --data-dir /Volumes/T7/iam-platform/k0s-data > /Volumes/T7/iam-platform/kubeconfig.yaml
      This creates a file named kubeconfig.yaml inside your project folder on the SSD.

   2. Tell your terminal to use this file: To interact with your cluster, you need to set an environment variable. Run this command:

   1     export KUBECONFIG=/Volumes/T7/iam-platform/kubeconfig.yaml
      Important: You will need to run this export command every time you open a new terminal window to work on the project.

  Step 5: Verify Everything is Working

  Now, run a standard kubectl command to see if you can connect to your new cluster running on the SSD:

   1 kubectl get nodes

  If everything worked, you should see output showing one node with a "Ready" status.
