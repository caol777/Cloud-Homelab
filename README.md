
# Cloud Homelab Project  

This project involves setting up a cloud homelab using AWS as the host and Terraform to provision the infrastructure. The lab includes three machines: Kali Linux, Ubuntu Security Tools, and Windows Server 2022, enabling a practical environment for cloud and cybersecurity exploration.

---

## Network-Topology
![image](https://github.com/user-attachments/assets/431d73be-0b0a-4a5f-bcf8-07ef342eff0c)


### Overview
This project involves setting up a cloud-based homelab using AWS as the host and Terraform for infrastructure automation. The environment includes three virtual machines:
- **Kali Linux**
- **Ubuntu**
- **Windows Server 2022**

The project utilizes Ubuntu through WSL for configuration.

---

## Steps to Set Up

### 1. Configure AWS IAM User
1. Go to **IAM Dashboard** > **Users** > **Add User**.
2. Assign a name to the user and attach the following policies:
   - `VPCFullAccess` (to create the VPC).
   - `EC2FullAccess` (to create and manage EC2 instances).
3. After creating the user:
   - Navigate to **Security Credentials** and create an access key.
   - Copy both the **Access Key** and **Secret Access Key**. The **Secret Access Key** is displayed only once—store it securely.

### 2. Install AWS CLI
Run the following commands in the WSL Ubuntu terminal:
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

### 3. Configure Terraform
1. Ensure the AWS configuration files (`.aws/credentials` and `.aws/config`) are in place.
2. Clone the homelab repository:
   ```bash
   git clone https://github.com/caol777/Cloud-Homelab.git
   cd Cloud-Homelab
   ```
3. Generate an RSA key pair in AWS (ensure the format is `.pem`) and store it in the `.ssh` directory.

### 4. Deploy Infrastructure with Terraform
1. Initialize Terraform:
   ```bash
   terraform init
   ```
2. Plan the infrastructure:
   ```bash
   terraform plan
   ```
3. Apply the configuration:
   ```bash
   terraform apply -var="aws-RSA-key"
   ```
   - Replace `"aws-RSA-key"` with the name of the RSA key pair.
   - Ensure the region is set to `us-east-2`.
4. If using marketplace images (e.g., Kali, Ubuntu), accept their terms before deployment.

---

## Connecting to the Instances

### Kali Linux
Follow provided setup instructions to access the machine.

### Ubuntu
1. Use the instance's IP address (from Terraform output) in a browser.
2. Log in using the EC2 instance ID.
3. Change the VNC password if needed.

### Windows Server 2022
1. Go to the EC2 Dashboard, right-click the Windows instance, and select **Connect**.
2. Under **RDP Client**, retrieve the password:
   - Upload the private key stored in `.ssh`.
   - Use the default username `Administrator`.
3. Use an RDP client to connect to the machine.

---

## Configuring Splunk

### Ubuntu (Security Box)
1. Download and install Splunk Enterprise:
   ```bash
   sudo dpkg -i splunk-deb
   cd /opt/splunk/bin
   sudo ./splunk start
   ```
2. Log in using admin credentials.
3. Navigate to **Settings** > **Forwarding and Receiving** > **Configure Receiving**:
   - Add a new port (e.g., `9997`).

### Windows (Splunk Universal Forwarder)
1. Download and install the Universal Forwarder.
2. During installation, customize options as required.
3. On the Ubuntu Splunk instance:
   - Create a new index: **Settings** > **Indexes** > **New Index** (e.g., `win-security`).
4. On the Windows instance:
   - Navigate to `C:\Program Files\SplunkUniversalForwarder\etc\system\local`.
   - Copy `outputs.conf` and rename it to `inputs.conf`.
   - Edit `inputs.conf` to include:
     ```
     [WinEventLog://Security]
     index = win-security
     disabled = 0
     ```
5. Restart Splunk:
   ```bash
   cd forwarder/bin
   splunk restart
   ```
6. Verify data in Splunk by searching for `index="win-security"`.

---

## Configuring Nessus on Ubuntu
1. Install Nessus:
   ```bash
   dpkg -i "Nessus-[version number]-debian6_amd64.deb"
   sudo systemctl start nessusd.service
   ```
2. Save the activation code during installation.

---

## Notes
- Ensure SSH keys and AWS credentials are securely stored.
- Address any deployment issues by reviewing Terraform logs or AWS Marketplace terms.
