# 🚀 Full Stack DevOps Pipeline: Jenkins, SonarQube, & Docker on AWS
---

An automated DevOps pipeline for containerized web applications using Jenkins, SonarQube, and Docker.

## **📌 Project Overview**

This project demonstrates a complete CI/CD lifecycle across a distributed AWS infrastructure. It automates code integration, static analysis for quality gates, and remote deployment to a Docker environment.

### **🏗 Architecture**

- **Jenkins (Build Server):** Orchestrates the pipeline and handles automation triggers.
- **SonarQube (Quality Server):** Performs Static Application Security Testing (SAST) and code quality analysis.
- **Docker (Deployment Server):** Hosts the containerized application.
- **GitHub:** Source Control Management with automated Webhooks.

---

## **🛠 Prerequisites**

- An active **AWS Account**.
- Basic knowledge of **Linux/Bash**.
- **GitHub** repository containing your web project.

## **🚀 Implementation Steps**

### **Step 1: Setup Code Project and Git Repository**

- Prepare your application code/template.
- Create a new repository on GitHub.
- Push your local code to the GitHub repository.

---

### **Step 2: Provision AWS EC2 Instances**

- Log in to your AWS Management Console.
- Launch **three (3) EC2 instances** using **Ubuntu** as the Operating System.
- **Naming Convention:**
    1. `Jenkins-Server`
    2. `SonarQube-Server`
    3. `Docker-Server`

---

### **Step 3: Setup Jenkins on the Instance**

- Access the `Jenkins-Server` via your local terminal using SSH.
- Update the system packages: `sudo apt update`.
- **Install Java:** Jenkins requires Java to run. (e.g., `sudo apt install openjdk-17-jre`).
- **Install Jenkins:** Go to the [official Jenkins website](https://pkg.jenkins.io/debian-stable/), follow the Debian/Ubuntu instructions to add the repository and install.
- **Verify Installation:** Check if Jenkins is running: `systemctl status jenkins`.

---

### **Step 4: Configure AWS Security Group for Jenkins**

- In the AWS Console, go to the **Security** section of your Jenkins instance.
- Click on **Edit Inbound Rules**.
- **Add Rule:** Select **Custom TCP**, Port Range **8080**, Source **Anywhere (0.0.0.0/0)**.
- Save the rule. (Jenkins only communicates on port 8080 by default).

---

### **Step 5: Initial Jenkins Web Setup**

- Open your browser and enter: `http://<Jenkins_Public_IP>:8080`.
- **Unlock Jenkins:** Copy the path shown on the screen (e.g., `/var/lib/jenkins/secrets/initialAdminPassword`).
- In your terminal, run: `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`.
- Copy the password, paste it into the browser, and click **Continue**.
- Select **Install Suggested Plugins**.
- Create your **Admin User**, click **Save and Finish**.

---

### **Step 6: Create a Freestyle Pipeline**

- Click **New Item** on the Jenkins dashboard.
- Enter a name and select **Freestyle Project**. Click **OK**.
- Under **Source Code Management**, select **Git**.
- Paste your **GitHub Repository URL**.
- Specify the branch (usually `/main` or `/master`).
- Under **Build Triggers**, check **GitHub hook trigger for GITScm polling**.
- Click **Save**.
- Click **Build Now** to verify that Jenkins can pull the code successfully.

---

### **Step 7: Enable GitHub Webhooks for Automation**

- Go to your **GitHub Repository** > **Settings** > **Webhooks**.
- Click **Add webhook**.
- **Payload URL:** `http://<Jenkins_Public_IP>:8080/github-webhook/` (The `/` at the end is vital).
- **Content type:** `application/json`.
- Select **Let me select individual events** and check **Pushes** and **Pull Requests**.
- Click **Add webhook**. (A green checkmark means it's connected).

---

### **Step 8: Verify the Trigger**

- Make a small change to a file in your GitHub repo and commit it.
- Check Jenkins; a new build should start automatically.

---

### **Step 9: Setup SonarQube Server**

- Access the `SonarQube-Server` via terminal.
- **Set Hostname:** `sudo hostnamectl set-hostname sonarqube`. Refresh with `exec bash`.
- Update Ubuntu: `sudo apt update`.
- **Install Java 17:** `sudo apt install openjdk-17-jre`.
- **Download SonarQube:**
    - `sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.0.65466.zip`
    - `sudo apt install unzip`
    - `sudo unzip sonarqube-*.zip -d /opt`
- **Run SonarQube:** Navigate to `/opt/sonarqube-*/bin/linux-x86-64/` and run `./sonar.sh start`.

---

### **Step 10: Configure Port 9000 for SonarQube**

- In AWS EC2 Security Groups for the SonarQube instance, add an Inbound Rule.
- **Custom TCP**, Port **9000**, Source **Anywhere (0.0.0.0/0)**.

---

### **Step 11: Access and Setup SonarQube**

- Go to `http://<SonarQube_Public_IP>:9000`.
- **Login:** Default username: `admin`, Password: `admin`. Change the password immediately.
- Click **Create Project Manually**.
- Give it a name/key. Choose **Jenkins** as your CI tool and **GitHub** as your platform.
- Follow the prompts to generate a **Token**. **Copy this token and save it!**

---

### **Step 12: Install Plugins in Jenkins**

- Go to **Manage Jenkins** > **Plugins** > **Available Plugins**.
- Install **SonarQube Scanner** and **SSH2 Easy** (or **Publish Over SSH**).

---

### **Step 13: Configure SonarQube in Jenkins Tools**

- Go to **Manage Jenkins** > **Tools**.
- Find **SonarQube Scanner**. Click **Add SonarQube Scanner**.
- Give it a name (e.g., `sonar-scanner`) and check **Install automatically**. Save.

---

### **Step 14: Configure SonarQube Server Link**

- Go to **Manage Jenkins** > **System**.
- Find **SonarQube servers**. Click **Add SonarQube**.
- **Name:** `SonarQube`. **Server URL:** `http://<SonarQube_IP>:9000`.
- **Server authentication token:** Click **Add** > **Jenkins**.
- **Kind:** Select **Secret text**. Paste your SonarQube Token into the **Secret** field. Give it an ID and Save.
- Select that token in the dropdown and Save.

---

### **Step 15: Add SonarQube to Pipeline**

- Go to your Project > **Configure** > **Build Steps**.
- Add **Execute SonarQube Scanner**.
- In **Analysis properties**, paste the configuration provided by SonarQube (Step 11). Save and Build.

---

### **Step 16: Setup Docker-Server**

- Access `Docker-Server` terminal.
- Install Docker: Follow the [Official Docker Ubuntu Guide](https://docs.docker.com/engine/install/ubuntu/).
- **Update:** `sudo apt update`.

---

### **Step 17: Secure Connection between two instances**

1. **On Jenkins Server:** Switch to the jenkins user: `sudo su - jenkins`.
2. **Generate Key:** `ssh-keygen -t rsa`. (Press Enter for all prompts).
3. **Get the Key:** `cat ~/.ssh/id_rsa.pub`. **Copy this long text.**
4. **On Docker Server:** * Go to the ubuntu user's home: `cd /home/ubuntu/.ssh`.
    - Open the authorized keys: `nano authorized_keys`.
    - **Paste** the Jenkins key on a new line. Save and Exit (`Ctrl+O`, `Enter`, `Ctrl+X`).
5. **Test:** In Jenkins terminal, type `ssh ubuntu@<Docker_Server_IP>`. If you log in without a password, it works! Type `exit`.

---

### **Step 18: Add Docker Server to Jenkins UI**

- Go to **Manage Jenkins** > **System**.
- Find **SSH Remote Hosts** (or SSH2 Easy).
- Add the **Docker-Server IP**, Port **22**, and use the **ubuntu** username.
- For credentials, add the **SSH Private Key** you generated on the Jenkins server (found in `~/.ssh/id_rsa`).

---

### **Step 19: Create the Dockerfile in GitHub**

- In your GitHub repo, create a file named `Dockerfile`:Dockerfile
    
    `FROM nginx:latest
    COPY . /usr/share/nginx/html/`
    

---

### **Step 20: Copy files from Jenkins to Docker Server**

1. Go to Jenkins > Project > **Configure**.
2. In **Build Steps**, select **Execute Shell** (This runs on the Jenkins machine).
3. Type the command:
    
    `scp -r ./* ubuntu@<Docker_Server_IP>:~/website/`
    

### **Step 21: Testing the Remote Connection (The "Handshake" Test)**

Before we try to build Docker images, we need to make sure Jenkins can actually "touch" the Docker server.

1. Go to your Jenkins Pipeline in the browser.
2. Click **Configure** on the left menu.
3. Scroll down to **Build Steps**.
4. Click the **Add build step** button and select **Execute shell script on remote host using ssh** (or "Remote Shell").
5. **Select Target Server:** Choose the Docker-Server you added in Step 21.
6. **Command:** Type a simple test command:Bash
    
    `touch connection_test.txt
    ls -l`
    
7. Click **Save**, then click **Build Now**.
8. **Verify:** * If the build is **Blue/Green (Success)**: Jenkins successfully talked to the Docker server!
    - Now, go to your terminal where you are logged into the **Docker-Server** and type `ls`. You should see `connection_test.txt` sitting there.

### **Step 22: Build and Run the Container (The Professional Way)**

1. In the same **Configure** page, click **Add build step** > **Remote Shell** (This runs on the Docker machine).
2. Select your target server (Docker-Server).
3. Write these commands exactly:Bash
    
    `cd ~/website
    # 1. Build the new image
    sudo docker build -t mywebsite .
    
    # 2. Stop the old container if it is running (so the name is free)
    sudo docker stop give_a_name || true
    
    # 3. Remove the old container
    sudo docker rm give_a_name || true
    
    # 4. Run the new container
    sudo docker run -d -p 8085:80 --name=give_a_name mywebsite`
    
    *(Note: The `|| true` is a trick. It tells Jenkins: "If the container isn't running, don't worry, keep going.")*
    

---

### **Step 23: Open the Port**

- Go to AWS > Security Groups > Docker Instance.
- Add Port **8085** to Inbound Rules.
- Visit `http://<IP>:8085`.

---

## 📈 Future Enhancements
- [ ] Migrate from Freestyle to **Jenkinsfile (Declarative Pipeline)**.
- [ ] Implement **Terraform** for Infrastructure as Code (IaC).
- [ ] Add **Slack notifications** for build status updates.

---
Created by [Nur Hossain Sakil] - Learning Purpose.
