
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

![image](https://github.com/user-attachments/assets/7f1c73b7-16ec-4651-9ab4-08cb052b318d)

   
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
2. Ensure that your config file looks like this ![image](https://github.com/user-attachments/assets/a08535f5-cedb-40e3-885e-69402b75b7b7)
3. Ensure that your credentials file has your access keys from the IAM user. ![image](https://github.com/user-attachments/assets/a044c8d6-0e7c-4422-ad77-4f767b51833d)

4. Clone the homelab repository:
   ```bash
   git clone https://github.com/caol777/Cloud-Homelab.git
   cd Cloud-Homelab
   ```
5. Generate an RSA key pair in AWS (ensure the format is `.pem`) and store it in the `.ssh` directory.

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
1.    ssh to the kali machine ![image](https://github.com/user-attachments/assets/ea03bda1-16e7-4816-bb50-ba5191eed0e8)

2.    Do nano rdp.sh and add the following lines ![image](https://github.com/user-attachments/assets/8fc1ef4d-2c67-414a-81f6-97828913d19b)

3.    Make sure to make the script executable and run the script ![image](https://github.com/user-attachments/assets/2d702608-6d21-4002-90c3-c274f5f58a2e)

4.    Change the password of the machine, enable xrdp and check that its running  ![image](https://github.com/user-attachments/assets/8ecea866-cb89-4d4a-b842-10728c0bbb0b)

5.    Now you can connect to the kali machine using rdp on your windows client with the credentials kali:kali 

### Ubuntu
1. Use the instance's IP address (from Terraform output) in a browser.
2. Log in using the EC2 instance ID.
3. Change the VNC password if needed.

### Windows Server 2022
1. Go to the EC2 Dashboard, right-click the Windows instance, and select **Connect**.
2. Under **RDP Client**, retrieve the password:
   - Upload the private key stored in `.ssh`.
   - Use the default username `Administrator`.
   - Copy the password
3. Use an RDP client to connect to the machine.
4. Make sure to disable the firewall and windows defender to make sure our splunk configuration will work. ![image](https://github.com/user-attachments/assets/f26c94a0-c852-4d4b-b366-3dcee35ac64f)

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
3. 
![image](https://github.com/user-attachments/assets/ad0d09f1-024e-4f46-8b17-e5b3fdf09273)
4. 
![image](https://github.com/user-attachments/assets/570d7153-e43e-41d3-a746-dd9d69e7fb72)
5. 
![image](https://github.com/user-attachments/assets/312aff24-2f86-43cf-98fe-28ccd86125da)
6. For this step make sure to grab the IP address from the Ubuntu security box using ip a in the terminal and use the default port 9997. ![image](https://github.com/user-attachments/assets/7bb394ea-d1e6-4ca0-ae8e-0d39230d97c4)


7. On the Ubuntu Splunk instance:
   - Create a new index: **Settings** > **Indexes** > **New Index** (e.g., `win-security`).
8. On the Windows instance:
   - Navigate to `C:\Program Files\SplunkUniversalForwarder\etc\system\local`.
   - Copy `outputs.conf` and rename it to `inputs.conf`.
   - Edit `inputs.conf` to include:
     ```
     [WinEventLog://Security]
     index = win-security
     disabled = 0
     ```
9. Restart Splunk:
   ```bash
   cd C:\Program Files\SplunkUniversalforwarder\bin
   splunk.exe restart
   ```
10. Verify data in Splunk by searching for `index="win-security"`.
11. 
![image](https://github.com/user-attachments/assets/ed75a9e5-c99a-4778-a7f0-3baf2dd4e388)


---

## Configuring Nessus on Ubuntu
1. Install Nessus:
   ```bash
   dpkg -i "Nessus-[version number]-debian6_amd64.deb"
   sudo systemctl start nessusd.service
   ```
2. Save the activation code during installation.
3. 
![image](https://github.com/user-attachments/assets/0b46c22e-afc8-4759-988a-13a9441deb29)


---

## Notes
- Ensure SSH keys and AWS credentials are securely stored.
- Address any deployment issues by reviewing Terraform logs or AWS Marketplace terms.
