# CI/CD Pipeline for Java Maven Application using Jenkins

This repository contains a complete CI/CD pipeline setup for a Java-based Maven application using Jenkins.  
The pipeline automates building, testing, code analysis, artifact storage, and notifications using the following tools:

- **Jenkins** (running on AWS EC2)
- **Maven** (for building the application)
- **SonarQube** (for code quality analysis)
- **Nexus** (for storing build artifacts)
- **Slack** (for build status notifications)

This project is ideal for showcasing DevOps skills using real-world tools in a full CI/CD workflow.
---

##  Project Overview

This CI/CD pipeline automates the software development lifecycle for a Java Maven application by integrating several DevOps tools.  
The pipeline runs automatically on every code push and performs the following steps:

1. **Clone the source code** from GitHub.
2. **Build the application** using Maven.
3. **Run unit tests** to validate the code.
4. **Perform static code analysis** using Checkstyle and SonarQube.
5. **Apply a Quality Gate** using SonarQube rules to enforce code quality.
6. **Upload the generated .war file** to a Nexus repository.
7. **Send build status notifications** to a Slack channel.

This setup mimics a real-world DevOps pipeline used in production environments.
---

##  Tools & Technologies

The project integrates multiple DevOps tools and technologies to create a complete CI/CD pipeline:

- **Jenkins** – Automation server used to orchestrate the CI/CD workflow.
- **Maven** – Build tool for compiling and packaging the Java application.
- **JDK 17 & 21** – Java Development Kits used during the build process.
- **Git & GitHub** – Source code management and version control.
- **SonarQube** – Static code analysis tool used to assess code quality and enforce rules.
- **Checkstyle** – Tool for coding standards enforcement.
- **Nexus Repository Manager 3** – Used to store and manage the compiled `.war` file.
- **Slack** – Sends build notifications for success or failure.
---

##  Pipeline Stages

The Jenkins pipeline is written using the Declarative Pipeline syntax and contains the following stages:

###  1. Fetch Code
- Pulls the source code from the `atom` branch of the GitHub repository.

###  2. Build
- Runs the Maven build command: `mvn install -DskipTests`
- Skips tests during the build process to speed up artifact generation.

###  3. Unit Testing
- Executes unit tests using the command: `mvn test`
- Ensures that the code behaves as expected.

###  4. Checkstyle Analysis
- Performs static code analysis to detect style violations using: `mvn checkstyle:checkstyle`

###  5. SonarQube Analysis
- Runs SonarQube scanner to analyze code for bugs, vulnerabilities, and code smells.
- Generates metrics and sends them to the SonarQube dashboard.

###  6. Quality Gate
- Waits for the result of the SonarQube Quality Gate.
- If the gate fails, the pipeline is aborted automatically.

###  7. Upload Artifact to Nexus
- Uploads the generated `.war` file to a Nexus repository using the `nexusArtifactUploader` plugin.
- Includes versioning using build ID and timestamp.

###  8. Slack Notification
- Sends a formatted message to a Slack channel with the build status (SUCCESS/FAILURE).
---

##  Setup Instructions

This CI/CD pipeline is designed to run on a main Jenkins EC2 instance, connected to separate SonarQube and Nexus EC2 instances. Follow these steps to set everything up:

---

###  1. Launch Jenkins EC2 Instance

- Launch an EC2 instance (Ubuntu is recommended).
- Install **JDK 21** and **Jenkins** using the following commands:

```bash
sudo apt update
sudo apt install openjdk-21-jdk -y

sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/" | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update
sudo apt-get install jenkins
```
- Update your EC2 **Security Group** to open the following ports:
  - `8080` – for accessing Jenkins UI  
  - `22` – for SSH access

###  2. Access Jenkins & Install Tools

- Open your browser and navigate to your Jenkins server using its public IP address.  
  For example:  
  `http://13.59.122.100:8080`

- Follow the setup wizard and install the **Suggested Plugins**.

- Once inside Jenkins, go to:  
  `Manage Jenkins > Global Tool Configuration`

- Add the following tools:
  - **JDK 21**  
    Name: `JDK21`
  - **Maven 3.9**  
    Name: `MAVEN3.9`

> These tools will be used in the Jenkinsfile to build and test the application.

###  3. Launch Nexus & SonarQube EC2 Instances

- Create two additional EC2 instances (Ubuntu or Amazon Linux):

  - One for **Nexus Repository Manager**
  - One for **SonarQube Server**

- For each instance:
  - Open the required ports in the security group:
    - `8081` → Nexus
    - `9000` → SonarQube
    - `22` → SSH

  - Allow inbound traffic **only** from the Jenkins EC2 instance (either by using private IP or referencing its security group).

- After installation:
  - Access Nexus in browser: `http://<nexus-ip>:8081`
  - Access SonarQube in browser: `http://<sonarqube-ip>:9000`

###  4. Configure Integration in Jenkins

####  SonarQube Integration:
1. Go to `Manage Jenkins > Configure System`.
2. Scroll to the **SonarQube Servers** section.
3. Add a new server:
   - **Name**: `sonarserver`
   - **Server URL**: `http://<your-sonarqube-ip>:9000`
   - **Authentication Token**: (Generate it from your SonarQube account)

> This name (`sonarserver`) is referenced in the Jenkinsfile.

---

####  Nexus Credentials:
1. Go to `Manage Jenkins > Credentials > (Global)` > `Add Credentials`.
2. Set:
   - **Kind**: Username with password
   - **ID**: `nexuslogin`
   - **Username**: Your Nexus username (e.g., `admin`)
   - **Password**: Your Nexus password

> This credential ID (`nexuslogin`) is also used in the Jenkinsfile to authenticate with Nexus.

###  5. Set Up Slack Notifications

To receive build notifications in a Slack channel, follow these steps:

---

####  1. Create a Slack App & Webhook

1. Go to: https://api.slack.com/apps
2. Click **"Create New App"**
3. Choose **"From scratch"**, give it a name, and select your workspace.
4. Under **"Incoming Webhooks"**, activate it and click **"Add New Webhook to Workspace"**.
5. Select your channel (e.g., `#devopscicd`) and allow access.
6. Copy the generated Webhook URL.

---

####  2. Install Slack Plugin in Jenkins

1. Go to `Manage Jenkins > Plugin Manager`.
2. Install the **Slack Notification** plugin.

---

####  3. Configure Slack in Jenkins

1. Go to `Manage Jenkins > Configure System`.
2. Scroll to the **Slack** section.
3. Set the following:
   - **Workspace**: (name doesn’t matter)
   - **Integration Token Credential ID**: Add a credential using your webhook URL.
   - **Default channel**: `#devopscicd` (or whatever you chose)
4. Test the connection to make sure it works.

---

> After this setup, Slack messages will be sent automatically by the pipeline using the `slackSend` step in the `Jenkinsfile`.

---

---

## 📝 Jenkinsfile

This project uses a Declarative Jenkins Pipeline, defined in a `Jenkinsfile` at the root of the repository.

It includes stages for building, testing, code analysis, and deployment.

Key steps include:
- Git Checkout
- Maven Build
- Unit Testing
- Checkstyle & SonarQube analysis
- Quality Gate enforcement
- Artifact upload to Nexus
- Slack notification

📄 View the full Jenkinsfile here: [Jenkinsfile](./Jenkinsfile)

## 👤 Author

- **Mahmoud Shiha**  

