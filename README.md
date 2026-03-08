# Jenkins + Terraform AWS Demo

Automated AWS infrastructure deployment using Jenkins and Terraform.

## What This Project Does
- Developer pushes code to GitHub.
- Jenkins pipeline runs Terraform.
- Terraform creates AWS infrastructure.
- EC2 serves a simple web page.

## Architecture
GitHub -> Jenkins Pipeline -> Terraform -> AWS EC2

## Infrastructure Scope
- 1 x EC2 `t2.micro`
- 1 x Security Group (HTTP 80)
- 1 x Web page served by Apache
- Region: `ca-central-1` (Toronto)

## Project Structure
```text
jenkins-terraform-project/
|-- terraform/
|   |-- main.tf
|   |-- variables.tf
|   `-- outputs.tf
|-- app/
|   `-- index.html
|-- jenkins/
|   |-- Dockerfile
|   `-- plugins.txt
|-- Jenkinsfile
|-- docker-compose.yml
`-- README.md
```

## Prerequisites
- Docker Desktop (running)
- Git
- AWS account
- IAM user credentials with permission to create EC2 + security group

## Step 1: Start Jenkins In Docker
From project root:

```powershell
Set-Location "c:\Users\Cryptoboy2\Desktop\Skills Upgrade\Jankins & Terraform\jenkins-terraform-project"
docker compose up --build -d
```

Check container:

```powershell
docker ps
```

Get initial Jenkins admin password:

```powershell
docker exec jenkins-server cat /var/jenkins_home/secrets/initialAdminPassword
```

Open Jenkins:
- `http://localhost:8080`

## Step 2: Verify Terraform And Plugins In Jenkins Container
The custom Docker image already installs Terraform and required plugins.

Check Terraform version:

```powershell
docker exec jenkins-server terraform version
```

Plugins included by default from `jenkins/plugins.txt`:
- Git
- GitHub
- Pipeline (`workflow-aggregator`)
- Credentials Binding
- Terraform

## Step 3: Add AWS Credentials In Jenkins
In Jenkins:
- Go to `Manage Jenkins` -> `Credentials` -> `(global)` -> `Add Credentials`
- Add Secret Text:
  - ID: `aws-access-key-id`
  - Secret: your `AWS_ACCESS_KEY_ID`
- Add Secret Text:
  - ID: `aws-secret-access-key`
  - Secret: your `AWS_SECRET_ACCESS_KEY`

These IDs must match `Jenkinsfile`.

## Step 4: Create Pipeline Job
- New Item -> `Pipeline`
- Under Pipeline definition:
  - Choose `Pipeline script from SCM`
  - SCM: `Git`
  - Repository URL: your GitHub repo URL
  - Script Path: `Jenkinsfile`

## Step 5: Trigger Strategy
Because Jenkins is local, GitHub webhook needs a public tunnel to reach `localhost`.

Options:
- Use a tunnel (for example ngrok) and configure GitHub webhook.
- For local testing, trigger builds manually in Jenkins.

## Step 6: Run Deploy (`ACTION=apply`)
- Build with Parameters
- Select `ACTION=apply`
- Pipeline runs: checkout, init, fmt, validate, plan
- Approve manual gate when prompted
- Apply executes and creates infrastructure

After success, check `terraform output` stage for:
- `public_ip`
- `web_url`

Open the URL in browser.

Expected page:
- `Hello from Terraform deployed server!`

## Step 7: Destroy Resources (`ACTION=destroy`)
To avoid ongoing cost:
- Build with Parameters
- Select `ACTION=destroy`
- Pipeline runs destroy plan and apply

## Useful Commands
Stop Jenkins container:

```powershell
docker compose down
```

Stop and remove volume (clears Jenkins data):

```powershell
docker compose down -v
```

## Notes
- This project uses local Terraform state for simplicity.
- For production-style setup, use remote state (S3 + lock table) and tighter IAM permissions.
