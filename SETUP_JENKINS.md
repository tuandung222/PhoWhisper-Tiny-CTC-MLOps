# Jenkins CI/CD Setup Guide (Linux/Cloud)

This guide will help you set up a Jenkins server for automating CI/CD pipelines for the PhoWhisper-Tiny-CTC-MLOps project on a remote Linux machine or cloud VM.

---

## 1. Prerequisites

- Ubuntu 20.04/22.04 (or similar Linux distribution)
- Admin (sudo) privileges
- Docker installed and running
- Java 11+ installed
- Git installed
- (Optional) Domain and SSL for production

## 2. Install Jenkins

```bash
# Install Java (if not already installed)
sudo apt update
sudo apt install -y openjdk-11-jre

# Add Jenkins repository and key
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

# Install Jenkins
sudo apt update
sudo apt install -y jenkins
sudo systemctl enable --now jenkins
```

- Access Jenkins at: `http://<your-server-ip>:8080`

## 3. Initial Setup

- Get the admin password:
  ```bash
  sudo cat /var/lib/jenkins/secrets/initialAdminPassword
  ```
- Open Jenkins in your browser and complete the setup wizard.
- Install suggested plugins.

## 4. Install Required Plugins

- Docker Pipeline
- Blue Ocean
- GitHub Integration
- Credentials Binding
- Pipeline Utility Steps

(Manage Jenkins → Manage Plugins)

## 5. Configure Docker Access

- Add the `jenkins` user to the `docker` group:
  ```bash
  sudo usermod -aG docker jenkins
  sudo systemctl restart jenkins
  ```
- Ensure Docker is running: `sudo systemctl status docker`

## 6. Set Up Credentials

- Go to **Manage Jenkins → Credentials → (global)**
- Add:
  - **DockerHub**: Username/Password (ID: `docker-hub-credentials`)
  - **DigitalOcean API Token**: Secret Text (ID: `do-api-token`)

## 7. Create a Pipeline Job

- New Item → Pipeline → Name: `PhoWhisper-Tiny-CTC-MLOps`
- In Pipeline config:
  - Definition: Pipeline script from SCM
  - SCM: Git
  - Repository URL: `https://github.com/tuandung222/Convert-PhoWhisper-ASR-from-encdec-to-ctc.git`
  - Script Path: `Jenkinsfile`

## 8. Trigger Builds

- Configure webhooks in GitHub (Settings → Webhooks) to trigger Jenkins on push.
- Or trigger manually in Jenkins.

## 9. Monitor and Troubleshoot

- View build logs in Jenkins UI.
- Check Docker and Jenkins logs for errors.

---

**For more details, see the Jenkinsfile and project README.**
