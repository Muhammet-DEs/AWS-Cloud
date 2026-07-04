# ☕ Café Dynamic Website — Amazon EC2 Challenge Lab

![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-Secrets_Manager-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-AMI-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-Challenge-blueviolet?style=for-the-badge)
![Duration](https://img.shields.io/badge/Duration-60%20min-informational?style=for-the-badge)

> Deploying a **dynamic PHP/MySQL café website** on Amazon EC2, securing credentials with **AWS Secrets Manager**, and replicating the environment to a second AWS Region using a custom **AMI** for dev/prod separation.

---

## 📖 Scenario

The café's static S3 website was a great start — but customers began asking if they could **place orders online**. Sofía takes on the challenge: she configures an EC2 instance, installs a full LAMP stack, deploys a PHP web application backed by MySQL, and protects database credentials with **AWS Secrets Manager**. She then captures an AMI and launches a production copy in a second AWS Region.

---

## 🎯 Objectives

- 🖥️ Connect to a **VS Code IDE** hosted on an EC2 instance
- ⚙️ Configure a **LAMP stack** (Linux, Apache, MariaDB, PHP)
- 🔐 Store and retrieve app secrets using **AWS Secrets Manager**
- 🌐 Deploy and test a **dynamic café ordering web application**
- 📸 Create a custom **Amazon Machine Image (AMI)**
- 🌍 Launch a **production copy** of the app in a second AWS Region (us-west-2)

---

## 🏗️ Architecture

```
──────────────────────────────────────────────────────────────┘
```

---

## ⚙️ Challenge 1 — Configuring the EC2 Instance

### Task 1: Analyzing the Existing EC2 Instance

1. Opened the **Amazon EC2 console** from the AWS Management Console
2. Chose **Instances** and located the running instance named **Lab IDE**
3. Reviewed the instance details to answer the lab questions:

| Question | Finding |
|---|---|
| Is the instance in a public subnet? | ✅ Yes |
| Does it have a Public IPv4 address? | ✅ Yes |
| Open inbound TCP ports? | Port 80 (HTTP) and Port 8000 |
| IAM role associated? | ✅ Yes — grants access to Secrets Manager |

📸 *Screenshot placeholder — EC2 Lab IDE instance details*
<!-- ![Task 1 Screenshot](./screenshots/task1-instance.png) -->

---

### Task 2: Connecting to the VS Code IDE

1. From the lab instructions, chose **AWS Details**
2. Copied the values for **LabIDEURL** and **LabIDEPassword** to a text editor
3. Opened **LabIDEURL** in a new browser tab
4. Entered **LabIDEPassword** on the welcome prompt and chose **Submit**
5. Confirmed the VS Code IDE loaded with:
   - 📁 File browser (left panel) — shows `/home/ec2-user/environment`
   - 🖊️ File editor (upper-right panel)
   - 💻 Bash terminal (bottom-right panel)

📸 *Screenshot placeholder — VS Code IDE open in browser*
<!-- ![Task 2 Screenshot](./screenshots/task2-vscode.png) -->

---

### Task 3: Configuring the LAMP Stack and Web Server

1. Checked the OS version in the bash terminal:
   ```bash
   cat /proc/version
   ```
   > Output confirmed: Amazon Linux (Red Hat 7 compatible)

2. Configured Apache to run on **port 8000** (since port 80 is used by the IDE), then started and enabled it:
   ```bash
   sudo sed -i 's/Listen 80/Listen 8000/g' /etc/httpd/conf/httpd.conf
   sudo systemctl start httpd
   sudo systemctl enable httpd
   sudo service httpd status
   php --version
   ```

3. Installed **MariaDB** (MySQL-compatible database) and set it to auto-start:
   ```bash
   sudo dnf install -y mariadb105-server
   sudo systemctl start mariadb
   sudo systemctl enable mariadb
   sudo mariadb --version
   sudo service mariadb status
   ```
   > 💡 If the terminal hangs after status, press `Q` to exit.

4. Configured file permissions so the VS Code IDE can edit web server files:
   ```bash
   ln -s /var/www/ /home/ec2-user/environment
   sudo chown ec2-user:ec2-user /var/www/html
   ```
   - First command: creates a symlink from the IDE workspace to `/var/www`
   - Second command: grants `ec2-user` ownership of the html directory

5. Created a test webpage via the file browser:
   - Navigated to `CafeWebServer > www > html`
   - Created a new file named `index.html`
   - Added the content:
     ```html
     <html>Hello from the café web server!</html>
     ```
   - Saved the file

6. Verified the web server was accessible from the internet by opening:
   ```
   http://<public-ip>:8000/
   ```
   > ✅ The test page loaded successfully

📸 *Screenshot placeholder — test page loaded in browser & httpd running status*
<!-- ![Task 3 Screenshot](./screenshots/task3-lamp.png) -->

---

## 🌐 Challenge 2 — Installing the Dynamic Web Application

### Task 4: Installing the Café Application

1. Downloaded and extracted all application files:
   ```bash
   cd ~/environment

   wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/.../setup.zip
   unzip setup.zip

   wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/.../db.zip
   unzip db.zip

   wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/.../cafe.zip
   unzip cafe.zip -d /var/www/html/
   ```

2. Installed the **AWS SDK for PHP** inside the application:
   ```bash
   cd /var/www/html/cafe/
   wget https://docs.aws.amazon.com/aws-sdk-php/v3/download/aws.zip
   wget https://docs.aws.amazon.com/aws-sdk-php/v3/download/aws.phar
   unzip aws -d /var/www/html/cafe/
   chmod -R +r /var/www/html/cafe/
   ```

3. Reviewed `index.php` — noted it calls `getAppParameters.php` which invokes the AWS SDK to fetch secrets from **Secrets Manager**

4. Ran the setup script to push the **7 application secrets** into AWS Secrets Manager:
   ```bash
   cd ~/environment/setup/
   ./set-app-parameters.sh
   ```

5. Opened **AWS Secrets Manager** in the console and confirmed **7 secrets** were created under `/cafe/` prefix (e.g. `/cafe/dbPassword`, `/cafe/dbUser`, etc.)

6. Copied the value of `/cafe/dbPassword` (via **Retrieve secret value**) for use in the next step

7. Configured the **MySQL database** for the café application:
   ```bash
   cd ../db/
   ./set-root-password.sh
   ./create-db.sh
   ```

8. Connected to the database and verified the tables:
   ```bash
   mysql -u admin -p
   ```
   ```sql
   show databases;
   use cafe_db;
   show tables;
   select * from product;
   exit;
   ```

9. Configured the **PHP timezone** and restarted the web server:
   ```bash
   sudo sed -i "2i date.timezone = \"America/New_York\" " /etc/php.ini
   sudo service httpd restart
   ```

10. Tested the café website in the browser:
    ```
    http://<public-ip>:8000/cafe
    ```
    > ✅ Full café menu page loaded with all products displayed

📸 *Screenshot placeholder — café application running in browser with menu*
<!-- ![Task 4 Screenshot](./screenshots/task4-cafeapp.png) -->

---

### Task 5: Testing the Web Application

1. Opened the café website: `http://<public-ip>:8000/cafe`
2. Chose **Menu** from the navigation
3. Selected at least one item and submitted an order using the **Submit Order** button
4. Returned to the menu and placed a second order
5. Navigated to the **Order History** page and confirmed all orders were recorded correctly

📸 *Screenshot placeholder — Order History page showing submitted orders*
<!-- ![Task 5 Screenshot](./screenshots/task5-orders.png) -->

---

## 🌍 Challenge 3 — Dev & Prod Environments in Different Regions

### Task 6: Creating an AMI and Launching a Production Instance

#### 6.1 — Preparing the Instance and Creating the AMI

1. Set a static internal hostname and generated an SSH key pair on the instance:
   ```bash
   sudo hostname cafeserver
   ssh-keygen -t rsa -f ~/.ssh/id_rsa
   # Press Enter twice when prompted for passphrase
   ```

2. Added the new public key to the authorized keys:
   ```bash
   cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
   ```

3. In the **EC2 console**, selected the **Lab IDE** instance
4. Chose **Actions → Images and templates → Create image**
5. Set **Image name** to `CafeServer`
6. Chose **Create Image**
7. Navigated to **AMIs** (under Images in the left panel)
8. Waited until the AMI status changed to **Available** (~2 minutes)

| AMI Setting | Value |
|---|---|
| Image name | `CafeServer` |
| Status | `Available` |
| Source Region | `us-east-1` |

📸 *Screenshot placeholder — AMI CafeServer status Available in EC2 console*
<!-- ![Task 6a Screenshot](./screenshots/task6a-ami.png) -->

#### 6.2 — Deploying the Production Instance in us-west-2

1. Copied the **CafeServer AMI** to the **Oregon (us-west-2)** Region
2. Switched to the **us-west-2** Region in the AWS Management Console
3. Launched a new EC2 instance from the copied AMI with the following configuration:

| Setting | Value |
|---|---|
| Name | `ProdCafeServer` |
| Instance type | `t2.small` |
| Key pair | Proceed without a key pair |
| VPC | `Lab VPC Region 2` |
| Subnet | `Public Subnet` |
| Security group name | `cafeSG` |
| Inbound TCP port 22 | Open to anywhere |
| Inbound TCP port 8000 | Open to anywhere |
| IAM instance profile | `CafeRole` |

4. Chose **Launch instance** and waited for a **Public IPv4 DNS** to be assigned
5. Copied the **Public IPv4 DNS** value of `ProdCafeServer`

#### 6.3 — Creating Secrets Manager Secrets in us-west-2

1. Returned to the VS Code IDE in us-east-1
2. Opened `CafeWebServer/setup/set-app-parameters.sh` in the editor
3. Updated **line ~15** to target Oregon:
   ```bash
   region="us-west-2"
   ```
4. Updated **line ~21** with the production server DNS:
   ```bash
   publicDNS="<public-dns-of-ProdCafeServer-instance>"
   ```
5. Saved the file and ran the script again:
   ```bash
   cd ~/environment/setup/
   ./set-app-parameters.sh
   ```
   > ✅ JSON output confirmed the 7 secrets were created in the us-west-2 Secrets Manager

📸 *Screenshot placeholder — ProdCafeServer launched & Secrets Manager in us-west-2*
<!-- ![Task 6b Screenshot](./screenshots/task6b-prod.png) -->

---

### Task 7: Verifying the Production Café Instance

1. Returned to the **EC2 console in Oregon (us-west-2)**
2. Confirmed **ProdCafeServer** was in **Running** state
3. Copied the **Public IPv4 address** and opened it in a browser:
   ```
   http://<public-ip>/
   ```
   > ✅ "Hello from the café web server!" message displayed

4. Loaded the full café app:
   ```
   http://<public-ip>:8000/cafe/
   ```
   > ✅ Full café website loaded correctly

5. Opened the **Menu** page — all items displayed
6. Placed a test order to confirm end-to-end functionality works in the production environment

> 💡 If SSH troubleshooting is needed on the production instance, connect from the dev IDE:
> ```bash
> ssh -i ~/.ssh/id_rsa ec2-user@<public-ip-of-ProdCafeServer>
> ```

📸 *Screenshot placeholder — Production café website live in Oregon Region*
<!-- ![Task 7 Screenshot](./screenshots/task7-prod-live.png) -->

---

## 🧠 Key Concepts Demonstrated

| Concept | Detail |
|---|---|
| 🖥️ LAMP Stack | Apache (port 8000) + MariaDB + PHP on Amazon Linux |
| 🔐 Secrets Manager | 7 app secrets stored and retrieved via AWS SDK for PHP |
| 📸 AMI | Full system snapshot used to replicate the environment |
| 🌍 Multi-Region Deployment | Dev in us-east-1, Prod in us-west-2 |
| 🔑 IAM Role | `CafeRole` grants EC2 access to Secrets Manager |
| 🛡️ Security Group | Ports 22 and 8000 open for SSH and web access |

---

## 📋 Lab Questions Summary

| # | Question | Answer |
|---|---|---|
| 1 | Is the Lab IDE instance in a public subnet? | ✅ Yes |
| 2 | Does it have a Public IPv4 address? | ✅ Yes |
| 3 | Open inbound TCP ports? | Port 80 & Port 8000 |
| 4 | IAM role associated? | ✅ Yes — CafeRole |
| 5 | Will instance reboot when creating an AMI? | ✅ Yes (by default) |
| 6 | Ways to modify root volume when creating AMI? | Size, type, encryption, delete-on-termination |
| 7 | Can you add more volumes to the AMI? | ✅ Yes |

---

## 🔁 Dev vs Prod — Environment Summary

| Property | Development (us-east-1) | Production (us-west-2) |
|---|---|---|
| Instance Name | `Lab IDE` | `ProdCafeServer` |
| Purpose | Feature testing & development | Live customer-facing orders |
| Source | Original setup | Launched from `CafeServer` AMI |
| Secrets Manager | us-east-1 region | us-west-2 region |
| DR Benefit | — | Separate Region for resilience |

---

## 🏁 Result

A fully functional dynamic café website was deployed on Amazon EC2 using a LAMP stack, with database credentials securely managed by AWS Secrets Manager. A custom AMI was captured and used to launch an identical production environment in a second AWS Region — enabling safe development workflows and cross-region disaster recovery.

---

<p align="center">
  🧾 <b>AWS Hands-On Lab Series</b> — Amazon EC2 Dynamic Website & Multi-Region Deployment
</p>
