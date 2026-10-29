# FDE Task Tracker - Native AWS CI/CD Hands-On Lab

## 1. Lab Goal

Build a student-friendly AWS CI/CD workflow with three source repositories:

```text
UI source  -> CodeBuild -> S3 UI artifact ----+
                                            |
API source -> CodeBuild -> S3 API artifact    |
                                            |
Infra source -> CodeBuild -> EC2 + SSM <------+ 
                                  |
                                  v
                             Apache Web UI
```

This lab teaches students how source code, build automation, artifacts, IAM roles, EC2, S3, and Systems Manager fit together.

## 2. Lab Values Used

- AWS Region: `ap-south-2`
- Artifact bucket: `fde-artifacts-wilson`
- UI repository: `ui-src`
- API repository: `api-src`
- Infrastructure repository: `infra-src`
- UI CodeBuild project: `fde-ui-build`
- API CodeBuild project: `fde-api-build`
- Infrastructure CodeBuild project: `fde-infra-build`
- EC2 name tag: `fde-task-ui`
- EC2 IAM role / instance profile: `fde-task-ec2-role`

> Student note: Resource IDs such as AMI IDs, subnet IDs, security-group IDs, and key-pair names are account and Region specific. Always verify them in your own AWS account before running `infra.sh`.

---

## 3. Prerequisites

Students need access to AWS services used in the lab:

- AWS CodeCommit
- AWS CodeBuild
- Amazon S3
- Amazon EC2
- AWS IAM
- AWS Systems Manager
- Amazon VPC

Before the lab, select `ap-south-2` consistently in the AWS console.

---

# Part A - Artifact Storage

## 4. Create the S3 Artifact Bucket

Open **Amazon S3 -> Create bucket**.

Create:

```text
fde-artifacts-wilson
```

Recommended lab settings:

- Region: `ap-south-2`
- Block Public Access: enabled
- ACLs: disabled
- Bucket versioning: enabled for the lab
- Default encryption: keep enabled

### Why do we need S3?

The UI and API build jobs produce deployable artifacts. S3 provides a central location where later stages can retrieve those artifacts.

Expected artifact layout:

```text
fde-artifacts-wilson/
├── ui-artifact.zip
└── api-artifact.zip
```

---

# Part B - UI Pipeline

## 5. Create UI CodeCommit Repository

Create:

```text
ui-src
```

Create these files at the repository root:

```text
ui-src/
├── index.html
├── styles.css
├── app.js
└── buildspec.yml
```

## 6. Sample `index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FDE Task Tracker</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
<div class="container">
    <h1>FDE Task Tracker</h1>
    <p>Native AWS CI/CD Pipeline Lab</p>
    <div class="status">
        <h2>Deployment Successful</h2>
        <p>This application was deployed through the AWS lab pipeline.</p>
    </div>
    <button onclick="showMessage()">Test Application</button>
    <p id="message"></p>
</div>
<script src="app.js"></script>
</body>
</html>
```

## 7. Sample `styles.css`

```css
body {
    margin: 0;
    font-family: Arial, sans-serif;
    background: #f4f7fb;
    color: #232f3e;
}

.container {
    width: 90%;
    max-width: 800px;
    margin: 80px auto;
    padding: 40px;
    background: white;
    border-radius: 10px;
    text-align: center;
}

h1 { color: #ff9900; }
.status { padding: 20px; background: #e8f5e9; }
button { padding: 12px 24px; cursor: pointer; }
```

## 8. Sample `app.js`

```javascript
function showMessage() {
    document.getElementById("message").innerText =
        "AWS CI/CD deployment is working successfully!";
}
```

## 9. UI `buildspec.yml`

```yaml
version: 0.2

phases:
  install:
    commands:
      - echo "Starting UI build"

  pre_build:
    commands:
      - echo "Validating required UI files"
      - test -f index.html
      - test -f styles.css
      - test -f app.js

  build:
    commands:
      - echo "Checking UI source files"
      - |
        if grep -nE '<<<<<<<|=======|>>>>>>>' index.html styles.css app.js; then
          echo "Possible merge conflict markers found"
          exit 1
        fi
      - echo "UI validation completed successfully"

artifacts:
  files:
    - index.html
    - styles.css
    - app.js
  name: ui-artifact
  packaging: ZIP
```

### What this buildspec teaches

1. `install` initializes the build.
2. `pre_build` confirms that required UI files exist.
3. `build` performs basic validation.
4. `artifacts` tells CodeBuild which files are outputs.

## 10. Create `fde-ui-build`

Configure CodeBuild:

```text
Project name: fde-ui-build
Source: AWS CodeCommit
Repository: ui-src
Branch: main
Buildspec: buildspec.yml
Artifact type: Amazon S3
Bucket: fde-artifacts-wilson
Artifact name: ui-artifact.zip
Packaging: Zip
```

Let CodeBuild create its service role if permitted by the lab account.

### UI success checkpoint

The build should complete successfully and the expected UI artifact should be visible in S3 before continuing to infrastructure deployment.

---

# Part C - API Pipeline

## 11. Create API Repository

Create:

```text
api-src
```

Repository layout:

```text
api-src/
├── app.py
├── requirements.txt
└── buildspec.yml
```

## 12. `app.py`

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route("/")
def home():
    return jsonify({
        "application": "FDE Task Tracker API",
        "status": "running"
    })

@app.route("/health")
def health():
    return jsonify({"status": "healthy"}), 200

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

## 13. `requirements.txt`

```text
Flask>=3.0,<4.0
```

## 14. API `buildspec.yml`

```yaml
version: 0.2

phases:
  install:
    runtime-versions:
      python: 3.12
    commands:
      - echo "Installing API build dependencies"
      - python --version
      - python -m pip install --upgrade pip

  pre_build:
    commands:
      - test -f app.py
      - test -f requirements.txt
      - python -m pip install -r requirements.txt

  build:
    commands:
      - python -m py_compile app.py
      - |
        if grep -nE '^(<<<<<<<|=======|>>>>>>>)' app.py requirements.txt; then
          echo "ERROR: Possible merge conflict markers found."
          exit 1
        fi
      - |
        if [ -d tests ]; then
          python -m pip install pytest
          python -m pytest tests/ -v
        else
          echo "No tests directory found; skipping pytest."
        fi
      - rm -rf package api-artifact.zip
      - mkdir -p package
      - cp app.py requirements.txt package/
      - cd package && zip -r ../api-artifact.zip . && cd ..

  post_build:
    commands:
      - echo "API build and packaging completed"
      - ls -lh api-artifact.zip

artifacts:
  files:
    - api-artifact.zip
  name: api-artifact
  discard-paths: yes
```

## 15. Create `fde-api-build`

```text
Project: fde-api-build
Source: AWS CodeCommit
Repository: api-src
Branch: main
Buildspec: buildspec.yml
Artifact type: Amazon S3
Bucket: fde-artifacts-wilson
Artifact packaging: None
```

### Why packaging is `None`

The API build already creates `api-artifact.zip`. Avoid wrapping an already-created ZIP inside another ZIP unnecessarily.

---

# Part D - EC2 Networking and IAM

## 16. Networking

For this lab, an existing/default VPC may be reused when allowed by the environment.

Create a dedicated security group such as:

```text
fde-task-ui-sg
```

For web testing, add the HTTP access required by the lab environment. Prefer a restricted source such as the student's current public IP when possible.

Do not add SSH merely because EC2 is used. This design uses AWS Systems Manager for remote commands.

## 17. Create EC2 Role

Create an IAM role for EC2:

```text
fde-task-ec2-role
```

Attach:

```text
AmazonSSMManagedInstanceCore
```

For the current lab, S3 access must also allow EC2 to retrieve the UI artifact. The lab temporarily used `AmazonS3FullAccess`; after the exercise, replace broad S3 access with least-privilege `s3:GetObject` access to the artifact bucket.

### Why this role exists

The EC2 instance needs an AWS identity for SSM registration and for retrieving the S3 artifact.

---

# Part E - Infrastructure Repository

## 18. Create Infrastructure Repository

Create:

```text
infra-src
```

Repository layout:

```text
infra-src/
├── infra.sh
└── buildspec.yml
```

## 19. `infra.sh` Concept

`infra.sh` launches an Ubuntu EC2 instance and supplies bootstrapping instructions that install tools such as Apache and the SSM agent.

The `aws ec2 run-instances` section must use values from the student's own account:

```bash
aws ec2 run-instances \
  --image-id YOUR_VALID_AMI_ID \
  --instance-type t2.micro \
  --subnet-id YOUR_VALID_SUBNET_ID \
  --security-group-ids YOUR_VALID_SECURITY_GROUP_ID \
  --associate-public-ip-address \
  --key-name YOUR_VALID_KEY_NAME \
  --iam-instance-profile Name=fde-task-ec2-role \
  --user-data "$USER_DATA" \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=fde-task-ui},{Key=Project,Value=FDETaskTracker},{Key=Role,Value=UI}]'
```

> Do not copy AMI, subnet, security-group, or key-pair IDs from another account. Verify each value first.

## 20. Infrastructure Build Variables

The infrastructure build uses:

```yaml
env:
  variables:
    UI_ARTIFACT_S3_URI: "s3://fde-artifacts-wilson/ui-artifact.zip"
    INSTANCE_NAME_TAG: "fde-task-ui"
    WEB_ROOT: "/var/www/html"
```

Its high-level sequence is:

```text
Validate AWS identity
       ↓
Validate infra.sh
       ↓
Confirm UI artifact exists in S3
       ↓
Run infra.sh
       ↓
Wait for EC2 Running
       ↓
Wait for SSM Online
       ↓
Send deployment command through SSM
       ↓
Download UI artifact
       ↓
Extract to /var/www/html
       ↓
Restart Apache
       ↓
Verify deployment
```

## 21. Create `fde-infra-build`

```text
Project name: fde-infra-build
Source: AWS CodeCommit
Repository: infra-src
Branch: main
Buildspec: buildspec.yml
Artifact type: No artifacts
AWS_DEFAULT_REGION: ap-south-2
```

Infrastructure is a deployment job, so this project does not need to produce another application artifact.

---

# Part F - Infra CodeBuild Permissions

## 22. Infrastructure Build Role

The CodeBuild role is different from the EC2 instance role.

```text
CodeBuild role
     |
     +-- Read artifact metadata/data from S3
     +-- Run and inspect EC2
     +-- Pass the EC2 role
     +-- Check SSM instance state
     +-- Send SSM deployment commands
             |
             v
     fde-task-ec2-role
             |
             v
            EC2
```

Example lab policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadFDEArtifacts",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:GetObjectVersion",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::fde-artifacts-wilson",
        "arn:aws:s3:::fde-artifacts-wilson/*"
      ]
    },
    {
      "Sid": "ManageFDEEC2Instance",
      "Effect": "Allow",
      "Action": [
        "ec2:RunInstances",
        "ec2:DescribeInstances",
        "ec2:DescribeInstanceStatus",
        "ec2:DescribeImages",
        "ec2:DescribeSubnets",
        "ec2:DescribeSecurityGroups",
        "ec2:DescribeKeyPairs",
        "ec2:CreateTags"
      ],
      "Resource": "*"
    },
    {
      "Sid": "UseSystemsManagerForDeployment",
      "Effect": "Allow",
      "Action": [
        "ssm:DescribeInstanceInformation",
        "ssm:SendCommand",
        "ssm:GetCommandInvocation",
        "ssm:ListCommandInvocations"
      ],
      "Resource": "*"
    },
    {
      "Sid": "PassFDEEC2Role",
      "Effect": "Allow",
      "Action": "iam:PassRole",
      "Resource": "arn:aws:iam::*:role/fde-task-ec2-role",
      "Condition": {
        "StringEquals": {
          "iam:PassedToService": "ec2.amazonaws.com"
        }
      }
    }
  ]
}
```

If AWS reports an authorization error, use the exact denied action/resource from the AWS error before widening permissions.

---

# Part G - Correct Execution Order

## 23. Run the Lab in This Order

Never start with infrastructure.

### Stage 1 - UI

```text
ui-src
  ↓
fde-ui-build
  ↓
S3
  ↓
ui-artifact.zip
```

Verify the artifact before proceeding.

### Stage 2 - API

```text
api-src
  ↓
fde-api-build
  ↓
S3
  ↓
api-artifact.zip
```

### Stage 3 - Infrastructure

Only after the UI artifact is available:

```text
infra-src
  ↓
fde-infra-build
  ↓
EC2
  ↓
SSM Online
  ↓
Download ui-artifact.zip
  ↓
Apache /var/www/html
```

---

# Part H - CodeBuild Quota Troubleshooting

## 24. Error Seen During the Lab

```text
Build failed to start.
Cannot have more than 0 builds in queue for the account
```

In this lab account, the relevant CodeBuild applied account-level quota was observed as `0`, and a request was submitted to raise the applicable quota to `1`.

### Student lesson

This error occurs before the buildspec executes. Therefore, do not immediately modify:

- application code
- buildspec syntax
- S3 bucket
- IAM artifact permissions

First inspect **Service Quotas -> AWS CodeBuild** in the same AWS Region and match the quota to the CodeBuild compute environment.

Record a screenshot as troubleshooting evidence.

---

# Part I - Validation Checklist

## 25. UI Validation

- [ ] `ui-src` exists
- [ ] `index.html` exists
- [ ] `styles.css` exists
- [ ] `app.js` exists
- [ ] `buildspec.yml` exists
- [ ] `fde-ui-build` exists
- [ ] UI build succeeds
- [ ] `ui-artifact.zip` exists in S3

## 26. API Validation

- [ ] `api-src` exists
- [ ] `app.py` exists
- [ ] `requirements.txt` exists
- [ ] `buildspec.yml` exists
- [ ] `fde-api-build` exists
- [ ] API build succeeds
- [ ] API artifact exists in S3

## 27. Infrastructure Validation

- [ ] `infra-src` exists
- [ ] `infra.sh` exists
- [ ] Infrastructure `buildspec.yml` exists
- [ ] Correct AMI ID
- [ ] Correct subnet ID
- [ ] Correct security-group ID
- [ ] Correct key-pair name
- [ ] EC2 instance profile specified
- [ ] EC2 role has SSM permissions
- [ ] EC2 can retrieve the S3 artifact
- [ ] `fde-infra-build` exists
- [ ] Infra CodeBuild role can pass the EC2 role
- [ ] EC2 reaches Running state
- [ ] SSM reaches Online state
- [ ] Apache is active
- [ ] Browser can reach the application through the access allowed by the security group

---

# Part J - Suggested Student Screenshots

Capture evidence at each important checkpoint:

```text
01-region.png
02-s3-artifact-bucket.png
03-ui-repository.png
04-ui-buildspec.png
05-ui-codebuild-project.png
06-ui-build-success.png
07-ui-artifact-s3.png
08-api-repository.png
09-api-build-success.png
10-api-artifact-s3.png
11-infra-repository.png
12-ec2-role.png
13-infra-codebuild-role.png
14-infra-codebuild-project.png
15-ec2-running.png
16-ssm-online.png
17-browser-validation.png
18-service-quota-troubleshooting.png
```

---

# Part K - Key Concepts for Students

## CodeCommit

Stores the source code used by the builds.

## CodeBuild

Reads the source repository and executes the commands described by `buildspec.yml`.

## S3

Stores build artifacts so that later deployment stages can retrieve them.

## IAM

Controls which AWS identities may call AWS APIs. The CodeBuild role and EC2 role have different purposes.

## EC2

Runs the web-server workload in this lab.

## Systems Manager

Provides the management channel used by the infrastructure build to send the deployment command to EC2.

## Apache

Serves the static UI from:

```text
/var/www/html
```

---

# Part L - Cleanup After the Lab

To avoid leaving unnecessary resources, review and remove resources that are no longer required after the exercise, according to the lab/account rules. Typical lab cleanup candidates include:

- EC2 instance created by the lab
- CodeBuild projects
- CodeCommit repositories
- S3 artifacts/bucket when permitted
- Lab-specific security group
- Lab-specific IAM policies/roles when permitted

Do not remove shared/default VPC resources just because the lab used them.

---

# Final Student Summary

The most important concept is the dependency chain:

```text
SOURCE
  ↓
BUILD
  ↓
ARTIFACT
  ↓
INFRASTRUCTURE
  ↓
DEPLOYMENT
  ↓
VALIDATION
```

Do not troubleshoot the final EC2 deployment until the earlier stages are healthy. Verify each checkpoint before continuing to the next stage.
