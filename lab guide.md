# gcp-5th-7th

# Lab Guide 
# Lab 1 : Deploying a VM in GCP with Ubuntu and Nginx

## Objective
In this lab, you will:
1. Create a Virtual Machine (VM) in Google Cloud Platform (GCP) with an Ubuntu OS disk.
2. SSH into the VM.
3. Install Nginx on the VM.
4. Access the Nginx welcome page via a web browser.



## Step 1: Create a VM Instance in GCP
### Using Google Cloud Console
1. Navigate to the [Google Cloud Console](https://console.cloud.google.com/).
2. Select your project from the top navigation bar.
3. In the left menu, go to **Compute Engine** > **VM instances**.
4. Click **Create Instance**.
5. Configure the instance:
   - **Name**: `ubuntu-nginx-vm`
   - **Region**: Select a region close to you.
   - **Machine Configuration**:
     - Machine family: General-purpose
     - Series: `E2`
     - Machine type: `e2-medium` (2 vCPUs, 4GB RAM)
   - **Boot Disk**:
     - Click **Change**.
     - Select **Ubuntu** (e.g., Ubuntu 22.04 LTS).
     - Click **Select**.
   - **Firewall**:
     - Check **Allow HTTP traffic**.
     - Check **Allow HTTPS traffic**.
6. Click **Create** and wait for the VM to be provisioned.

---

## Step 2: SSH into the VM
### Using Cloud Console
1. Go to **Compute Engine** > **VM Instances**.
2. Click **SSH** next to your VM name.

---

## Step 3: Install Nginx
Once logged into the VM, run the following commands:
```sh
sudo apt update
sudo apt install -y nginx
```

Start and enable Nginx to run on boot:
```sh
sudo systemctl start nginx
sudo systemctl enable nginx
```

Check the status:
```sh
sudo systemctl status nginx
```

---

## Step 4: Allow HTTP Traffic
### Verify Firewall Rules
1. In **Compute Engine**, go to **VM Instances**.
2. Click on your VM name.
3. Under the **Network interfaces** section, ensure **Allow HTTP traffic** is enabled.

---

## Step 5: Access the Nginx Welcome Page
1. In **Compute Engine**, go to **VM Instances**.
2. Copy the **External IP** of your VM.
3. Open a web browser and enter:
   ```
   http://<EXTERNAL_IP>
   ```
4. You should see the Nginx welcome page.

---

## Cleanup (Optional)
To avoid charges, delete the VM when you're done:
1. Go to **Compute Engine** > **VM Instances**.
2. Select your VM.
3. Click **Delete** and confirm.

---

## Summary
You have successfully:
- Created an Ubuntu VM in GCP.
- Connected to the VM via SSH.
- Installed and started Nginx.
- Accessed the Nginx welcome page through your browser.

# Lab 2: Creating an Instance Template and Managed Instance Group in GCP

## Objective
In this lab, you will:
1. Create an **Instance Template** with an Ubuntu OS and a startup script to install Nginx.
2. Use this template to create a **Managed Instance Group** with auto-healing and auto-scaling enabled.
3. Stress test the instance group to observe auto-healing and scaling behavior.

---

## Prerequisites
- A Google Cloud account with billing enabled (Free Tier is sufficient).
- Basic familiarity with Compute Engine.

---

## Step 1: Create an Instance Template
### Using Google Cloud Console
1. Navigate to the [Google Cloud Console](https://console.cloud.google.com/).
2. Select your project from the top navigation bar.
3. In the left menu, go to **Compute Engine** > **Instance templates**.
4. Click **Create instance template**.
5. Configure the template:
   - **Name**: `nginx-template`
   - **Region**: Select a preferred region.
   - **Machine Configuration**:
     - Machine family: General-purpose
     - Series: `E2`
     - Machine type: `e2-micro`
   - **Boot Disk**:
     - Click **Change**.
     - Select **Ubuntu** (e.g., Ubuntu 22.04 LTS).
     - Click **Select**.
   - **Automation**:
     - Under **Automation** In management, paste the following script:

```bash
#! /bin/bash
apt update && apt install -y nginx
INSTANCE_ID=$(curl -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/instance/id)
HOSTNAME=$(curl -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/instance/name)
INTERNAL_IP=$(curl -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/instance/network-interfaces/0/ip)
echo "<h1>Welcome to NGINX</h1>" > /var/www/html/index.html
echo "<p>Instance ID: $INSTANCE_ID</p>" >> /var/www/html/index.html
echo "<p>Hostname: $HOSTNAME</p>" >> /var/www/html/index.html
echo "<p>Internal IP: $INTERNAL_IP</p>" >> /var/www/html/index.html
systemctl restart nginx
```
   - **Firewall**:
     - Check **Allow HTTP traffic**.
     - Check **Allow HTTPS traffic**.
6. Click **Create** to save the template.

---

## Step 2: Create a Managed Instance Group
1. Navigate to **Compute Engine** > **Instance groups**.
2. Click **Create instance group**.
3. Configure the instance group:
   - **Name**: `nginx-instance-group`
   - **Location**: Select a region.
   - **Instance template**: Select `nginx-template`.
   - **Autoscaling**: Enable autoscaling.
     - **Minimum instances**: `2`
     - **Maximum instances**: `3`
   - **Health check**:
     - Protocol: `HTTP`
     - Port: `80`
     - Check interval: `60s`
     - Timeout: `5s`
     - Unhealthy threshold: `10`
     - Healthy threshold: `2`
4. Click **Create**.

---

## Step 3: Verify Auto-Healing and Auto-Scaling
### Check Auto-Healing
1. Navigate to **Compute Engine** > **Instance groups**.
2. Click on `nginx-instance-group`.
3. Identify an instance and SSH into it.
4. Stop the Nginx service:
   ```sh
   sudo systemctl stop nginx
   ```
5. Wait for a few minutes and check if the instance is automatically replaced.

### Check Auto-Scaling
1. SSH into an instance and install stress testing tools:
   ```sh
   sudo apt update && sudo apt install -y stress
   ```
2. Run the stress test to consume CPU:
   ```sh
   stress --cpu 2 
   ```
3. Monitor the instance group in the Google Cloud Console.
   - If CPU usage crosses the threshold, a new instance should be added.
   - After the stress test ends, the group should scale down automatically.

---

## Summary
You have successfully:
- Created an **Instance Template** with a startup script for Nginx.
- Created a **Managed Instance Group** with auto-healing and auto-scaling.
- Tested auto-healing by stopping Nginx.
- Tested auto-scaling by applying CPU stress.

# Lab 3 : Creating a Regional External Load Balancer in GCP

## Objective
In this lab, you will:
1. Use the existing **Managed Instance Group** (`nginx-instance-group`) as the backend.
2. Create a **Regional External HTTP Load Balancer** to distribute traffic.
3. Verify the load balancer setup by accessing the external IP.

---

## Prerequisites
- A running **Managed Instance Group** (`nginx-instance-group`) from the previous lab.
- Basic familiarity with Compute Engine and Load Balancing in GCP.

---

## Step 1: Configure the Frontend
1. Under **Frontend Configuration**, click **Add Frontend IP and Port**.
2. **Protocol**: `HTTP`
3. **IP Version**: `IPv4`
4. **IP Address**: Click **Reserve a new static IP** and name it `nginx-lb-ip`.
5. **Port**: `80`
6. Click **Done**.

---
## Step 2: Create a Backend Service
1. Navigate to **Networking** > **Load balancing** in the [Google Cloud Console](https://console.cloud.google.com/).
2. Click **Create Load Balancer**.
3. Select **HTTP(S) Load Balancing** and click **Start Configuration**.
4. Choose **From Internet to my VMs** and click **Continue**.
5. Under **Backend Configuration**:
   - Click **Create a Backend Service**.
   - **Name**: `nginx-backend-service`
   - **Backend Type**: `Instance Group`
   - **Instance Group**: Select `nginx-instance-group`
   - **Port Numbers**: `80`
   - **Health Check**: Select an existing one or create a new one:
     - Protocol: `HTTP`
     - Port: `80`
     - Request Path: `/`
   - Click **Create**.

---



## Step 3: Review and Create Load Balancer
1. Click **Review and Finalize**.
2. Ensure the backend and frontend settings are correct.
3. Click **Create**.
4. Wait for the load balancer to provision.

---

## Step 4: Verify Load Balancer
1. Go to **Load Balancing** > **Load Balancers**.
2. Copy the **External IP** of the load balancer.
3. Open a web browser and enter:
   ```
   http://<LOAD_BALANCER_EXTERNAL_IP>
   ```
4. You should see the **NGINX welcome page** with instance details.

---

## Step 5: Test Load Balancing
1. Refresh the browser multiple times.
2. You should see different instance details, confirming the load balancer is distributing traffic.

---

## Cleanup (Optional)
To avoid charges, delete the load balancer:
1. Go to **Load Balancing** and delete `nginx-lb`.
2. Release the reserved static IP (`nginx-lb-ip`) from **VPC Network** > **External IP addresses**.
3. Go to Compute Engine > Instance groups and delete nginx-instance-group.
4. Go to Compute Engine > Instance templates and delete nginx-template

---

## Summary
You have successfully:
- Created a **Regional External HTTP Load Balancer**.
- Configured it to distribute traffic to a **Managed Instance Group**.
- Verified load balancing by accessing the external IP.

# Lab 4: Creating a VM and Setting Up Observability in GCP

## Objective
In this lab, you will:
1. Create a **VM instance** with Ubuntu OS.
2. Enable the **Ops Agent** during VM creation for advanced monitoring.
3. Monitor the VM using Cloud Monitoring.
4. Check different metrics from the VM.
5. Create an **alert policy** with an email notification channel.
6. Install and use the **stress** tool to test the alerting mechanism.

---

## Prerequisites
- A Google Cloud account with billing enabled (Free Tier is sufficient).
- Basic familiarity with Compute Engine and Cloud Monitoring.

---

## Step 1: Create a VM Instance

### Using Google Cloud Console
1. Navigate to the [Google Cloud Console](https://console.cloud.google.com/).
2. Select your project.
3. Go to **Compute Engine** > **VM Instances**.
4. Click **Create Instance**.
5. Configure the VM:
   - **Name**: `monitoring-vm`
   - **Region**: Select a preferred region.
   - **Machine Configuration**:
     - Machine type: `e2-micro`
   - **Boot Disk**:
     - Click **Change**.
     - Select **Ubuntu** (e.g., Ubuntu 22.04 LTS).
     - Click **Select**.
   - **Observability**:
     - Go to the **Observability** tab.
     - Check **Enable Google Cloud Ops Agent** to install it automatically.
   - **Firewall**:
     - Check **Allow HTTP traffic** (optional).
6. Click **Create** and wait for the VM to start.

---

## Step 2: Verify Ops Agent Installation
1. SSH into the VM:
   - Go to **Compute Engine** > **VM Instances**.
   - Click **SSH** next to `monitoring-vm`.
2. Verify that the **Ops Agent** is running:
   ```sh
   systemctl status google-cloud-ops-agent*
   ```
   If the agent is running, the installation was successful.

---

## Step 3: Set Up Cloud Monitoring

1. Navigate to VM list and click on the VM name in Google Cloud Console.
2. Click **Observability**.
3. Select **Compute Engine VM Instance** as the resource type.
4. Choose a metric to observe, such as:
   - **CPU Usage** (`compute.googleapis.com/instance/cpu/utilization`)
   - **Memory Usage**
   - **Disk I/O**
5. Monitor the metric graphs to get real-time insights.

---

## Step 4: Check Different Metrics from the VM
1. In **Metrics Explorer**, explore different system metrics:
   - **Network traffic**: `compute.googleapis.com/instance/network/received_bytes_count`
   - **Disk read/write operations**: `compute.googleapis.com/instance/disk/write_bytes_count`
   - **Process count**: `agent.googleapis.com/processes/count`
2. Observe how these metrics change over time and correlate with system activity.

---

## Step 5: Create an Alert Policy

### Create a Notification Channel
1. In **Monitoring**, go to **Alerting** > **Notification Channels**.
2. Click **Add New** > **Email**.
3. Enter your email address and save it.

### Create an Alert Policy
1. In **Monitoring**, go to **Alerting** > **Create Policy**.
2. Click **Select a condition**.
3. Set the condition:
   - **Target**: Select `monitoring-vm`.
   - **Metric**: `CPU Utilization`
   - **Threshold**: Set to `80%` for testing.
4. Click **Next**.
5. Add **Notification Channel** (your email).
6. Name the alert policy and click **Save**.

---

## Step 6: Stress Test the VM

### Install the Stress Tool
1. SSH into the VM if not already connected.
2. Run the following commands:
   ```sh
   sudo apt update
   sudo apt install -y stress
   ```

### Generate CPU Load
Run:
```sh
stress --cpu 2 
```
This will simulate high CPU usage for 5 minutes.

---

## Step 7: Verify the Alert
1. Monitor CPU usage in **Cloud Monitoring**.
2. Wait for the alert to trigger (can take a few minutes).
3. Check your email for an alert notification.
4. Once verified, stop the stress test using:
   ```sh
   pkill stress
   ```

---

## Cleanup (Optional)
To avoid charges:
1. Delete the **VM Instance**.
2. Remove the **Alert Policy** and **Notification Channel** in Cloud Monitoring.

---

## Summary
You have successfully:
- Created an **Ubuntu VM** in GCP.
- Enabled the **Ops Agent** during VM creation.
- Set up **Cloud Monitoring**.
- Checked different system metrics.
- Configured an **alert policy** with email notifications.
- Simulated CPU stress to test alerting.

# Lab 5: Granting IAM Permissions for Incident Viewing in GCP

## Objective
In this lab, you will:
1. Grant **IAM roles** to an email used in a **notification channel**.
2. Assign **Monitoring Viewer** and **Logs Viewer** roles.
3. Verify that the user can access the **Incident page** from an email notification.

---

## Prerequisites
- A Google Cloud account with billing enabled.
- An alert policy set up with an **email notification channel**.
- Owner or IAM Admin permissions on the GCP project.

---

## Step 1: Navigate to IAM in Google Cloud Console
1. Open the [Google Cloud Console](https://console.cloud.google.com/).
2. Select your project from the top menu.
3. Navigate to **IAM & Admin** > **IAM**.

---

## Step 2: Grant IAM Roles to the Email
1. Click **+ Grant Access** at the top.
2. In the **New principals** field, enter the email address that was used as the **notification channel**.
3. Click **Select a role** and assign the following roles:
   - **Monitoring Viewer** (roles/monitoring.viewer): Provides read access to monitoring data and dashboards.
   - **Logs Viewer** (roles/logging.viewer): Allows viewing of log entries for troubleshooting.
4. Click **Save**.

---

## Step 3: Verify Access to Incident Page
1. Check your email for an alert notification from GCP.
2. Click the **View Incident** link in the email.
3. Ensure you can access the **Incident Details Page** in Cloud Monitoring.
4. If access is denied, verify that the correct email has been granted the roles and try again.

---

## Cleanup (Optional)
To remove access:
1. Go to **IAM & Admin** > **IAM**.
2. Find the email under **Principals**.
3. Click the **Edit** icon and remove the assigned roles.
4. Click **Save**.

---

## Summary
You have successfully:
- Assigned **Monitoring Viewer** and **Logs Viewer** roles to an email.
- Verified that the email recipient can access the **Incident Details Page**.
- Ensured secure and limited access to monitoring data.

# Lab 6: Using a Service Account to Access a Cloud Storage Bucket

## Objective
In this lab, you will:
1. Create a **Cloud Storage bucket** and upload a text file.
2. Create a **service account** with permissions to download the file.
3. Assign the service account to a **VM instance**.
4. Verify that the VM can download the file.
5. Remove the permissions and confirm that access is denied.

---

## Prerequisites
- A Google Cloud account with billing enabled.
- Basic familiarity with Cloud Storage, IAM, and Compute Engine.

---

## Step 1: Create a Cloud Storage Bucket
1. Open the [Google Cloud Console](https://console.cloud.google.com/).
2. Navigate to **Cloud Storage** > **Buckets**.
3. Click **Create** and configure:
   - **Name**: `my-storage-bucket-<your-unique-id>`
   - **Region**: Choose a region
   - **Storage class**: Standard (default)
4. Click **Create**.

---

## Step 2: Upload a Text File
1. Open your newly created bucket.
2. Click **Upload Files** and select a text file (`sample.txt`).
3. Click **Upload** and verify that the file is in the bucket.

---

## Step 3: Create a Service Account
1. Navigate to **IAM & Admin** > **Service Accounts**.
2. Click **Create Service Account**.
3. Set:
   - **Name**: `storage-access-sa`
   - **ID**: `storage-access-sa`
   - **Description**: `Service account for storage access`
4. Click **Create & Continue**.
5. Assign **Storage Object Viewer** role.
6. Click **Done**.

---

## Step 4: Create a VM with the Service Account
1. Navigate to **Compute Engine** > **VM Instances**.
2. Click **Create Instance** and configure:
   - **Name**: `storage-vm`
   - **Machine Type**: `e2-micro`
   - **Boot Disk**: Ubuntu 22.04 LTS
   - **Service Account**: Select `storage-access-sa`
   - **Access Scopes**: Select "Allow full access to all Cloud APIs" (or set custom scopes for storage access).
3. Click **Create**.

---

## Step 5: Verify File Download
1. SSH into the VM.
2. Run the following command to download the file:
   ```sh
   gsutil cp gs://my-storage-bucket-<your-unique-id>/sample.txt .
   ```
3. Confirm that the file is successfully downloaded.

---

## Step 6: Remove Permissions from the Service Account
1. Navigate to **IAM & Admin** > **IAM**.
2. Find `storage-access-sa` and **edit permissions**.
3. Remove the **Storage Object Viewer** role.
4. Click **Save**.

---

## Step 7: Verify Access Denied
1. SSH into the VM again.
2. Try downloading the file again:
   ```sh
   gsutil cp gs://my-storage-bucket-<your-unique-id>/sample.txt .
   ```
3. You should see a **PermissionDenied** error.

---

## Cleanup (Optional)
To avoid extra charges:
1. Delete the **VM Instance**.
2. Remove the **Service Account**.
3. Delete the **Storage Bucket**.

---

## Summary
You have successfully:
- Created a **Cloud Storage bucket** and uploaded a text file.
- Created a **service account** with permissions to access the file.
- Assigned the service account to a **VM instance**.
- Verified access and later revoked permissions to test restricted access.

# Lab 7: VPC Isolation and Peering in GCP

## Objective
In this lab, you will:
1. Create two **custom VPCs** with separate **subnets**.
2. Launch VMs in each VPC and verify **network isolation** (no connectivity).
3. Establish **VPC Peering** between the two VPCs.
4. Verify connectivity after peering by **pinging** between VMs.

---

## Prerequisites
- A Google Cloud account with billing enabled.
- Basic familiarity with **VPC networking** in GCP.

---

## Step 1: Create Custom VPCs
### Create VPC-1
1. Open the [Google Cloud Console](https://console.cloud.google.com/).
2. Navigate to **VPC network** > **VPC networks**.
3. Click **Create VPC network**.
4. Set:
   - **Name**: `vpc-1`
   - **Subnet creation mode**: Custom
   - **Subnet Name**: `subnet-1`
   - **Region**: Choose a region (e.g., `us-central1`)
   - **IP range**: `10.1.0.0/24`
5. Click **Create**.

### Create VPC-2
1. Click **Create VPC network** again.
2. Set:
   - **Name**: `vpc-2`
   - **Subnet Name**: `subnet-2`
   - **Region**: Same as VPC-1 (e.g., `us-central1`)
   - **IP range**: `10.2.0.0/24`
3. Click **Create**.

---

## Step 2: Create VM Instances in Each VPC
### Create VM in VPC-1
1. Navigate to **Compute Engine** > **VM Instances**.
2. Click **Create Instance** and configure:
   - **Name**: `vm-vpc-1`
   - **Region**: Same as `vpc-1`
   - **Network**: `vpc-1`
   - **Subnet**: `subnet-1`
3. Click **Create**.

### Create VM in VPC-2
1. Click **Create Instance** again.
2. Set:
   - **Name**: `vm-vpc-2`
   - **Region**: Same as `vpc-2`
   - **Network**: `vpc-2`
   - **Subnet**: `subnet-2`
3. Click **Create**.

---

## Step 3: Verify Network Isolation
1. SSH into `vm-vpc-1`.
2. Try to **ping** `vm-vpc-2`:
   ```sh
   ping 10.2.0.X  # Replace X with the actual IP of vm-vpc-2
   ```
3. The request should **time out**, showing that the VMs cannot communicate.

---

## Step 4: Create VPC Peering
1. Navigate to **VPC network** > **VPC network peering**.
2. Click **Create Peering Connection**.
3. Configure the first connection:
   - **Name**: `peering-vpc1-to-vpc2`
   - **VPC Network**: `vpc-1`
   - **Peer VPC Network**: Enter `vpc-2`
4. Click **Create**.
5. Repeat the steps to create a second peering connection:
   - **Name**: `peering-vpc2-to-vpc1`
   - **VPC Network**: `vpc-2`
   - **Peer VPC Network**: `vpc-1`
6. Click **Create**.

---

## Step 5: Verify Connectivity After Peering
1. SSH into `vm-vpc-1`.
2. Try to **ping** `vm-vpc-2` again:
   ```sh
   ping 10.2.0.X
   ```
3. This time, the ping should be **successful**, confirming that VPC peering allows connectivity.
4. Similarly, SSH into `vm-vpc-2` and ping `vm-vpc-1`.

---

## Cleanup (Optional)
To avoid extra charges:
1. Delete both **VM instances**.
2. Remove the **VPC Peering** connections.
3. Delete **VPC-1** and **VPC-2**.

---

## Summary
You have successfully:
- Created **two custom VPCs** with separate **subnets**.
- Launched **isolated VMs** that could not communicate initially.
- Configured **VPC Peering**.
- Verified that **connectivity** was established between the VPCs after peering.



# Lab 8: Connecting VPCs Using VPN in GCP

## Objective
In this lab, you will:
1. Create **two separate GCP projects** (simulating an on-prem data center and a cloud environment).
2. Set up **custom VPCs** with **subnets** in each project.
3. Deploy **VMs** in each VPC to test connectivity.
4. Configure a **Classic VPN** with **custom routing** between the two projects.
5. Verify connectivity after establishing the VPN tunnel.

---

## Prerequisites
- A Google Cloud account with billing enabled.
- Basic familiarity with **VPC networking** and **VPN** in GCP.

---

## Step 1: Create Two GCP Projects
1. Open the [Google Cloud Console](https://console.cloud.google.com/).
2. Navigate to **IAM & Admin** > **Manage Resources**.
3. Click **Create Project**.
4. Create two projects:
   - **Project 1** (Cloud VPC): `cloud-network`
   - **Project 2** (On-Prem Simulation): `onprem-network`
5. Switch between projects as needed during the lab.

---

## Step 2: Create Custom VPCs in Each Project
### In Project `cloud-network`:
1. Navigate to **VPC network** > **VPC networks**.
2. Click **Create VPC network** and set:
   - **Name**: `cloud-vpc`
   - **Subnet Name**: `cloud-subnet`
   - **Region**: Choose a preferred region (e.g., `us-central1`)
   - **IP range**: `10.1.0.0/24`
3. Click **Create**.

### In Project `onprem-network`:
1. Switch to `onprem-network` project.
2. Repeat the same steps above, using:
   - **VPC Name**: `onprem-vpc`
   - **Subnet Name**: `onprem-subnet`
   - **IP range**: `10.2.0.0/24`

---

## Step 3: Create VM Instances in Each VPC
### In `cloud-network` Project:
1. Navigate to **Compute Engine** > **VM Instances**.
2. Click **Create Instance** and set:
   - **Name**: `vm-cloud`
   - **Region**: Same as `cloud-vpc`
   - **Network**: `cloud-vpc`
   - **Subnet**: `cloud-subnet`
3. Click **Create**.

### In `onprem-network` Project:
1. Switch to `onprem-network` project.
2. Create another VM with:
   - **Name**: `vm-onprem`
   - **Network**: `onprem-vpc`
   - **Subnet**: `onprem-subnet`
3. Click **Create**.

---

## Step 4: Verify Network Isolation
1. SSH into `vm-cloud`.
2. Try to **ping** `vm-onprem`:
   ```sh
   ping 10.2.0.X  # Replace X with the actual IP of vm-onprem
   ```
3. The request should **time out**, confirming that VMs in different VPCs cannot communicate.

---

## Step 5: Configure a Classic VPN Tunnel with Custom Routing
### Create a VPN Gateway in `cloud-network` Project
1. Navigate to **Hybrid Connectivity** > **VPN**.
2. Click **Create VPN**.
3. Select **Classic VPN**.
4. Set:
   - **Name**: `vpn-cloud`
   - **Network**: `cloud-vpc`
   - **Region**: Same as `cloud-subnet`
5. Under **Tunnel 1**, set:
   - **Remote Peer IP**: Leave blank for now (to be filled later from `onprem-network` project).
   - **IKE Version**: `IKEv2`
   - **Shared Secret**: Choose a secure key.
6. Under **Routing Options**, select **Custom Routes** and enter:
   - **Destination Range**: `10.2.0.0/24` (Subnet of `onprem-network` project).
7. Click **Create**.

### Create a VPN Gateway in `onprem-network` Project
1. Switch to `onprem-network` project.
2. Navigate to **Hybrid Connectivity** > **VPN**.
3. Click **Create VPN**.
4. Select **Classic VPN**.
5. Set:
   - **Name**: `vpn-onprem`
   - **Network**: `onprem-vpc`
   - **Region**: Same as `onprem-subnet`
6. Under **Tunnel 1**, enter:
   - **Remote Peer IP**: Use the public IP of `vpn-cloud`.
   - **IKE Version**: `IKEv2`
   - **Shared Secret**: Use the same key as before.
7. Under **Routing Options**, select **Custom Routes** and enter:
   - **Destination Range**: `10.1.0.0/24` (Subnet of `cloud-network` project).
8. Click **Create**.

---

## Step 6: Verify Connectivity After VPN Setup
1. SSH into `vm-cloud`.
2. Try to **ping** `vm-onprem` again:
   ```sh
   ping 10.2.0.X  # Replace X with the actual IP of vm-onprem
   ```
3. This time, the ping should be **successful**, confirming that the VPN tunnel is working.
4. Similarly, SSH into `vm-onprem` and ping `vm-cloud`.

---

## Cleanup (Optional)
To avoid extra charges:
1. Delete both **VM instances**.
2. Delete **VPN tunnels** and **gateways** in both projects.
3. Delete **VPCs** and **projects** if no longer needed.

---

## Summary
You have successfully:
- Created **two GCP projects** to simulate **cloud and on-prem environments**.
- Configured **custom VPCs** in both projects.
- Verified initial **network isolation**.
- Established a **Classic VPN tunnel with custom routing** between the VPCs.
- Verified that the VPN tunnel enabled **cross-project connectivity**.


# Lab 9: Creating a .NET Project with Razor & MVC in VS Code and Dockerizing It

## Objective
In this lab, you will:
1. Create a **.NET MVC project** with Razor Pages in **VS Code** using the GUI.
2. Run the project using the **dotnet run** command in the terminal.
3. Create a **Dockerfile** for containerization.
4. Build and run the project as a Docker container.

---

## Prerequisites
- Install **.NET SDK** (Download from [here](https://dotnet.microsoft.com/en-us/download/dotnet)).
- Install **VS Code** and **C# Dev Kit Extension**.
- Install **Docker** and ensure the daemon is running.

---

## Step 1: Create a New .NET MVC Project Using VS Code GUI

1. Open **VS Code**.
2. Click on **File > Open Folder**, then choose a location for your project.
3. Open the **Command Palette** (`Ctrl+Shift+P` or `Cmd+Shift+P` on macOS) and search for **“.NET: New Project”**, then select it.
4. Choose **ASP.NET Core Web App (Model-View-Controller)**.
5. Select the latest available .NET version.
6. Enter a project name, e.g., `MyDotNetApp`, and select a directory.
7. Click **Create**.
8. Once the project is created, open the `Program.cs` file to verify the project structure.

---

## Step 2: Run the Application Locally

1. Open a terminal in **VS Code**.
2. Navigate to the project directory if not already there.
3. Run the following command:
   ```sh
   dotnet run
   ```
4. The application should launch, and you will see output indicating it is running on a URL such as `http://localhost:5000`.
5. Open the URL in a browser to see the default **Welcome Page**.

---

## Step 3: Create a Dockerfile

1. In **VS Code**, create a new file in the project root named **Dockerfile**.
2. Add the following content:
   ```dockerfile
   # Use official .NET SDK image as build environment
   FROM mcr.microsoft.com/dotnet/sdk AS build-env
   WORKDIR /app

   # Copy and restore dependencies
   COPY *.csproj ./
   RUN dotnet restore

   # Copy the rest of the application and build it
   COPY . ./
   RUN dotnet publish -c Release -o out

   # Use the runtime image for execution
   FROM mcr.microsoft.com/dotnet/aspnet
   WORKDIR /app
   COPY --from=build-env /app/out .
   ENTRYPOINT ["dotnet", "MyDotNetApp.dll"]
   ```

---

## Step 4: Build and Run the Docker Image

1. Open a terminal in the project root and build the Docker image:
   ```sh
   docker build -t my-dotnet-app .
   ```
2. Run the Docker container:
   ```sh
   docker run -p 8080:80 my-dotnet-app
   ```
3. Open `http://localhost:8080` in your browser to verify the application is running in Docker.

---

## Cleanup (Optional)
- Stop the running container using `CTRL + C`.
- Remove the Docker image if not needed:
  ```sh
  docker rmi my-dotnet-app
  ```

---

## Summary
You have successfully:
- Created a **.NET MVC project** with Razor Pages using the **VS Code GUI**.
- Run the application locally using the **dotnet run** command.
- Built a **Docker image** and executed the application inside a container.

# Lab 10: Pushing a .NET Project Image to Google Artifact Registry Using Cloud Shell

## Objective
In this lab, you will:
1. Download and install **Google Cloud SDK** (optional if using Cloud Shell).
2. Push the .NET project to **GitHub**.
3. Use **Google Cloud Console** to create an **Artifact Registry**.
4. Use **Cloud Shell** to clone the project.
5. Build and push the **Docker image** to **Artifact Registry**.

---

## Prerequisites
- A **Google Cloud Platform (GCP) account**.
- A **GitHub repository** for your project.
- **Cloud Shell** access in GCP.

---

## Step 1: Install Google Cloud SDK (Skip if Using Cloud Shell)

If you're running this outside Cloud Shell, install the **Google Cloud SDK**:
1. Download from [Google Cloud SDK](https://cloud.google.com/sdk/docs/install).
2. Follow the installation steps for your OS.
3. Authenticate with GCP:
   ```sh
   gcloud auth login
   ```
4. Set the active project:
   ```sh
   gcloud config set project YOUR_PROJECT_ID
   ```

---

## Step 2: Push the .NET Project to GitHub

1. Initialize a Git repository in your project folder:
   ```sh
   git init
   ```
2. Add and commit your files:
   ```sh
   git add .
   git commit -m "Initial commit"
   ```
3. Create a **GitHub repository** (if not already created).
4. Link your local repo to GitHub:
   ```sh
   git remote add origin https://github.com/YOUR_GITHUB_USERNAME/YOUR_REPO_NAME.git
   git branch -M main
   git push -u origin main
   ```

---

## Step 3: Create an Artifact Registry Using Google Cloud Console

1. Open **Google Cloud Console**.
2. Navigate to **Artifact Registry** from the left-hand menu.
3. Click **Create Repository**.
4. Provide the following details:
   - **Name**: `my-repo`
   - **Format**: `Docker`
   - **Region**: `us-central1` (or your preferred region)
   - **Repository Mode**: `Standard`
5. Click **Create**.

---

## Step 4: Clone the Project in Cloud Shell

1. Open **Google Cloud Console**.
2. Click on the **Cloud Shell** icon in the top right corner.
3. Clone the GitHub repository:
   ```sh
   git clone https://github.com/YOUR_GITHUB_USERNAME/YOUR_REPO_NAME.git
   ```
4. Navigate into the project directory:
   ```sh
   cd YOUR_REPO_NAME
   ```

---

## Step 5: Build and Push Docker Image to Artifact Registry

1. Configure **Docker authentication**:
   ```sh
   gcloud auth configure-docker us-central1-docker.pkg.dev
   ```
2. Build the Docker image:
   ```sh
   docker build -t us-central1-docker.pkg.dev/YOUR_PROJECT_ID/my-repo/my-dotnet-app .
   ```
3. Push the image to **Artifact Registry**:
   ```sh
   docker push us-central1-docker.pkg.dev/YOUR_PROJECT_ID/my-repo/my-dotnet-app
   ```

---

## Step 6: Verify the Image in Artifact Registry

1. Go to **Google Cloud Console**.
2. Navigate to **Artifact Registry > Repositories**.
3. Click on **my-repo** and check if the image is listed.

---

## Summary
You have successfully:
- Installed **Google Cloud SDK** (if needed).
- Pushed the project to **GitHub**.
- Used **Cloud Shell** to clone and work with the project.
- Created an **Artifact Registry** using the **Google Cloud Console**.
- Built and pushed a **Docker image** to **Artifact Registry**.

# Lab 11: Deploying a .NET Application on GKE Autopilot Using Artifact Registry

## Objective
In this lab, you will:
1. Create a **GKE Autopilot Cluster** using **Google Cloud Console**.
2. Deploy a workload using a **Docker image** stored in **Artifact Registry**.
3. Expose the application using a **LoadBalancer Service**.

---

## Prerequisites
- A **Google Cloud Platform (GCP) account**.
- A **Docker image** already pushed to **Artifact Registry**.
- **Google Kubernetes Engine (GKE) API enabled**.

---

## Step 1: Create a GKE Autopilot Cluster

### Using Console
1. Open **Google Cloud Console**.
2. Navigate to **Kubernetes Engine > Clusters**.
3. Click **Create** and choose **Autopilot cluster**.
4. Provide the following details:
   - **Name**: `gke-autopilot-cluster`
   - **Region**: Select your preferred region (e.g., `us-central1`)
   - Leave other settings as default.
5. Click **Create** and wait for the cluster to be provisioned.

### Using Cloud Shell
1. Open **Cloud Shell**.
2. Run the following command to create the Autopilot cluster:
   ```sh
   gcloud container clusters create-auto gke-autopilot-cluster --region=us-central1
   ```
3. Wait for the cluster to be created.

---

## Step 2: Connect to the GKE Cluster

### Using Console
1. Go to **Kubernetes Engine > Clusters** in **Google Cloud Console**.
2. Click on **gke-autopilot-cluster**.
3. Click **Connect**, then **Run in Cloud Shell** to authenticate and configure `kubectl`.

### Using Cloud Shell
1. Authenticate and configure `kubectl` manually:
   ```sh
   gcloud container clusters get-credentials gke-autopilot-cluster --region=us-central1
   ```
2. Verify the cluster connection:
   ```sh
   kubectl get nodes
   ```

---

## Step 3: Deploy the Application

### Using Console
1. Go to **Kubernetes Engine > Workloads**.
2. Click **Deploy**.
3. Provide the following details:
   - **Name**: `my-dotnet-app`
   - **Container Image**: `us-central1-docker.pkg.dev/YOUR_PROJECT_ID/my-repo/my-dotnet-app:latest`
   - **Port**: `80`
4. Click **Deploy**.

### Using Cloud Shell
1. Create a Kubernetes deployment manifest (`deployment.yaml`) using the Artifact Registry image:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: my-dotnet-app
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: my-dotnet-app
     template:
       metadata:
         labels:
           app: my-dotnet-app
       spec:
         containers:
         - name: my-dotnet-app
           image: us-central1-docker.pkg.dev/YOUR_PROJECT_ID/my-repo/my-dotnet-app:latest
           ports:
           - containerPort: 8080
   ```
2. Apply the deployment:
   ```sh
   kubectl apply -f deployment.yaml
   ```
3. Verify the deployment:
   ```sh
   kubectl get pods
   ```

---

## Step 4: Expose the Application

### Using Console
1. Go to **Kubernetes Engine > Services & Ingress**.
2. Click **Expose**.
3. Select **my-dotnet-app** as the workload.
4. Choose **LoadBalancer** as the service type.
5. Set the port to **80** and click **Create**.

### Using Cloud Shell
1. Create a service manifest (`service.yaml`) to expose the application:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: my-dotnet-app-service
   spec:
     type: LoadBalancer
     selector:
       app: my-dotnet-app
     ports:
       - protocol: TCP
         port: 8080
         targetPort: 8080
   ```
2. Apply the service:
   ```sh
   kubectl apply -f service.yaml
   ```
3. Check the external IP address:
   ```sh
   kubectl get services
   ```
4. Once the **EXTERNAL-IP** is assigned, open it in a browser to view the application.

---

## Summary
You have successfully:
- Created a **GKE Autopilot Cluster**.
- Deployed a **.NET application** using an image from **Artifact Registry**.
- Exposed the application using a **LoadBalancer Service**.

# Lab 12: Setting Up a Cloud Build CI/CD Pipeline for GKE Deployment

## Objective

In this lab, you will:

1. Link a **GitHub repository** to **Google Cloud Build** using a **2nd Gen host connection**.
2. Create a **Cloud Build trigger** to deploy a **.NET application** from GitHub to GKE Autopilot.
3. Use a **cloudbuild.yaml** file to automate the build and deployment process.

---

## Prerequisites

- A **Google Cloud Platform (GCP) account**.
- A **GitHub repository** containing your .NET application with a **Dockerfile**.
- An existing **Artifact Registry** for storing container images.
- A **GKE Autopilot Cluster** created in your project.
- **Cloud Build API** enabled in GCP.

---

## Step 1: Link GitHub Repository to Cloud Build

### Using Google Cloud Console

1. Open **Google Cloud Console**.
2. Navigate to **Cloud Build > Repositories**.
3. Select GitHub (2nd Gen). 
4. Click \*\*Create Host Connection \*\*.
5. Select region and give your GitHub account name. 
6. It will open a new browser page in there click continue, authorize and install cloud build token in your GitHub repo.
7. Click **Link Repository** and authorize access to your GitHub account.
8. Choose the **GitHub repository** you want to link and click **Next**.

---

## Step 2: Create a Cloud Build Trigger

1. In the **Trigger Configuration** page:
   - **Name**: `gke-deployment-trigger`
   - **Event**: `Push to a branch`
   - **Branch**: `main`
   - **Configuration**: `cloudbuild.yaml`
2. Click **Create Trigger**.

---

## Step 3: Configure Cloud Build YAML File

1. In your **GitHub repository**, create a file named **cloudbuild.yaml** with the following content:

   ```yaml
   steps:
     # Step 1: Build the container image using Cloud Build
     - name: 'gcr.io/cloud-builders/docker'
       args:
         - 'build'
         - '-t'
         - 'asia-south2-docker.pkg.dev/YOUR_PROJECT_ID/gccpdemo/gcpdemo:$SHORT_SHA'
         - '.'

     # Step 2: Push the image to Artifact Registry
     - name: 'gcr.io/cloud-builders/docker'
       args:
         - 'push'
         - 'asia-south2-docker.pkg.dev/YOUR_PROJECT_ID/gccpdemo/gcpdemo:$SHORT_SHA'

     # Step 3: Authenticate with GKE
     - name: 'gcr.io/cloud-builders/gcloud'
       args:
         - 'container'
         - 'clusters'
         - 'get-credentials'
         - 'autopilot-cluster-1'
         - '--region=us-central1'

     # Step 4: Update the Kubernetes deployment with the new image
     - name: 'gcr.io/cloud-builders/kubectl'
       args:
         - 'set'
         - 'image'
         - 'deployment/deployment-1'
         - 'gcpdemo-sha256-1=asia-south2-docker.pkg.dev/YOUR_PROJECT_ID/gccpdemo/gcpdemo:$SHORT_SHA'
       env:
         - 'CLOUDSDK_COMPUTE_REGION=us-central1'
         - 'CLOUDSDK_CONTAINER_CLUSTER=autopilot-cluster-1'

   images:
     - 'asia-south2-docker.pkg.dev/YOUR_PROJECT_ID/gccpdemo/gcpdemo:$SHORT_SHA'

   options:
     logging: CLOUD_LOGGING_ONLY  # Store logs in Cloud Logging
   ```

2. Commit and push this file to the **main** branch in GitHub.

---

## Step 4: Verify Automatic Triggering of Cloud Build

1. Open your **GitHub repository**.
2. Navigate to `Pages/Index.cshtml` in your project.
3. Make a small change (e.g., update the welcome message inside the `<h1>` tag).
4. Commit and push the changes to the **main** branch.
5. Go to **Google Cloud Console > Cloud Build > Dashboard**.
6. Check if a new build has been automatically triggered.
7. Once completed, verify the deployment by checking the **GKE Workloads**.

---

## Step 5: Verify the Deployment

### Using Console

1. Navigate to **Kubernetes Engine > Workloads**.
2. Check that `deployment-1` is running successfully.
3. Go to **Services & Ingress** and find the **LoadBalancer**.
4. Copy the **External IP** and open it in a browser.

### Using Cloud Shell

1. Run the following command to get the **external IP**:
   ```sh
   kubectl get services
   ```
2. Copy the **EXTERNAL-IP** and access it in a browser.

---

## Summary

You have successfully:

- Linked **GitHub** to **Cloud Build** using **2nd Gen host connection**.
- Created a **Cloud Build trigger** to automate the deployment process.
- Configured **cloudbuild.yaml** to build, push, and deploy the application to GKE.
- Verified that Cloud Build automatically triggers on GitHub commits.
- Confirmed the deployment updates in GKE.

# Lab 13: Deploying a .NET Application to App Engine and Managing Traffic

## Objective

In this lab, you will:
1. Deploy a **.NET application** to **Google App Engine** using **VS Code or Cloud Shell**.
2. Modify the application and redeploy it.
3. Manage traffic migration and split traffic between two versions.

---

## Prerequisites

- A **Google Cloud Platform (GCP) account**.
- **Google Cloud SDK** installed (or use **Cloud Shell**).
- A **GitHub repository** cloned locally with your .NET project.
- **App Engine API enabled** in GCP.

---

## Step 1: Set Up App Engine

### Using Console
1. Open **Google Cloud Console**.
2. Navigate to **App Engine > Dashboard**.
3. Click **Create Application**.
4. Select a **Region** (e.g., `us-central1`).
5. Choose the **.NET runtime**.
6. Click **Create**.

---

## Step 2: Prepare the .NET Application for Deployment

### Using VS Code or Cloud Shell
1. Open **VS Code** or **Cloud Shell**.
2. Clone your GitHub repository if not already done:
   ```sh
   git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git
   cd YOUR_REPO
   ```
3. Ensure you have an `app.yaml` file in the project root with the following content:
   ```yaml
   runtime: dotnet
   instance_class: F1
   automatic_scaling:
     target_cpu_utilization: 0.65
     min_instances: 1
     max_instances: 3
   ```

---

## Step 3: Deploy the Application

### Using Cloud Shell or VS Code Terminal
1. Authenticate with GCP:
   ```sh
   gcloud auth login
   gcloud config set project YOUR_PROJECT_ID
   ```
2. Deploy the application:
   ```sh
   gcloud app deploy
   ```
3. Confirm deployment by typing `Y` when prompted.
4. Once deployed, check the application by running:
   ```sh
   gcloud app browse
   ```

---

## Step 4: Modify and Redeploy the Application

1. Open **`Pages/Index.cshtml`** in VS Code.
2. Modify the content, e.g., update the `<h1>` tag:
   ```html
   <h1>Welcome to Version 2 of My App</h1>
   ```
3. Save the file and commit the changes:
   ```sh
   git add Pages/Index.cshtml
   git commit -m "Updated Index page"
   git push origin main
   ```
4. Deploy the updated version:
   ```sh
   gcloud app deploy
   ```
5. Open the application again:
   ```sh
   gcloud app browse
   ```

---

## Step 5: Manage Traffic Migration & Split Traffic

### Migrate 100% Traffic to New Version
1. Navigate to **Google Cloud Console > App Engine > Versions**.
2. Click **Migrate Traffic** to shift all traffic to the new version.

### Split Traffic Between Versions
1. Go to **App Engine > Versions**.
2. Select both versions.
3. Click **Split Traffic**.
4. Choose **Weighted Splitting** and distribute traffic (e.g., `50%-50%`).
5. Click **Save**.

---

## Summary

You have successfully:
- Deployed a .NET application to **App Engine**.
- Modified and redeployed the application.
- Managed **traffic migration** and **traffic splitting** between versions.




















