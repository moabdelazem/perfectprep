# DevOps Interview Preparation

## Table of Contents

- [Linux & System Administration](#linux--system-administration)
- [Networking Fundamentals](#networking-fundamentals)
- [Version Control (Git)](#version-control-git)
- [CI/CD Pipelines](#cicd-pipelines)
- [Containerization (Docker)](#containerization-docker)
- [Container Orchestration (Kubernetes)](#container-orchestration-kubernetes)
- [Infrastructure as Code (Terraform)](#infrastructure-as-code-terraform)
- [Configuration Management (Ansible)](#configuration-management-ansible)
- [Cloud Platforms (AWS/GCP/Azure)](#cloud-platforms-awsgcpazure)
- [Monitoring & Logging](#monitoring--logging)
- [Security & DevSecOps](#security--devsecops)
- [Scripting (Bash/Python)](#scripting-bashpython)
- [Behavioral Questions](#behavioral-questions)

---

## Linux & System Administration

<!-- Add your questions and answers here -->

---

## Networking Fundamentals

<!-- Add your questions and answers here -->

---

## Version Control (Git)

<details>
<summary><strong>Q1: What is Git and how does it differ from other VCS?</strong></summary>

Git is a **distributed version control system** (DVCS) that tracks changes in source code during software development.

| Feature | Git (Distributed) | SVN (Centralized) |
|---------|-------------------|-------------------|
| **Repository** | Full copy on every machine | Single central server |
| **Offline work** | Full functionality offline | Requires server connection |
| **Speed** | Fast (local operations) | Slower (network dependent) |
| **Branching** | Lightweight, fast | Heavy, slow |
| **Backup** | Every clone is a backup | Single point of failure |

**Key Concepts:**
- **Working Directory** - Where you edit files
- **Staging Area (Index)** - Files ready to commit
- **Repository (.git)** - Commit history and metadata

</details>

<details>
<summary><strong>Q2: Explain Git branching and common branch strategies</strong></summary>

A branch is a lightweight pointer to a commit, allowing parallel development.

```bash
# Create and switch to branch
git checkout -b feature/login

# List branches
git branch -a

# Delete branch
git branch -d feature/login
git branch -D feature/login  # Force delete
```

**Common Branching Strategies:**

| Strategy | Description | Best For |
|----------|-------------|----------|
| **Git Flow** | main, develop, feature, release, hotfix | Scheduled releases |
| **GitHub Flow** | main + feature branches, deploy from main | Continuous deployment |
| **Trunk-Based** | Short-lived branches, merge frequently | CI/CD, small teams |

</details>

<details>
<summary><strong>Q3: What is the difference between merge and rebase?</strong></summary>

Both integrate changes from one branch into another, but differently:

| Aspect | Merge | Rebase |
|--------|-------|--------|
| **History** | Preserves all commits, creates merge commit | Rewrites history, linear |
| **Conflicts** | Resolve once | Resolve per commit |
| **Use case** | Public branches, preserving context | Clean up local commits |

```bash
# Merge: Creates merge commit
git checkout main
git merge feature

# Rebase: Replays commits on top of main
git checkout feature
git rebase main
```

**Golden Rule:** Never rebase public/shared branches!

**Interactive Rebase (cleanup commits):**
```bash
git rebase -i HEAD~3  # Edit last 3 commits
# Options: pick, squash, reword, drop, fixup
```

</details>

<details>
<summary><strong>Q4: What is cherry-pick and when would you use it?</strong></summary>

Cherry-pick applies a **specific commit** from one branch to another without merging the entire branch.

```bash
# Apply single commit
git cherry-pick abc1234

# Apply multiple commits
git cherry-pick abc1234 def5678

# Cherry-pick without committing
git cherry-pick abc1234 --no-commit
```

**Use Cases:**
- Hotfix a bug in production without merging unfinished features
- Port a specific feature to a different branch
- Undo an accidental commit to the wrong branch

</details>

<details>
<summary><strong>Q5: Explain git reset vs git revert</strong></summary>

| Command | What it does | History | Safe for shared branches? |
|---------|--------------|---------|---------------------------|
| `git reset` | Moves HEAD, can discard commits | Rewrites history | ❌ No |
| `git revert` | Creates new commit undoing changes | Preserves history | ✅ Yes |

**Reset Modes:**
```bash
git reset --soft HEAD~1   # Keep changes staged
git reset --mixed HEAD~1  # Keep changes unstaged (default)
git reset --hard HEAD~1   # Discard all changes ⚠️
```

**Revert (safe for shared branches):**
```bash
git revert abc1234        # Create commit that undoes abc1234
git revert HEAD~3..HEAD   # Revert last 3 commits
```

</details>

<details>
<summary><strong>Q6: What is git stash and how do you use it?</strong></summary>

Stash temporarily saves uncommitted changes so you can switch branches.

```bash
# Save changes to stash
git stash
git stash push -m "work in progress"

# List stashes
git stash list

# Apply and keep in stash
git stash apply stash@{0}

# Apply and remove from stash
git stash pop

# Drop specific stash
git stash drop stash@{0}

# Clear all stashes
git stash clear

# Create branch from stash
git stash branch new-branch stash@{0}
```

**Use Cases:**
- Switch branches with uncommitted work
- Pull changes when you have local modifications
- Temporarily set aside experimental changes

</details>

<details>
<summary><strong>Q7: Explain .gitignore and common patterns</strong></summary>

`.gitignore` specifies files/directories Git should ignore.

```gitignore
# Dependencies
node_modules/
vendor/

# Build outputs
dist/
build/
*.exe

# Environment files
.env
.env.local

# IDE/Editor
.idea/
.vscode/
*.swp

# OS files
.DS_Store
Thumbs.db

# Logs
*.log
logs/

# Patterns
*.tmp           # All .tmp files
!important.tmp  # Except this one
**/cache/       # cache/ in any directory
```

**Already tracked files:**
```bash
# Stop tracking file but keep locally
git rm --cached filename

# Stop tracking entire directory
git rm -r --cached directory/
```

</details>

<details>
<summary><strong>Q8: What is Git Flow workflow?</strong></summary>

Git Flow is a branching model with defined branch types:

| Branch | Purpose | Merges Into |
|--------|---------|-------------|
| **main** | Production-ready code | - |
| **develop** | Integration branch | main (via release) |
| **feature/** | New features | develop |
| **release/** | Release preparation | main & develop |
| **hotfix/** | Production fixes | main & develop |

```bash
# Start feature
git checkout develop
git checkout -b feature/user-auth

# Finish feature
git checkout develop
git merge --no-ff feature/user-auth

# Start release
git checkout -b release/1.0 develop

# Finish release
git checkout main
git merge --no-ff release/1.0
git tag -a v1.0

# Hotfix
git checkout -b hotfix/critical-bug main
# ... fix bug ...
git checkout main && git merge hotfix/critical-bug
git checkout develop && git merge hotfix/critical-bug
```

</details>

<details>
<summary><strong>Q9: How do you resolve merge conflicts?</strong></summary>

Conflicts occur when Git can't auto-merge changes.

**Conflict markers in file:**
```
<<<<<<< HEAD
Your changes
=======
Their changes
>>>>>>> feature-branch
```

**Resolution steps:**
```bash
# 1. Identify conflicted files
git status

# 2. Open and edit files, remove markers

# 3. Stage resolved files
git add resolved-file.txt

# 4. Complete merge
git commit -m "Resolve merge conflicts"

# Or abort merge
git merge --abort
```

**Tools:**
```bash
# Use merge tool
git mergetool

# VS Code, IntelliJ have built-in conflict resolution
```

</details>

<details>
<summary><strong>Q10: What is git reflog and when is it useful?</strong></summary>

Reflog records when branch tips and HEAD are updated. It's your **safety net** for recovering lost commits.

```bash
# View reflog
git reflog

# Output example:
# abc1234 HEAD@{0}: commit: Add feature
# def5678 HEAD@{1}: checkout: moving from main to feature
# 789abcd HEAD@{2}: reset: moving to HEAD~1

# Recover "lost" commit after hard reset
git reset --hard HEAD@{2}

# Recover deleted branch
git checkout -b recovered-branch abc1234
```

**Use Cases:**
- Recover from accidental `git reset --hard`
- Find commits from deleted branches
- Undo a bad rebase

**Note:** Reflog is local only and expires (default 90 days).

</details>

<details>
<summary><strong>Q11: Explain Git hooks</strong></summary>

Git hooks are scripts that run automatically at certain points in Git workflow.

| Hook | When | Use Case |
|------|------|----------|
| `pre-commit` | Before commit | Lint, format code, run tests |
| `commit-msg` | After commit message entered | Validate commit message format |
| `pre-push` | Before push | Run tests, check branch policy |
| `post-merge` | After merge | Install dependencies |
| `pre-rebase` | Before rebase | Prevent rebasing shared branches |

**Location:** `.git/hooks/`

**Example pre-commit hook:**
```bash
#!/bin/sh
# .git/hooks/pre-commit

# Run linter
npm run lint
if [ $? -ne 0 ]; then
  echo "Lint failed. Commit aborted."
  exit 1
fi
```

**Team sharing:** Use tools like **Husky** (JS) or **pre-commit** (Python) to version-control hooks.

</details>

<details>
<summary><strong>Q12: Common Git troubleshooting commands</strong></summary>

```bash
# Undo last commit, keep changes
git reset --soft HEAD~1

# Amend last commit message
git commit --amend -m "New message"

# Add forgotten file to last commit
git add forgotten-file
git commit --amend --no-edit

# Undo uncommitted changes in file
git checkout -- filename
git restore filename  # Modern

# Discard all local changes
git checkout -- .
git clean -fd  # Remove untracked files/dirs

# Find which commit introduced a bug
git bisect start
git bisect bad HEAD
git bisect good v1.0

# See who changed each line
git blame filename

# Find commits by message
git log --grep="bug fix"

# Find commits changing specific code
git log -S "functionName" --source --all
```

</details>

---

## CI/CD Pipelines

<details>
<summary><strong>Q1: What is CI/CD and why is it important?</strong></summary>

**CI (Continuous Integration)**: Developers frequently merge code into a shared repository, triggering automated builds and tests.

**CD (Continuous Delivery/Deployment)**:
- **Continuous Delivery**: Code is always in a deployable state, but deployment is manual
- **Continuous Deployment**: Every change that passes tests is automatically deployed to production

| Benefit | Description |
|---------|-------------|
| **Faster feedback** | Catch bugs early with automated testing |
| **Reduced risk** | Small, frequent changes are easier to debug |
| **Consistency** | Automated pipelines eliminate human error |
| **Faster releases** | From weeks/months to hours/days |

</details>

<details>
<summary><strong>Q2: What is Jenkins and its architecture?</strong></summary>

Jenkins is an open-source automation server for building CI/CD pipelines.

**Architecture Components:**

| Component | Role |
|-----------|------|
| **Jenkins Controller (Master)** | Orchestrates pipelines, schedules jobs, manages agents |
| **Jenkins Agent (Slave)** | Executes build jobs delegated by controller |
| **Executor** | Thread on an agent that runs a build |
| **Workspace** | Directory where build occurs |
| **Plugins** | Extend Jenkins functionality (1800+ available) |

**Distributed Architecture:**
```
                    ┌─────────────┐
                    │  Controller │
                    │   (Master)  │
                    └──────┬──────┘
           ┌───────────────┼───────────────┐
           ▼               ▼               ▼
      ┌─────────┐    ┌─────────┐    ┌─────────┐
      │ Agent 1 │    │ Agent 2 │    │ Agent 3 │
      │ (Linux) │    │ (Windows)│   │ (Docker)│
      └─────────┘    └─────────┘    └─────────┘
```

</details>

<details>
<summary><strong>Q3: Declarative vs Scripted Pipeline - what's the difference?</strong></summary>

| Aspect | Declarative | Scripted |
|--------|-------------|----------|
| **Syntax** | Structured, opinionated | Flexible Groovy code |
| **Learning curve** | Easier | Steeper |
| **Error handling** | Built-in `post` blocks | Manual try/catch |
| **Validation** | Syntax checked before run | Runtime errors |
| **Best for** | Most pipelines | Complex logic |

**Declarative Pipeline:**
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'make build'
            }
        }
        stage('Test') {
            steps {
                sh 'make test'
            }
        }
    }
    post {
        always { cleanWs() }
        failure { mail to: 'team@example.com' }
    }
}
```

**Scripted Pipeline:**
```groovy
node {
    try {
        stage('Build') {
            sh 'make build'
        }
        stage('Test') {
            sh 'make test'
        }
    } catch (e) {
        mail to: 'team@example.com', subject: 'Build Failed'
        throw e
    } finally {
        cleanWs()
    }
}
```

</details>

<details>
<summary><strong>Q4: Explain key Jenkinsfile directives</strong></summary>

| Directive | Purpose |
|-----------|---------|
| `pipeline` | Root block for declarative pipeline |
| `agent` | Where the pipeline runs (any, none, label, docker) |
| `stages` | Container for stage blocks |
| `stage` | Logical grouping of steps |
| `steps` | Actual commands to execute |
| `environment` | Define environment variables |
| `parameters` | Define build parameters |
| `options` | Pipeline options (timeout, retry, etc.) |
| `when` | Conditional stage execution |
| `post` | Actions after pipeline/stage completion |
| `parallel` | Run stages in parallel |

**Example with multiple directives:**
```groovy
pipeline {
    agent any
    
    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
    
    parameters {
        string(name: 'BRANCH', defaultValue: 'main')
        booleanParam(name: 'DEPLOY', defaultValue: false)
    }
    
    environment {
        DOCKER_REGISTRY = 'myregistry.com'
        APP_VERSION = "${env.BUILD_NUMBER}"
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'docker build -t $DOCKER_REGISTRY/app:$APP_VERSION .'
            }
        }
        
        stage('Deploy') {
            when {
                expression { params.DEPLOY == true }
            }
            steps {
                sh './deploy.sh'
            }
        }
    }
}
```

</details>

<details>
<summary><strong>Q5: What are Jenkins agents and how do you configure them?</strong></summary>

Agents are machines that execute Jenkins jobs.

**Agent Types:**

| Type | Description |
|------|-------------|
| `agent any` | Run on any available agent |
| `agent none` | No global agent, define per stage |
| `agent { label 'linux' }` | Run on agent with specific label |
| `agent { docker 'node:18' }` | Run inside Docker container |
| `agent { kubernetes { ... } }` | Run in Kubernetes pod |

**Docker Agent Example:**
```groovy
pipeline {
    agent {
        docker {
            image 'node:18-alpine'
            args '-v /tmp:/tmp'
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'npm install && npm run build'
            }
        }
    }
}
```

**Per-Stage Agents:**
```groovy
pipeline {
    agent none
    stages {
        stage('Build') {
            agent { label 'linux' }
            steps { sh 'make build' }
        }
        stage('Test Windows') {
            agent { label 'windows' }
            steps { bat 'run-tests.bat' }
        }
    }
}
```

</details>

<details>
<summary><strong>Q6: How do you handle credentials in Jenkins?</strong></summary>

Jenkins Credentials Plugin securely stores secrets.

**Credential Types:**
- Username/Password
- SSH Key
- Secret text
- Secret file
- Certificate

**Using Credentials in Pipeline:**
```groovy
pipeline {
    agent any
    environment {
        // Bind credentials to environment variables
        DOCKER_CREDS = credentials('docker-hub-creds')
        AWS_CREDS = credentials('aws-credentials')
    }
    stages {
        stage('Docker Login') {
            steps {
                // Username/password credentials
                sh 'echo $DOCKER_CREDS_PSW | docker login -u $DOCKER_CREDS_USR --password-stdin'
            }
        }
        stage('Deploy') {
            steps {
                // Using withCredentials block
                withCredentials([
                    usernamePassword(
                        credentialsId: 'db-creds',
                        usernameVariable: 'DB_USER',
                        passwordVariable: 'DB_PASS'
                    )
                ]) {
                    sh './deploy.sh'
                }
            }
        }
    }
}
```

**Best Practices:**
- Never print credentials in logs
- Use credential binding, not environment variables in scripts
- Rotate credentials regularly
- Use least-privilege access

</details>

<details>
<summary><strong>Q7: What are Jenkins Shared Libraries?</strong></summary>

Shared Libraries allow reusing pipeline code across multiple projects.

**Directory Structure:**
```
(root)
├── vars/
│   └── myPipeline.groovy     # Global variables/functions
├── src/
│   └── org/company/Utils.groovy  # Helper classes
└── resources/
    └── templates/            # Non-Groovy files
```

**Define in vars/myPipeline.groovy:**
```groovy
def call(Map config = [:]) {
    pipeline {
        agent any
        stages {
            stage('Build') {
                steps {
                    sh "echo Building ${config.appName}"
                }
            }
        }
    }
}
```

**Use in Jenkinsfile:**
```groovy
@Library('my-shared-library') _

myPipeline(appName: 'my-app')
```

**Or use specific functions:**
```groovy
@Library('my-shared-library') _

pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                script {
                    utils.buildDocker('my-app')
                }
            }
        }
    }
}
```

</details>

<details>
<summary><strong>Q8: Explain Jenkins pipeline triggers</strong></summary>

| Trigger | Description |
|---------|-------------|
| **Poll SCM** | Check source control on schedule |
| **Webhook** | External system triggers build |
| **Upstream** | Trigger when another job completes |
| **Cron** | Time-based schedule |
| **GitHub/GitLab** | Push/PR events |

**Pipeline Triggers:**
```groovy
pipeline {
    agent any
    
    triggers {
        // Poll SCM every 5 minutes
        pollSCM('H/5 * * * *')
        
        // Cron schedule (nightly at 2 AM)
        cron('0 2 * * *')
        
        // After upstream job
        upstream(upstreamProjects: 'build-job', threshold: hudson.model.Result.SUCCESS)
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'make build'
            }
        }
    }
}
```

**Webhook Setup (GitHub):**
1. Add webhook URL: `http://jenkins-url/github-webhook/`
2. Configure in Jenkinsfile:
```groovy
triggers {
    githubPush()
}
```

</details>

<details>
<summary><strong>Q9: How do you run parallel stages in Jenkins?</strong></summary>

Parallel execution speeds up pipelines by running independent tasks simultaneously.

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'make build'
            }
        }
        
        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'make test-unit'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'make test-integration'
                    }
                }
                stage('Security Scan') {
                    steps {
                        sh 'make security-scan'
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                sh 'make deploy'
            }
        }
    }
}
```

**With Different Agents:**
```groovy
stage('Cross-Platform Tests') {
    parallel {
        stage('Linux') {
            agent { label 'linux' }
            steps { sh './run-tests.sh' }
        }
        stage('Windows') {
            agent { label 'windows' }
            steps { bat 'run-tests.bat' }
        }
    }
}
```

</details>

<details>
<summary><strong>Q10: What is Blue Ocean in Jenkins?</strong></summary>

Blue Ocean is a modern UI for Jenkins with better visualization of pipelines.

**Features:**
- Visual pipeline editor
- Better visualization of parallel stages
- GitHub/Bitbucket integration
- Personalized dashboard
- Native support for pull request builds

**Key Differences:**

| Classic UI | Blue Ocean |
|------------|------------|
| Text-based job config | Visual pipeline editor |
| List of builds | Pipeline visualization |
| Manual SCM setup | GitHub/GitLab integration |
| Dense information | Clean, modern design |

**Note:** Blue Ocean is being deprecated in favor of newer UIs, but still widely used.

</details>

<details>
<summary><strong>Q11: How do you implement CI/CD for Docker with Jenkins?</strong></summary>

```groovy
pipeline {
    agent any
    
    environment {
        REGISTRY = 'myregistry.com'
        IMAGE_NAME = 'myapp'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'docker build -t $REGISTRY/$IMAGE_NAME:$IMAGE_TAG .'
            }
        }
        
        stage('Test') {
            steps {
                sh '''
                    docker run --rm $REGISTRY/$IMAGE_NAME:$IMAGE_TAG npm test
                '''
            }
        }
        
        stage('Security Scan') {
            steps {
                sh 'trivy image $REGISTRY/$IMAGE_NAME:$IMAGE_TAG'
            }
        }
        
        stage('Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-registry',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh '''
                        echo $PASS | docker login $REGISTRY -u $USER --password-stdin
                        docker push $REGISTRY/$IMAGE_NAME:$IMAGE_TAG
                        docker tag $REGISTRY/$IMAGE_NAME:$IMAGE_TAG $REGISTRY/$IMAGE_NAME:latest
                        docker push $REGISTRY/$IMAGE_NAME:latest
                    '''
                }
            }
        }
        
        stage('Deploy to K8s') {
            steps {
                sh '''
                    kubectl set image deployment/myapp \
                        myapp=$REGISTRY/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
    }
    
    post {
        always {
            sh 'docker rmi $REGISTRY/$IMAGE_NAME:$IMAGE_TAG || true'
            cleanWs()
        }
    }
}
```

</details>

<details>
<summary><strong>Q12: What are Jenkins pipeline best practices?</strong></summary>

| Practice | Why |
|----------|-----|
| **Use Declarative syntax** | Cleaner, validated before run |
| **Store Jenkinsfile in SCM** | Version control, review changes |
| **Use Shared Libraries** | DRY, consistent standards |
| **Fail fast** | Run quick tests first |
| **Use parallel stages** | Reduce build time |
| **Clean workspace** | Avoid stale artifacts |
| **Set timeouts** | Prevent stuck builds |
| **Use agents wisely** | Controller for orchestration only |
| **Secure credentials** | Never hardcode secrets |
| **Archive artifacts** | Preserve build outputs |

**Example with best practices:**
```groovy
pipeline {
    agent none  // Define per stage
    
    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
        timestamps()
    }
    
    stages {
        stage('Quick Checks') {
            agent { docker 'node:18-alpine' }
            steps {
                sh 'npm run lint'  // Fail fast
            }
        }
        
        stage('Build & Test') {
            parallel {
                stage('Unit Tests') { /* ... */ }
                stage('Integration Tests') { /* ... */ }
            }
        }
    }
    
    post {
        always { cleanWs() }
        success { archiveArtifacts artifacts: 'dist/**' }
        failure { slackSend channel: '#builds', message: 'Build Failed!' }
    }
}
```

</details>

<details>
<summary><strong>Q13: How do you troubleshoot Jenkins pipeline failures?</strong></summary>

**Common Issues & Solutions:**

| Issue | Solution |
|-------|----------|
| **Build stuck** | Check agent availability, set timeouts |
| **Workspace issues** | Clean workspace, check disk space |
| **Permission denied** | Check file permissions, user context |
| **Plugin conflicts** | Update plugins, check compatibility |
| **Memory issues** | Increase heap, check for leaks |

**Debugging Techniques:**
```groovy
pipeline {
    agent any
    stages {
        stage('Debug') {
            steps {
                // Print environment
                sh 'printenv | sort'
                
                // Print workspace
                sh 'pwd && ls -la'
                
                // Enable verbose output
                sh 'set -x && ./build.sh'
            }
        }
    }
}
```

**Useful Locations:**
- **Build logs**: `$JENKINS_HOME/jobs/<job>/builds/<build>/log`
- **System logs**: Manage Jenkins → System Log
- **Pipeline steps**: Blue Ocean or Pipeline Steps view
- **Replay**: Modify and re-run pipeline without committing

</details>

<details>
<summary><strong>Q14: How do you secure Jenkins?</strong></summary>

| Security Measure | Description |
|------------------|-------------|
| **Enable authentication** | LDAP, Active Directory, SAML |
| **Authorization strategy** | Matrix-based, Role-based (RBAC) |
| **Secure controller** | Run behind reverse proxy with HTTPS |
| **Limit agent access** | Use agent-to-controller security |
| **Audit logging** | Track user actions |
| **Update regularly** | Patch vulnerabilities |
| **Credential management** | Use Jenkins Credentials, rotate regularly |
| **Disable script console** | Or restrict to admins only |

**Security Configuration:**
```groovy
// In Jenkins system config or init.groovy.d/
jenkins.model.Jenkins.instance.setSecurityRealm(
    new hudson.security.LDAPSecurityRealm(...)
)

jenkins.model.Jenkins.instance.setAuthorizationStrategy(
    new hudson.security.ProjectMatrixAuthorizationStrategy()
)
```

**Content Security Policy:**
```bash
# Restrict inline scripts
-Dhudson.model.DirectoryBrowserSupport.CSP="sandbox; default-src 'self';"
```

</details>

<details>
<summary><strong>Q15: Compare Jenkins with other CI/CD tools</strong></summary>

| Feature | Jenkins | GitHub Actions | GitLab CI | CircleCI |
|---------|---------|----------------|-----------|----------|
| **Hosting** | Self-hosted | Cloud | Cloud/Self | Cloud |
| **Config** | Jenkinsfile | YAML | YAML | YAML |
| **Plugins** | 1800+ | Marketplace | Built-in | Orbs |
| **Learning curve** | Steep | Easy | Medium | Easy |
| **Scalability** | High (agents) | High | High | High |
| **Cost** | Free (OSS) | Free tier | Free tier | Free tier |
| **Best for** | Enterprise, complex | GitHub projects | GitLab users | Fast setup |

**When to choose Jenkins:**
- Need full control over infrastructure
- Complex enterprise requirements
- Extensive plugin ecosystem needed
- On-premise/air-gapped environments
- Already have Jenkins expertise

</details>

<details>
<summary><strong>Q16: What is Azure DevOps and its components?</strong></summary>

Azure DevOps is Microsoft's cloud-based DevOps platform providing end-to-end development tools.

**Core Services:**

| Service | Purpose |
|---------|---------|
| **Azure Repos** | Git repositories (or TFVC) |
| **Azure Pipelines** | CI/CD pipelines |
| **Azure Boards** | Agile planning, work items, Kanban |
| **Azure Test Plans** | Manual & exploratory testing |
| **Azure Artifacts** | Package management (npm, NuGet, Maven) |

**Key Features:**
- Integrates with GitHub, Bitbucket, any Git repo
- Supports any language, platform, cloud
- Self-hosted agents for on-premise builds
- Marketplace with 1000+ extensions

</details>

<details>
<summary><strong>Q17: Explain Azure Pipelines YAML structure</strong></summary>

Azure Pipelines uses YAML for pipeline-as-code.

**Basic Structure:**
```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'

stages:
- stage: Build
  jobs:
  - job: BuildJob
    steps:
    - task: DotNetCoreCLI@2
      inputs:
        command: 'build'
        projects: '**/*.csproj'

- stage: Deploy
  dependsOn: Build
  condition: succeeded()
  jobs:
  - deployment: DeployWeb
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
          - script: echo Deploying to production
```

**Key Concepts:**

| Element | Description |
|---------|-------------|
| `trigger` | Branch/path triggers |
| `pool` | Agent pool (Microsoft-hosted or self-hosted) |
| `stages` | Major phases (Build, Test, Deploy) |
| `jobs` | Units of work that run on an agent |
| `steps` | Individual tasks/scripts |
| `environment` | Target for deployments with approvals |
| `templates` | Reusable YAML files |

**Multi-Stage Pipeline Example:**
```yaml
trigger:
  branches:
    include:
      - main
      - release/*

stages:
- stage: Build
  jobs:
  - job: Build
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - script: |
        docker build -t myapp:$(Build.BuildId) .
        docker push myregistry.azurecr.io/myapp:$(Build.BuildId)
      displayName: 'Build and Push Docker Image'

- stage: DeployDev
  dependsOn: Build
  jobs:
  - deployment: DeployToDev
    environment: 'dev'
    strategy:
      runOnce:
        deploy:
          steps:
          - script: kubectl apply -f k8s/

- stage: DeployProd
  dependsOn: DeployDev
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - deployment: DeployToProd
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
          - script: kubectl apply -f k8s/
```

</details>

<details>
<summary><strong>Q18: Azure DevOps vs Jenkins - when to use each?</strong></summary>

| Feature | Azure DevOps | Jenkins |
|---------|--------------|---------|
| **Hosting** | Cloud (+ Server option) | Self-hosted |
| **Setup** | Zero setup, SaaS | Install & maintain |
| **Config** | YAML | Jenkinsfile (Groovy) |
| **UI** | Modern, integrated | Classic, plugin-based |
| **Agents** | Microsoft-hosted + self-hosted | Only self-hosted |
| **Integration** | Native Azure/GitHub | Plugin-based |
| **Cost** | Free tier (1800 mins/month) | Free (OSS) |
| **Learning curve** | Easier | Steeper |

**Choose Azure DevOps when:**
- Using Microsoft stack (.NET, Azure)
- Want managed SaaS solution
- Need integrated project management (Boards)
- Prefer YAML over Groovy
- Limited DevOps team bandwidth

**Choose Jenkins when:**
- Need full infrastructure control
- Complex custom workflows
- Air-gapped/on-premise only
- Already invested in Jenkins ecosystem
- Need specific plugins not in Azure

**Hybrid Approach:**
Many organizations use both:
- Azure Boards for planning
- Jenkins for builds (existing investment)
- Azure Artifacts for packages

</details>

---

## Containerization (Docker)

<details>
<summary><strong>Q1: What is Docker and how does it differ from Virtual Machines?</strong></summary>

Docker is a platform for developing, shipping, and running applications in **containers** - lightweight, standalone packages that include everything needed to run an application.

| Aspect | Containers | Virtual Machines |
|--------|------------|------------------|
| **Architecture** | Share host OS kernel | Full OS per VM |
| **Size** | MBs (lightweight) | GBs (heavy) |
| **Startup** | Seconds | Minutes |
| **Isolation** | Process-level | Hardware-level |
| **Performance** | Near-native | Overhead from hypervisor |
| **Resource usage** | Efficient | Resource-intensive |

**Key Insight:** Containers virtualize the OS, VMs virtualize hardware.

</details>

<details>
<summary><strong>Q2: Explain the Docker architecture</strong></summary>

Docker uses a **client-server architecture**:

| Component | Role |
|-----------|------|
| **Docker Client** | CLI tool (`docker` command) that sends commands to daemon |
| **Docker Daemon** | Background service that manages containers, images, networks |
| **Docker Registry** | Stores Docker images (Docker Hub, ECR, GCR) |
| **containerd** | Container runtime that manages container lifecycle |
| **runc** | Low-level runtime that creates/runs containers |

**Flow:** Client → Daemon → containerd → runc → Container

</details>

<details>
<summary><strong>Q3: What is the difference between an Image and a Container?</strong></summary>

| Image | Container |
|-------|-----------|
| Read-only template | Running instance of an image |
| Blueprint/Class | Object/Instance |
| Stored on disk | Lives in memory |
| Immutable | Mutable (has writable layer) |
| Shareable | Isolated process |

```bash
# Image is the recipe, container is the cake
docker pull nginx          # Download image
docker run nginx           # Create container from image
docker ps                  # List running containers
docker images              # List images
```

</details>

<details>
<summary><strong>Q4: Explain Docker image layers and caching</strong></summary>

Docker images are built in **layers**. Each instruction in a Dockerfile creates a new layer. Layers are:

- **Cached** - unchanged layers are reused
- **Shared** - common base layers shared between images
- **Read-only** - only the top container layer is writable

```dockerfile
FROM node:18-alpine     # Layer 1: Base image
WORKDIR /app            # Layer 2
COPY package*.json ./   # Layer 3
RUN npm install         # Layer 4 (cached if package.json unchanged)
COPY . .                # Layer 5
CMD ["npm", "start"]    # Layer 6
```

**Best Practice:** Order instructions from least to most frequently changing to maximize cache hits.

</details>

<details>
<summary><strong>Q5: What are the most important Dockerfile instructions?</strong></summary>

| Instruction | Purpose |
|-------------|---------|
| `FROM` | Base image (required, must be first) |
| `RUN` | Execute commands during build |
| `COPY` | Copy files from host to image |
| `ADD` | Like COPY but can extract tars and fetch URLs |
| `WORKDIR` | Set working directory |
| `ENV` | Set environment variables |
| `EXPOSE` | Document which ports the container listens on |
| `CMD` | Default command when container starts (overridable) |
| `ENTRYPOINT` | Main executable (not easily overridden) |
| `ARG` | Build-time variables |
| `USER` | Set non-root user |
| `HEALTHCHECK` | Container health monitoring |

**CMD vs ENTRYPOINT:**
```dockerfile
ENTRYPOINT ["python", "app.py"]  # Fixed executable
CMD ["--port", "8080"]           # Default arguments (overridable)
```

</details>

<details>
<summary><strong>Q6: What is a multi-stage build and why use it?</strong></summary>

Multi-stage builds use multiple `FROM` statements to create smaller, more secure production images by separating build and runtime environments.

```dockerfile
# Stage 1: Build
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

# Stage 2: Production (minimal image)
FROM alpine:3.18
COPY --from=builder /app/myapp /myapp
CMD ["/myapp"]
```

**Benefits:**

| Without Multi-stage | With Multi-stage |
|--------------------|------------------|
| ~800MB (includes Go compiler) | ~15MB (just binary + alpine) |
| Build tools in production | Only runtime essentials |
| Larger attack surface | Minimal attack surface |

</details>

<details>
<summary><strong>Q7: Explain Docker networking modes</strong></summary>

| Network Mode | Description | Use Case |
|--------------|-------------|----------|
| **bridge** (default) | Private network, containers communicate via IP | Standard applications |
| **host** | Container shares host's network namespace | Maximum performance |
| **none** | No networking | Security-sensitive workloads |
| **overlay** | Multi-host networking for Swarm/K8s | Distributed applications |
| **macvlan** | Assigns MAC address, appears as physical device | Legacy apps requiring L2 |

```bash
# Create custom bridge network
docker network create mynet

# Run containers on same network (can communicate by name)
docker run -d --name db --network mynet postgres
docker run -d --name app --network mynet myapp
# app can reach db at hostname "db"
```

</details>

<details>
<summary><strong>Q8: What are Docker volumes and why use them?</strong></summary>

Volumes provide **persistent storage** for containers. Container data is ephemeral by default - volumes survive container deletion.

| Storage Type | Description | Use Case |
|--------------|-------------|----------|
| **Volumes** | Managed by Docker, stored in Docker area | Production, databases |
| **Bind mounts** | Map host directory to container | Development, config files |
| **tmpfs** | Stored in host memory only | Sensitive data, caches |

```bash
# Named volume (recommended)
docker volume create mydata
docker run -v mydata:/app/data nginx

# Bind mount
docker run -v /host/path:/container/path nginx

# Anonymous volume
docker run -v /app/data nginx
```

**Best Practice:** Use named volumes for databases and persistent data.

</details>

<details>
<summary><strong>Q9: What is Docker Compose?</strong></summary>

Docker Compose defines and runs **multi-container applications** using a YAML file.

```yaml
# docker-compose.yml
services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgres://db:5432/app
    depends_on:
      - db
    
  db:
    image: postgres:15
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=secret

volumes:
  pgdata:
```

**Common Commands:**
```bash
docker compose up -d      # Start all services
docker compose down       # Stop and remove
docker compose logs -f    # Follow logs
docker compose ps         # List services
docker compose exec web sh # Shell into service
```

</details>

<details>
<summary><strong>Q10: How do you reduce Docker image size?</strong></summary>

| Technique | Impact |
|-----------|--------|
| Use **alpine** or **distroless** base images | 5MB vs 100MB+ |
| **Multi-stage builds** | Remove build tools |
| Combine RUN commands | Fewer layers |
| Use `.dockerignore` | Exclude unnecessary files |
| Clean up in same layer | `apt-get clean && rm -rf /var/lib/apt/lists/*` |
| Use specific tags | Avoid `latest` bloat |

**Example: Optimized Dockerfile**
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
USER node
CMD ["node", "server.js"]
```

</details>

<details>
<summary><strong>Q11: What are Docker security best practices?</strong></summary>

| Practice | Why |
|----------|-----|
| **Run as non-root user** | Limit container privileges |
| **Use read-only filesystem** | `--read-only` flag |
| **Scan images for vulnerabilities** | `docker scout`, Trivy, Snyk |
| **Use minimal base images** | Smaller attack surface |
| **Don't store secrets in images** | Use Docker secrets or env vars |
| **Set resource limits** | Prevent DoS |
| **Use specific image tags** | Avoid supply chain attacks |
| **Enable content trust** | `DOCKER_CONTENT_TRUST=1` |

```bash
# Run container securely
docker run \
  --user 1000:1000 \
  --read-only \
  --cap-drop ALL \
  --security-opt no-new-privileges \
  --memory 512m \
  --cpus 0.5 \
  myapp
```

</details>

<details>
<summary><strong>Q12: How do you inspect and debug containers?</strong></summary>

```bash
# View container logs
docker logs <container> -f --tail 100

# Execute command in running container
docker exec -it <container> sh

# Inspect container details (network, mounts, env)
docker inspect <container>

# View resource usage
docker stats

# View running processes
docker top <container>

# Copy files from container
docker cp <container>:/path/file ./local

# View container filesystem changes
docker diff <container>

# Check why container exited
docker inspect <container> --format='{{.State.ExitCode}}'
docker logs <container> 2>&1 | tail -20
```

</details>

<details>
<summary><strong>Q13: What is the difference between COPY and ADD?</strong></summary>

| Feature | COPY | ADD |
|---------|------|-----|
| Copy local files | ✅ | ✅ |
| Extract tar archives | ❌ | ✅ (auto-extracts) |
| Fetch remote URLs | ❌ | ✅ |
| Transparency | More explicit | Magic behavior |

**Best Practice:** Use `COPY` unless you specifically need tar extraction.

```dockerfile
# Prefer COPY
COPY ./src /app/src

# Use ADD only for tar extraction
ADD archive.tar.gz /app/

# DON'T use ADD for URLs (use curl instead)
# BAD: ADD https://example.com/file /app/
# GOOD:
RUN curl -o /app/file https://example.com/file
```

</details>

<details>
<summary><strong>Q14: What is a Docker Registry?</strong></summary>

A registry stores and distributes Docker images.

| Registry | Type | Use Case |
|----------|------|----------|
| **Docker Hub** | Public/Private | Default, open-source images |
| **Amazon ECR** | Private | AWS workloads |
| **Google GCR/Artifact Registry** | Private | GCP workloads |
| **Azure ACR** | Private | Azure workloads |
| **GitHub Container Registry** | Public/Private | GitHub-integrated projects |
| **Harbor** | Self-hosted | Enterprise, air-gapped environments |

```bash
# Push to registry
docker tag myapp:v1 myregistry.com/myapp:v1
docker push myregistry.com/myapp:v1

# Pull from registry
docker pull myregistry.com/myapp:v1

# Login to private registry
docker login myregistry.com
```

</details>

---

## Container Orchestration (Kubernetes)

<details>
<summary><strong>Q1: What is Kubernetes?</strong></summary>

Kubernetes is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications. It provides a way to manage and run containerized applications in a distributed environment, making it easier to build, deploy, and manage containerized applications at scale.

</details>

<details>
<summary><strong>Q2: Explain the Kubernetes architecture</strong></summary>

Kubernetes follows master-worker architecture.

**Master architecture (Control Plane):**

- `API Server`: The API server is the entry point for the control plane.
- `etcd`: The etcd is the key-value store that stores the state of the cluster.
- `Controller Manager`: The controller manager is the component that manages the controllers.
- `Scheduler`: The scheduler is the component that assigns pods to nodes.

**Worker architecture (Data Plane):**

- `Kubelet`: The kubelet is the agent that runs on each node and manages the containers.
- `Container Runtime`: The container runtime is the runtime that runs the containers.
- `Kube Proxy`: The kube proxy is the network proxy that runs on each node and manages the network policies.

</details>

<details>
<summary><strong>Q3: What is the difference between a Pod and a container?</strong></summary>

A Pod is a group of containers that are scheduled together on the same node. A container is an instance of an image that is running in a Pod.

</details>

<details>
<summary><strong>Q4: Why do we need Services if pod can be reached by IP address?</strong></summary>

Pods are ephemeral; their IPs change when they restart. A Service provides a stable IP and DNS name that load-balances traffic across a set of Pods.

</details>

<details>
<summary><strong>Q5: Explain the ReplicaSet and Deployment</strong></summary>

A `Deployment` is the high-level object you interact with. It manages the `ReplicaSet`, which in turn ensures the exact number of Pods are running. Deployments allow for rolling updates and rollbacks.

</details>

<details>
<summary><strong>Q6: What are Namespaces, and why do we need them?</strong></summary>

They are virtual clusters within a physical cluster. They provide logical isolation for different teams, environments (Prod/Dev), or projects and help apply RBAC and resource quotas.

</details>

<details>
<summary><strong>Q7: Explain the difference between ConfigMap and Secret</strong></summary>

Both store configuration data, but Secret are specifically for sensitive data (passwords, tokens, etc), Secrets are only Base64 encoded by default and should be used with encryption-at-rest or external providers like HashiCorp Vault, Sealed Secrets, etc.

</details>

<details>
<summary><strong>Q8: What is difference between Deployment and StatefulSet?</strong></summary>

Deployment is for stateless applications, while StatefulSet is for stateful applications (Databases, etc).

</details>

<details>
<summary><strong>Q9: What is Headless Service?</strong></summary>

A Service with clusterIP: None. It doesn't load-balance; instead, it returns the individual IP addresses of the Pods via DNS. Often used for StatefulSets (Databases) where Pods need to talk to each other directly.

</details>

<details>
<summary><strong>Q10: Explain PV, PVC, and StorageClass</strong></summary>

These three components work together to manage persistent storage in Kubernetes:

- **PersistentVolume (PV)**: A piece of storage in the cluster that has been provisioned by an administrator or dynamically using a StorageClass. Think of it as the actual disk or storage resource (e.g., an AWS EBS volume, NFS share, or local disk). PVs have a lifecycle independent of any Pod.

- **PersistentVolumeClaim (PVC)**: A request for storage by a user. It's like a "ticket" that says "I need 10GB of fast storage." Kubernetes matches the PVC to an available PV that meets the requirements. Pods use PVCs to access storage.

- **StorageClass**: Defines the "type" of storage available (e.g., fast SSD, slow HDD, replicated, etc.). It enables **dynamic provisioning** - when a PVC is created, Kubernetes automatically creates a matching PV using the StorageClass.

**Quick Summary:**
| Component | Description |
|-----------|-------------|
| `StorageClass` | The menu of storage options (SSD, HDD, etc.) |
| `PVC` | Your order ("I want 10GB of SSD storage") |
| `PV` | The actual storage you receive |

</details>

<details>
<summary><strong>Q11: What are PVC Access Modes?</strong></summary>

PersistentVolumeClaims (PVCs) have access modes that define how a volume can be mounted by pods.

**Access Modes:**

| Mode | Abbreviation | Description |
|------|--------------|-------------|
| **ReadWriteOnce** | RWO | Volume can be mounted as read-write by a single node |
| **ReadOnlyMany** | ROX | Volume can be mounted as read-only by many nodes |
| **ReadWriteMany** | RWX | Volume can be mounted as read-write by many nodes |
| **ReadWriteOncePod** | RWOP | Volume can be mounted as read-write by a single pod (K8s 1.22+) |

**Storage Provider Support:**

| Storage Type | RWO | ROX | RWX |
|--------------|-----|-----|-----|
| AWS EBS | Yes | No | No |
| Azure Disk | Yes | No | No |
| GCE Persistent Disk | Yes | No | No |
| NFS | Yes | Yes | Yes |
| CephFS | Yes | Yes | Yes |
| HostPath | Yes | No | No |

**Example PVC:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: standard
```

**When to use each mode:**

- **RWO**: Single-instance databases (PostgreSQL, MySQL), stateful applications
- **ROX**: Shared configuration files, static content served by multiple pods
- **RWX**: Shared file storage, content management systems, build artifacts
- **RWOP**: Strict single-pod access (ensures data integrity)

**Important Notes:**
- A volume can have multiple access modes, but can only be mounted using one mode at a time
- Not all storage providers support all access modes
- RWX typically requires network-attached storage (NFS, CephFS, etc.)

</details>

<details>
<summary><strong>Q12: What is Ingress?</strong></summary>

Ingress is a Kubernetes API object that manages external HTTP/HTTPS access to services within a cluster. It acts as a smart router that sits at the edge of your cluster.

**Why use Ingress instead of LoadBalancer Services?**

- A LoadBalancer Service creates one external IP per service (expensive!)
- Ingress provides a single entry point for multiple services with path-based or host-based routing

**Key Components:**

- **Ingress Resource**: The YAML configuration that defines routing rules (e.g., `/api` goes to service A, `/web` goes to service B)
- **Ingress Controller**: The actual implementation that makes Ingress work (e.g., NGINX, Traefik, HAProxy). Without a controller, Ingress resources do nothing.

**What Ingress provides:**

- Path-based routing (`example.com/api` → api-service)
- Host-based routing (`api.example.com` → api-service)
- SSL/TLS termination
- Load balancing

</details>

<details>
<summary><strong>Q12: How do Liveness, Readiness, and Startup probes differ?</strong></summary>

| Probe | Purpose |
|-------|---------|
| **Startup** | Runs first; disables other probes until the app finishes starting up |
| **Readiness** | Tells K8s when the app is ready to receive traffic (added to Service endpoints) |
| **Liveness** | Tells K8s if the app is alive. If it fails, K8s restarts the container |

</details>

<details>
<summary><strong>Q13: What are the Kubernetes services?</strong></summary>

| Service Type | Description |
|--------------|-------------|
| **ClusterIP** | Exposes the service on a cluster-internal IP address |
| **NodePort** | Exposes the service on a static port on each node's IP address |
| **LoadBalancer** | Exposes the service using a cloud provider's load balancer |

</details>

<details>
<summary><strong>Q14: What are Taints and Tolerations?</strong></summary>

Taints and Tolerations work together to **repel pods from nodes** unless they explicitly allow it.

**Taints** (applied to Nodes):
A taint marks a node as "restricted." Pods won't be scheduled on tainted nodes unless they tolerate the taint.

```bash
# Add a taint to a node
kubectl taint nodes node1 key=value:NoSchedule
```

**Taint Effects:**

| Effect | Description |
|--------|-------------|
| `NoSchedule` | Pods won't be scheduled on this node (existing pods stay) |
| `PreferNoSchedule` | Scheduler will _try_ to avoid this node, but may still schedule if needed |
| `NoExecute` | Existing pods are evicted, new pods won't be scheduled |

**Tolerations** (applied to Pods):
A toleration allows a pod to be scheduled on nodes with matching taints.

```yaml
tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
```

**Use Cases:**

- Dedicated nodes for specific workloads (GPU nodes, high-memory nodes)
- Preventing regular workloads from running on master nodes
- Evicting pods during node maintenance

</details>

<details>
<summary><strong>Q15: What is a DaemonSet?</strong></summary>

A DaemonSet ensures that a copy of a Pod runs on **all (or some) nodes** in the cluster. When nodes are added, Pods are automatically added to them.

**Use Cases:**

- Log collectors (Fluentd, Filebeat)
- Monitoring agents (Prometheus Node Exporter, Datadog)
- Cluster storage daemons (Ceph, GlusterFS)
- Network plugins (Calico, Weave)

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluentd:latest
```

</details>

<details>
<summary><strong>Q16: What is the difference between Job and CronJob?</strong></summary>

| Type | Purpose |
|------|---------|
| **Job** | Runs a Pod to completion once (e.g., batch processing, database migration) |
| **CronJob** | Runs Jobs on a schedule (like cron in Linux) |

**Job Example:**
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
spec:
  template:
    spec:
      containers:
      - name: migration
        image: my-app:latest
        command: ["./migrate.sh"]
      restartPolicy: Never
  backoffLimit: 3
```

**CronJob Example:**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: nightly-backup
spec:
  schedule: "0 2 * * *"  # 2 AM daily
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup-tool:latest
          restartPolicy: OnFailure
```

</details>

<details>
<summary><strong>Q17: Explain Resource Requests and Limits</strong></summary>

**Requests**: The minimum resources guaranteed to a container. Used by the scheduler to decide which node to place the Pod on.

**Limits**: The maximum resources a container can use. If exceeded, CPU is throttled; memory causes OOMKill.

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "250m"      # 0.25 CPU cores
  limits:
    memory: "512Mi"
    cpu: "500m"      # 0.5 CPU cores
```

**Best Practices:**

- Always set requests for production workloads
- Set limits to prevent runaway processes
- `requests` = what you need; `limits` = maximum allowed

</details>

<details>
<summary><strong>Q18: What is RBAC in Kubernetes?</strong></summary>

RBAC (Role-Based Access Control) regulates access to Kubernetes resources based on the roles of users or service accounts.

**Key Components:**

| Component | Scope | Description |
|-----------|-------|-------------|
| **Role** | Namespace | Defines permissions within a namespace |
| **ClusterRole** | Cluster-wide | Defines permissions across the entire cluster |
| **RoleBinding** | Namespace | Binds a Role to users/service accounts |
| **ClusterRoleBinding** | Cluster-wide | Binds a ClusterRole to users/service accounts |

**Example:**
```yaml
# Role: Allow read-only access to pods
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
---
# RoleBinding: Bind role to a user
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

</details>

<details>
<summary><strong>Q19: What are Network Policies?</strong></summary>

Network Policies control traffic flow between Pods (East-West traffic) and from external sources (North-South traffic). By default, all Pods can communicate with each other.

**Example: Deny all ingress traffic except from specific Pods**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-allow
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

> **Note:** Requires a CNI plugin that supports Network Policies (Calico, Cilium, Weave).

</details>

<details>
<summary><strong>Q20: What is Horizontal Pod Autoscaler (HPA)?</strong></summary>

HPA automatically scales the number of Pod replicas based on observed metrics (CPU, memory, or custom metrics).

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

**Key Points:**

- Requires Metrics Server to be installed
- Scales based on average utilization across all Pods
- HPA takes precedence over Deployment's replica count (don't set both!)

</details>

<details>
<summary><strong>Q21: What are Init Containers?</strong></summary>

Init Containers run **before** the main application containers start. They run to completion sequentially, and the main containers only start after all init containers succeed.

**Use Cases:**

- Wait for a dependency (database, API) to be ready
- Perform setup tasks (clone a git repo, run migrations)
- Security setup (download secrets)

```yaml
spec:
  initContainers:
  - name: wait-for-db
    image: busybox
    command: ['sh', '-c', 'until nc -z db-service 5432; do sleep 2; done']
  - name: init-data
    image: my-app:latest
    command: ['./setup.sh']
  containers:
  - name: app
    image: my-app:latest
```

</details>

<details>
<summary><strong>Q22: How do Rolling Updates work in Kubernetes?</strong></summary>

Rolling updates gradually replace old Pods with new ones, ensuring zero downtime.

**Deployment Strategy:**
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Max extra Pods during update
      maxUnavailable: 0  # Min Pods always available
```

**How it works:**

1. Create new Pod with updated spec
2. Wait for new Pod to become Ready
3. Remove old Pod
4. Repeat until all Pods are updated

**Useful Commands:**
```bash
# Check rollout status
kubectl rollout status deployment/my-app

# View rollout history
kubectl rollout history deployment/my-app

# Rollback to previous version
kubectl rollout undo deployment/my-app

# Rollback to specific revision
kubectl rollout undo deployment/my-app --to-revision=2
```

</details>

<details>
<summary><strong>Q23: What is Helm and why use it?</strong></summary>

Helm is the **package manager for Kubernetes**. It packages Kubernetes manifests into reusable, versioned units called **Charts**.

**Why Helm?**

| Problem | Helm Solution |
|---------|---------------|
| Managing many YAML files | Bundle into a single Chart |
| Environment-specific configs | Use `values.yaml` for templating |
| Version control for deployments | Charts are versioned |
| Sharing configurations | Public/private Chart repositories |

**Key Concepts:**

- **Chart**: A package of Kubernetes resources
- **Release**: An instance of a Chart running in the cluster
- **Repository**: Where Charts are stored and shared

**Common Commands:**
```bash
# Install a chart
helm install my-release bitnami/nginx

# Upgrade a release
helm upgrade my-release bitnami/nginx --set replicaCount=3

# List releases
helm list

# Rollback
helm rollback my-release 1
```

</details>

<details>
<summary><strong>Q24: What are CRDs and Operators?</strong></summary>

**Custom Resource Definitions (CRDs)** extend Kubernetes with new resource types beyond built-in ones (Pods, Services, etc.).

**Operators** are applications that use CRDs to manage complex, stateful applications automatically. They encode operational knowledge (backup, upgrade, scaling) into software.

**Example: A PostgreSQL Operator might:**

- Define a `PostgresCluster` CRD
- Automatically handle replication setup
- Perform automated backups
- Handle failover and recovery

**Popular Operators:**

| Operator | Purpose |
|----------|---------|
| Prometheus Operator | Manages Prometheus instances |
| Cert-Manager | Automates TLS certificates |
| Strimzi | Manages Kafka clusters |
| Zalando Postgres Operator | Manages PostgreSQL clusters |

```yaml
# Example: Using a PostgreSQL CRD
apiVersion: acid.zalan.do/v1
kind: postgresql
metadata:
  name: my-postgres-cluster
spec:
  teamId: "myteam"
  numberOfInstances: 3
  postgresql:
    version: "15"
```

</details>

<details>
<summary><strong>Q25: What is Node Affinity and Anti-Affinity?</strong></summary>

Node Affinity controls **which nodes Pods can be scheduled on** based on node labels. It's more expressive than `nodeSelector`.

**Types of Node Affinity:**

| Type | Behavior |
|------|----------|
| `requiredDuringSchedulingIgnoredDuringExecution` | **Hard requirement** - Pod won't schedule if no matching node |
| `preferredDuringSchedulingIgnoredDuringExecution` | **Soft preference** - Scheduler tries to match but can ignore |

**Example: Required Node Affinity**
```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
```

**Example: Preferred Node Affinity**
```yaml
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 80
        preference:
          matchExpressions:
          - key: zone
            operator: In
            values:
            - us-east-1a
```

**Pod Anti-Affinity** (schedule Pods away from each other):
```yaml
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: redis
        topologyKey: kubernetes.io/hostname
```

**Use Cases:**

- Run Pods on specific hardware (GPU nodes, SSD nodes)
- Spread replicas across availability zones
- Co-locate Pods for low latency (Pod Affinity)
- Separate competing workloads (Pod Anti-Affinity)

**Node Affinity vs Taints/Tolerations:**

| Feature | Node Affinity | Taints/Tolerations |
|---------|---------------|-------------------|
| Direction | Pod attracts to Node | Node repels Pod |
| Use case | "Run me on these nodes" | "Don't run anything except X" |

</details>

<details>
<summary><strong>Q26: What happens behind the scenes when you run `kubectl apply -f deployment.yaml`?</strong></summary>

Here's the complete flow from command to running Pods:

**1. Client Side (kubectl)**
```
kubectl → reads YAML → validates syntax → sends HTTP request to API Server
```

**2. API Server Processing**
```
Request → Authentication → Authorization (RBAC) → Admission Controllers → etcd
```

- **Authentication**: Verifies identity (certificates, tokens, etc.)
- **Authorization**: Checks if user can perform the action (RBAC)
- **Admission Controllers**: Mutating (modify request) → Validating (reject if invalid)
- **Persistence**: Object stored in etcd

**3. Controller Manager (Deployment Controller)**
```
Watches for Deployment changes → Creates/Updates ReplicaSet
```

**4. Controller Manager (ReplicaSet Controller)**
```
Watches for ReplicaSet changes → Creates Pod objects (status: Pending)
```

**5. Scheduler**
```
Watches for Pods with no node assigned → Selects best node → Updates Pod.spec.nodeName
```

Scheduling considers:
- Resource requests (CPU, memory)
- Node affinity/anti-affinity
- Taints and tolerations
- Pod topology spread constraints

**6. Kubelet (on the selected node)**
```
Watches for Pods assigned to its node → Pulls images → Creates containers via CRI
```

**Complete Flow Diagram:**
```
┌─────────┐     ┌────────────┐     ┌───────┐
│ kubectl │────▶│ API Server │────▶│ etcd  │
└─────────┘     └─────┬──────┘     └───────┘
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
┌──────────────┐ ┌───────────┐ ┌─────────┐
│ Deployment   │ │ ReplicaSet│ │Scheduler│
│ Controller   │ │ Controller│ │         │
└──────┬───────┘ └─────┬─────┘ └────┬────┘
       │               │            │
       ▼               ▼            ▼
   Creates         Creates      Assigns
   ReplicaSet      Pods         Node
                                   │
                      ┌────────────┘
                      ▼
               ┌──────────┐
               │  Kubelet │──▶ Creates Container
               └──────────┘
```

</details>

<details>
<summary><strong>Q27: How does the Kubernetes Scheduler decide which node to place a Pod on?</strong></summary>

The scheduler uses a **two-phase process**: Filtering and Scoring.

**Phase 1: Filtering (Predicates)**

Eliminates nodes that cannot run the Pod:

| Filter | What it checks |
|--------|----------------|
| `PodFitsResources` | Does node have enough CPU/memory? |
| `PodFitsHostPorts` | Are required host ports available? |
| `NodeSelector` | Does node match pod's nodeSelector? |
| `NodeAffinity` | Does node satisfy affinity rules? |
| `TaintToleration` | Can pod tolerate node's taints? |
| `CheckVolumeBinding` | Can required volumes be bound? |

**Phase 2: Scoring (Priorities)**

Ranks remaining nodes (0-100 score each):

| Priority | What it favors |
|----------|----------------|
| `LeastRequestedPriority` | Nodes with most available resources |
| `BalancedResourceAllocation` | Even CPU/memory utilization |
| `NodeAffinityPriority` | Preferred affinity matches |
| `ImageLocalityPriority` | Nodes that already have container images |
| `PodTopologySpread` | Even distribution across zones/nodes |

**Final Selection:**
```
Final Score = Σ (Priority Score × Weight)
Winner = Node with highest total score
```

**Example Scenario:**
```
Pod requests: 500m CPU, 256Mi memory

Node A: 2000m free, 4Gi free → Score: 85
Node B: 800m free, 1Gi free  → Score: 45
Node C: 100m free, 512Mi free → FILTERED OUT (insufficient CPU)

Result: Pod scheduled on Node A
```

</details>

<details>
<summary><strong>Q28: How does DNS work inside a Kubernetes cluster?</strong></summary>

Kubernetes runs **CoreDNS** (or kube-dns) as a cluster add-on to provide DNS resolution.

**DNS Records Created:**

| Resource | DNS Format | Example |
|----------|------------|---------|
| Service | `<svc>.<namespace>.svc.cluster.local` | `nginx.default.svc.cluster.local` |
| Pod | `<pod-ip-dashed>.<namespace>.pod.cluster.local` | `10-244-1-5.default.pod.cluster.local` |
| Headless Service | `<pod-name>.<svc>.<namespace>.svc.cluster.local` | `web-0.nginx.default.svc.cluster.local` |

**How Pods Resolve DNS:**

1. Pod's `/etc/resolv.conf` points to CoreDNS Service IP (usually `10.96.0.10`)
2. Pod makes DNS query → CoreDNS Pod
3. CoreDNS resolves and returns IP

**Pod's /etc/resolv.conf:**
```
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

**DNS Resolution Flow:**
```
┌─────────┐  DNS Query   ┌─────────┐   Lookup    ┌───────────┐
│   Pod   │─────────────▶│ CoreDNS │────────────▶│ API Server│
│         │◀─────────────│   Pod   │◀────────────│  (etcd)   │
└─────────┘  IP Address  └─────────┘   Service   └───────────┘
                                        Endpoints
```

**Headless Services (clusterIP: None):**
- Returns Pod IPs directly instead of Service IP
- Essential for StatefulSets where each Pod needs a stable DNS name

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  clusterIP: None  # Headless
  selector:
    app: nginx
```

</details>

<details>
<summary><strong>Q29: How do you debug a Pod that keeps crashing (CrashLoopBackOff)?</strong></summary>

**Step-by-step debugging approach:**

**1. Check Pod Status and Events**
```bash
kubectl describe pod <pod-name>
# Look at: Events, State, Last State, Restart Count
```

**2. Check Container Logs**
```bash
# Current logs
kubectl logs <pod-name>

# Previous container (if restarted)
kubectl logs <pod-name> --previous

# Specific container in multi-container pod
kubectl logs <pod-name> -c <container-name>
```

**3. Common Causes and Solutions**

| Symptom | Likely Cause | Solution |
|---------|--------------|----------|
| `OOMKilled` | Out of memory | Increase memory limits |
| Exit code `1` | Application error | Check logs, fix app |
| Exit code `137` | SIGKILL (OOM or timeout) | Increase resources/timeout |
| Exit code `143` | SIGTERM | Graceful shutdown issue |
| `ImagePullBackOff` | Can't pull image | Check image name, registry auth |
| Probe failures | Liveness/readiness failing | Adjust probe timing or endpoint |

**4. Interactive Debugging**
```bash
# Exec into running container
kubectl exec -it <pod-name> -- /bin/sh

# If container keeps crashing, override entrypoint
kubectl run debug --image=<image> --command -- sleep infinity
kubectl exec -it debug -- /bin/sh
```

**5. Check Resource Constraints**
```bash
kubectl top pod <pod-name>
kubectl describe node <node-name> | grep -A5 "Allocated resources"
```

**6. Debugging Checklist:**
- [ ] Image exists and is pullable?
- [ ] Entrypoint/CMD correct?
- [ ] Environment variables set correctly?
- [ ] ConfigMaps/Secrets mounted?
- [ ] Resource limits sufficient?
- [ ] Probes configured correctly?
- [ ] Dependent services available?

</details>

<details>
<summary><strong>Q30: What is etcd and why is it critical for Kubernetes?</strong></summary>

**etcd** is a distributed, consistent key-value store that serves as Kubernetes' **single source of truth**.

**What etcd Stores:**
- All cluster state (Pods, Services, ConfigMaps, Secrets, etc.)
- Cluster configuration
- Service discovery data
- Current and desired state of all resources

**Why etcd is Critical:**

| Aspect | Impact |
|--------|--------|
| **All state lives here** | If etcd fails, cluster can't function |
| **All API calls go here** | Every kubectl command reads/writes etcd |
| **Distributed consensus** | Uses Raft algorithm for consistency |
| **Watch mechanism** | Controllers use watches to react to changes |

**Architecture:**
```
┌────────────┐    ┌────────────┐    ┌────────────┐
│   etcd-1   │◀──▶│   etcd-2   │◀──▶│   etcd-3   │
│  (Leader)  │    │ (Follower) │    │ (Follower) │
└─────┬──────┘    └────────────┘    └────────────┘
      │
      ▼
┌────────────┐
│ API Server │ (only talks to etcd)
└────────────┘
```

**Key Characteristics:**

- **Strongly consistent**: All reads return latest write
- **Highly available**: Tolerates (n-1)/2 failures in n-node cluster
- **3 nodes minimum** for production HA
- **5 nodes** for large clusters (tolerates 2 failures)

**etcd Operations:**
```bash
# Check etcd health
etcdctl endpoint health

# List all keys
etcdctl get / --prefix --keys-only

# Backup etcd
etcdctl snapshot save backup.db

# Restore from backup
etcdctl snapshot restore backup.db
```

**Best Practices:**
- Run on dedicated nodes (not with workloads)
- Use SSD storage for low latency
- Regular automated backups
- Monitor etcd metrics closely

</details>

---

## Infrastructure as Code (Terraform)

<!-- Add your questions and answers here -->

---

## Configuration Management (Ansible)

<!-- Add your questions and answers here -->

---

## Cloud Platforms (AWS/GCP/Azure)

<!-- Add your questions and answers here -->

---

## Monitoring & Logging

<!-- Add your questions and answers here -->

---

## Security & DevSecOps

<!-- Add your questions and answers here -->

---

## Scripting (Bash/Python)

### Bash Scripting

<details>
<summary><strong>Q1: What is the difference between Bash and Shell?</strong></summary>

**Shell** is a command-line interpreter that provides a user interface for the Unix/Linux operating system. **Bash** (Bourne Again SHell) is one of many shell implementations.

| Shell | Description |
|-------|-------------|
| **sh** | Original Bourne Shell |
| **bash** | Bourne Again Shell (most common on Linux) |
| **zsh** | Z Shell (default on macOS, powerful features) |
| **fish** | User-friendly interactive shell |
| **dash** | Lightweight POSIX shell |

**Shebang Line:**
```bash
#!/bin/bash    # Use Bash
#!/bin/sh      # Use default shell (POSIX compliant)
#!/usr/bin/env bash  # Portable way (finds bash in PATH)
```

**Check current shell:**
```bash
echo $SHELL     # Default login shell
echo $0         # Current shell
```

</details>

<details>
<summary><strong>Q2: Explain Bash variables and quoting</strong></summary>

**Variable Types:**
```bash
# Regular variable
NAME="John"
echo $NAME

# Environment variable (available to child processes)
export PATH="/usr/local/bin:$PATH"

# Special variables
$0    # Script name
$1-$9 # Positional parameters
$#    # Number of arguments
$@    # All arguments as separate words
$*    # All arguments as single string
$?    # Exit status of last command
$$    # Current process ID
$!    # PID of last background process
```

**Quoting:**
```bash
NAME="World"

# Double quotes - allows variable expansion
echo "Hello $NAME"     # Hello World
echo "Path is $PATH"   # expands $PATH

# Single quotes - literal string
echo 'Hello $NAME'     # Hello $NAME

# Backticks or $() - command substitution
echo "Today is $(date)"
echo "Files: `ls | wc -l`"

# Escape character
echo "Price: \$100"    # Price: $100
```

**Variable Operations:**
```bash
# Default values
echo ${VAR:-default}    # Use 'default' if VAR is unset
echo ${VAR:=default}    # Set VAR to 'default' if unset
echo ${VAR:+alternate}  # Use 'alternate' if VAR is set

# String operations
STR="Hello World"
echo ${#STR}            # Length: 11
echo ${STR:0:5}         # Substring: Hello
echo ${STR/World/Bash}  # Replace: Hello Bash
echo ${STR,,}           # Lowercase: hello world
echo ${STR^^}           # Uppercase: HELLO WORLD
```

</details>

<details>
<summary><strong>Q3: How do conditionals work in Bash?</strong></summary>

**If Statement:**
```bash
if [ condition ]; then
    # code
elif [ condition ]; then
    # code
else
    # code
fi
```

**Test Operators:**

| Type | Operator | Meaning |
|------|----------|---------|
| **String** | `-z "$str"` | String is empty |
| | `-n "$str"` | String is not empty |
| | `"$a" = "$b"` | Strings equal |
| | `"$a" != "$b"` | Strings not equal |
| **Numeric** | `-eq` | Equal |
| | `-ne` | Not equal |
| | `-lt` | Less than |
| | `-le` | Less than or equal |
| | `-gt` | Greater than |
| | `-ge` | Greater than or equal |
| **File** | `-f file` | File exists and is regular file |
| | `-d dir` | Directory exists |
| | `-e path` | Path exists |
| | `-r file` | File is readable |
| | `-w file` | File is writable |
| | `-x file` | File is executable |
| | `-s file` | File is not empty |
| **Logic** | `!` | NOT |
| | `-a` or `&&` | AND |
| | `-o` or `\|\|` | OR |

**Examples:**
```bash
# Check if file exists
if [ -f "/etc/passwd" ]; then
    echo "File exists"
fi

# Check if variable is set
if [ -n "$USER" ]; then
    echo "User is: $USER"
fi

# Numeric comparison
if [ "$count" -gt 10 ]; then
    echo "Count is greater than 10"
fi

# Multiple conditions with [[ ]] (Bash specific, preferred)
if [[ -f "$file" && -r "$file" ]]; then
    echo "File exists and is readable"
fi

# Pattern matching (Bash only)
if [[ "$name" == *.txt ]]; then
    echo "It's a text file"
fi
```

**Case Statement:**
```bash
case "$1" in
    start)
        echo "Starting..."
        ;;
    stop)
        echo "Stopping..."
        ;;
    restart)
        echo "Restarting..."
        ;;
    *)
        echo "Usage: $0 {start|stop|restart}"
        exit 1
        ;;
esac
```

</details>

<details>
<summary><strong>Q4: Explain loops in Bash</strong></summary>

**For Loop:**
```bash
# Loop over list
for item in apple banana cherry; do
    echo "$item"
done

# Loop over range
for i in {1..5}; do
    echo "Number: $i"
done

# C-style for loop
for ((i=0; i<5; i++)); do
    echo "Index: $i"
done

# Loop over files
for file in *.txt; do
    echo "Processing: $file"
done

# Loop over command output
for user in $(cat /etc/passwd | cut -d: -f1); do
    echo "User: $user"
done

# Loop over array
SERVERS=("web1" "web2" "db1")
for server in "${SERVERS[@]}"; do
    echo "Server: $server"
done
```

**While Loop:**
```bash
# Basic while
count=0
while [ $count -lt 5 ]; do
    echo "Count: $count"
    ((count++))
done

# Read file line by line (preferred method)
while IFS= read -r line; do
    echo "$line"
done < file.txt

# Infinite loop with break
while true; do
    read -p "Continue? (y/n): " answer
    if [ "$answer" = "n" ]; then
        break
    fi
done
```

**Until Loop:**
```bash
count=0
until [ $count -ge 5 ]; do
    echo "Count: $count"
    ((count++))
done
```

**Loop Control:**
```bash
# break - exit loop
# continue - skip to next iteration
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        continue  # Skip 5
    fi
    if [ $i -eq 8 ]; then
        break     # Stop at 8
    fi
    echo $i
done
# Output: 1 2 3 4 6 7
```

</details>

<details>
<summary><strong>Q5: How do you write functions in Bash?</strong></summary>

**Function Definition:**
```bash
# Method 1
function greet() {
    echo "Hello, $1!"
}

# Method 2 (POSIX compatible)
greet() {
    echo "Hello, $1!"
}

# Call function
greet "World"
```

**Parameters and Return Values:**
```bash
# Parameters accessed via $1, $2, etc.
add_numbers() {
    local num1=$1
    local num2=$2
    local sum=$((num1 + num2))
    echo $sum  # Return via stdout
}

# Capture return value
result=$(add_numbers 5 3)
echo "Sum is: $result"

# Exit status (0-255)
is_valid() {
    if [ "$1" -gt 0 ]; then
        return 0  # Success
    else
        return 1  # Failure
    fi
}

if is_valid 5; then
    echo "Valid"
fi
```

**Local Variables:**
```bash
my_function() {
    local local_var="I'm local"
    global_var="I'm global"
    echo "$local_var"
}

my_function
echo "$global_var"   # Accessible
echo "$local_var"    # Empty (not accessible)
```

**Practical Function Example:**
```bash
#!/bin/bash

log() {
    local level=$1
    shift
    local message="$@"
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    echo "[$timestamp] [$level] $message"
}

check_service() {
    local service=$1
    if systemctl is-active --quiet "$service"; then
        log INFO "$service is running"
        return 0
    else
        log ERROR "$service is not running"
        return 1
    fi
}

# Usage
check_service nginx || log WARN "Attempting restart"
```

</details>

<details>
<summary><strong>Q6: Explain text processing with grep, sed, and awk</strong></summary>

**grep - Pattern Searching:**
```bash
# Basic search
grep "error" logfile.txt

# Case insensitive
grep -i "ERROR" logfile.txt

# Show line numbers
grep -n "error" logfile.txt

# Count matches
grep -c "error" logfile.txt

# Invert match (lines NOT matching)
grep -v "debug" logfile.txt

# Recursive search
grep -r "TODO" /src/

# Extended regex
grep -E "error|warning|critical" logfile.txt

# Show context (2 lines before/after)
grep -B2 -A2 "error" logfile.txt

# Only filenames
grep -l "password" *.conf
```

**sed - Stream Editor:**
```bash
# Replace first occurrence per line
sed 's/old/new/' file.txt

# Replace all occurrences
sed 's/old/new/g' file.txt

# Replace in-place
sed -i 's/old/new/g' file.txt

# Delete lines matching pattern
sed '/pattern/d' file.txt

# Delete empty lines
sed '/^$/d' file.txt

# Print specific lines
sed -n '5,10p' file.txt

# Multiple operations
sed -e 's/foo/bar/g' -e 's/baz/qux/g' file.txt

# Insert line before match
sed '/pattern/i\New line before' file.txt

# Append line after match
sed '/pattern/a\New line after' file.txt
```

**awk - Text Processing:**
```bash
# Print specific columns
awk '{print $1, $3}' file.txt

# Custom field separator
awk -F: '{print $1}' /etc/passwd

# Print with formatting
awk '{printf "%-10s %s\n", $1, $2}' file.txt

# Filter by condition
awk '$3 > 100 {print $1, $3}' file.txt

# Sum a column
awk '{sum += $3} END {print sum}' file.txt

# Count lines
awk 'END {print NR}' file.txt

# Print line numbers
awk '{print NR, $0}' file.txt

# Pattern matching
awk '/error/ {print $0}' logfile.txt

# Built-in variables
# NR = current line number
# NF = number of fields
# $0 = entire line
# $1, $2... = individual fields
# FS = field separator
# OFS = output field separator

# Complex example: Parse Apache logs
awk '{
    ip[$1]++
} END {
    for (i in ip) print ip[i], i
}' access.log | sort -rn | head -10
```

</details>

<details>
<summary><strong>Q7: How do you handle errors in Bash scripts?</strong></summary>

**Exit Codes:**
```bash
# Exit with code
exit 0   # Success
exit 1   # General error

# Check last command status
command
if [ $? -ne 0 ]; then
    echo "Command failed"
    exit 1
fi

# Or more concise
command || { echo "Command failed"; exit 1; }
```

**Set Options for Safety:**
```bash
#!/bin/bash
set -e          # Exit on any error
set -u          # Exit on undefined variable
set -o pipefail # Fail on pipe errors
set -x          # Debug mode (print commands)

# Combined (recommended for scripts)
set -euo pipefail
```

**Trap for Cleanup:**
```bash
#!/bin/bash
set -euo pipefail

TMPFILE=$(mktemp)

cleanup() {
    rm -f "$TMPFILE"
    echo "Cleanup complete"
}

# Run cleanup on EXIT, INT (Ctrl+C), TERM
trap cleanup EXIT INT TERM

# Main script
echo "Working..."
# ... do stuff with TMPFILE
```

**Error Handling Function:**
```bash
#!/bin/bash

die() {
    echo "ERROR: $1" >&2
    exit "${2:-1}"
}

check_root() {
    if [ "$(id -u)" -ne 0 ]; then
        die "This script must be run as root" 2
    fi
}

check_command() {
    command -v "$1" &>/dev/null || die "$1 is required but not installed"
}

# Usage
check_root
check_command docker
check_command kubectl
```

**Try-Catch Pattern:**
```bash
#!/bin/bash

try() {
    "$@"
    return $?
}

catch() {
    local exit_code=$?
    if [ $exit_code -ne 0 ]; then
        echo "Error: Command failed with exit code $exit_code" >&2
        return $exit_code
    fi
}

# Usage
if ! try some_command; then
    catch
    # Handle error
fi
```

</details>

<details>
<summary><strong>Q8: Explain I/O redirection and pipes</strong></summary>

**File Descriptors:**
```bash
0 = stdin  (standard input)
1 = stdout (standard output)
2 = stderr (standard error)
```

**Output Redirection:**
```bash
# Redirect stdout to file (overwrite)
command > file.txt

# Redirect stdout to file (append)
command >> file.txt

# Redirect stderr to file
command 2> error.log

# Redirect both stdout and stderr
command > output.txt 2>&1
command &> output.txt  # Shorthand (Bash)

# Redirect stderr to stdout
command 2>&1

# Suppress all output
command > /dev/null 2>&1
command &> /dev/null
```

**Input Redirection:**
```bash
# Read from file
command < input.txt

# Here document
cat << EOF
This is a
multi-line
string
EOF

# Here string
grep "pattern" <<< "search in this string"
```

**Pipes:**
```bash
# Connect commands
cat file.txt | grep "error" | wc -l

# Tee - write to file and stdout
command | tee output.txt
command | tee -a output.txt  # Append

# xargs - convert stdin to arguments
find . -name "*.log" | xargs rm
find . -name "*.log" -print0 | xargs -0 rm  # Handle spaces

# Process substitution
diff <(sort file1.txt) <(sort file2.txt)
```

**Named Pipes (FIFO):**
```bash
# Create named pipe
mkfifo my_pipe

# Terminal 1: Write to pipe
echo "Hello" > my_pipe

# Terminal 2: Read from pipe
cat < my_pipe
```

</details>

<details>
<summary><strong>Q9: How do you work with arrays in Bash?</strong></summary>

**Array Declaration:**
```bash
# Indexed array
FRUITS=("apple" "banana" "cherry")
SERVERS=(web1 web2 db1 db2)

# Associative array (Bash 4+)
declare -A PORTS
PORTS[http]=80
PORTS[https]=443
PORTS[ssh]=22
```

**Array Operations:**
```bash
# Access elements
echo ${FRUITS[0]}          # First element
echo ${FRUITS[-1]}         # Last element (Bash 4.3+)
echo ${FRUITS[@]}          # All elements
echo ${!FRUITS[@]}         # All indices

# Array length
echo ${#FRUITS[@]}

# Add elements
FRUITS+=("mango")

# Remove element
unset FRUITS[1]

# Slice
echo ${FRUITS[@]:1:2}      # Elements 1-2

# Loop through array
for fruit in "${FRUITS[@]}"; do
    echo "$fruit"
done

# Loop with index
for i in "${!FRUITS[@]}"; do
    echo "$i: ${FRUITS[$i]}"
done
```

**Associative Array Example:**
```bash
#!/bin/bash
declare -A CONFIG

CONFIG[host]="localhost"
CONFIG[port]="8080"
CONFIG[user]="admin"

# Access
echo "Connecting to ${CONFIG[host]}:${CONFIG[port]}"

# Loop
for key in "${!CONFIG[@]}"; do
    echo "$key = ${CONFIG[$key]}"
done
```

</details>

<details>
<summary><strong>Q10: What are essential DevOps Bash scripts?</strong></summary>

**Log Rotation Script:**
```bash
#!/bin/bash
set -euo pipefail

LOG_DIR="/var/log/myapp"
MAX_AGE=7  # days

find "$LOG_DIR" -name "*.log" -type f -mtime +$MAX_AGE -delete
find "$LOG_DIR" -name "*.log" -type f -size +100M -exec gzip {} \;

echo "Log cleanup completed at $(date)"
```

**Health Check Script:**
```bash
#!/bin/bash
set -euo pipefail

SERVICES=("nginx" "postgresql" "redis")
ALERT_EMAIL="admin@example.com"

check_service() {
    if systemctl is-active --quiet "$1"; then
        echo "✓ $1 is running"
        return 0
    else
        echo "✗ $1 is NOT running"
        return 1
    fi
}

failed_services=()

for service in "${SERVICES[@]}"; do
    if ! check_service "$service"; then
        failed_services+=("$service")
    fi
done

if [ ${#failed_services[@]} -gt 0 ]; then
    echo "Failed services: ${failed_services[*]}" | mail -s "Service Alert" "$ALERT_EMAIL"
    exit 1
fi
```

**Backup Script:**
```bash
#!/bin/bash
set -euo pipefail

BACKUP_DIR="/backups"
DB_NAME="myapp"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=30

# Create backup
pg_dump "$DB_NAME" | gzip > "$BACKUP_DIR/${DB_NAME}_${DATE}.sql.gz"

# Remove old backups
find "$BACKUP_DIR" -name "*.sql.gz" -mtime +$RETENTION_DAYS -delete

# Sync to S3
aws s3 sync "$BACKUP_DIR" "s3://my-backups/db/"

echo "Backup completed: ${DB_NAME}_${DATE}.sql.gz"
```

**Deployment Script:**
```bash
#!/bin/bash
set -euo pipefail

APP_DIR="/var/www/myapp"
REPO="git@github.com:company/myapp.git"
BRANCH="${1:-main}"

cd "$APP_DIR"

# Pull latest code
git fetch origin
git checkout "$BRANCH"
git pull origin "$BRANCH"

# Install dependencies
npm ci --production

# Run migrations
npm run migrate

# Restart application
pm2 reload myapp

# Health check
sleep 5
if curl -sf http://localhost:3000/health > /dev/null; then
    echo "Deployment successful!"
else
    echo "Health check failed!"
    exit 1
fi
```

</details>

---

### Python Scripting

<details>
<summary><strong>Q11: Why is Python preferred for DevOps automation?</strong></summary>

| Advantage | Description |
|-----------|-------------|
| **Readability** | Clean syntax, easy to maintain |
| **Rich ecosystem** | Extensive libraries for any task |
| **Cross-platform** | Works on Linux, macOS, Windows |
| **API integration** | Excellent HTTP/REST support |
| **Infrastructure tools** | Ansible, SaltStack, Fabric use Python |
| **Cloud SDKs** | boto3 (AWS), azure-sdk, google-cloud |

**Common DevOps Python Libraries:**

| Library | Purpose |
|---------|---------|
| `requests` | HTTP requests |
| `boto3` | AWS SDK |
| `paramiko` | SSH connections |
| `fabric` | Remote execution |
| `pyyaml` | YAML parsing |
| `jinja2` | Template rendering |
| `click` | CLI applications |
| `pytest` | Testing |
| `docker` | Docker SDK |
| `kubernetes` | Kubernetes SDK |

</details>

<details>
<summary><strong>Q12: Explain Python file handling for DevOps</strong></summary>

**Reading Files:**
```python
# Read entire file
with open('config.txt', 'r') as f:
    content = f.read()

# Read lines into list
with open('hosts.txt', 'r') as f:
    lines = f.readlines()

# Read line by line (memory efficient)
with open('large_log.txt', 'r') as f:
    for line in f:
        process(line.strip())
```

**Writing Files:**
```python
# Write to file
with open('output.txt', 'w') as f:
    f.write('Hello, World!\n')

# Append to file
with open('log.txt', 'a') as f:
    f.write(f'{datetime.now()}: Event occurred\n')
```

**Working with JSON:**
```python
import json

# Read JSON
with open('config.json', 'r') as f:
    config = json.load(f)

# Write JSON
with open('output.json', 'w') as f:
    json.dump(data, f, indent=2)

# Parse JSON string
data = json.loads('{"name": "value"}')
```

**Working with YAML:**
```python
import yaml

# Read YAML
with open('config.yaml', 'r') as f:
    config = yaml.safe_load(f)

# Write YAML
with open('output.yaml', 'w') as f:
    yaml.dump(data, f, default_flow_style=False)
```

**Working with CSV:**
```python
import csv

# Read CSV
with open('data.csv', 'r') as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row['name'], row['value'])

# Write CSV
with open('output.csv', 'w', newline='') as f:
    writer = csv.DictWriter(f, fieldnames=['name', 'value'])
    writer.writeheader()
    writer.writerow({'name': 'foo', 'value': 'bar'})
```

</details>

<details>
<summary><strong>Q13: How do you make HTTP/API requests in Python?</strong></summary>

**Using requests library:**
```python
import requests

# GET request
response = requests.get('https://api.example.com/users')
if response.status_code == 200:
    users = response.json()
    
# GET with parameters
response = requests.get(
    'https://api.example.com/search',
    params={'q': 'python', 'limit': 10}
)

# POST request with JSON
response = requests.post(
    'https://api.example.com/users',
    json={'name': 'John', 'email': 'john@example.com'},
    headers={'Authorization': 'Bearer token123'}
)

# PUT request
response = requests.put(
    'https://api.example.com/users/1',
    json={'name': 'Updated Name'}
)

# DELETE request
response = requests.delete('https://api.example.com/users/1')

# Handling errors
try:
    response = requests.get(url, timeout=30)
    response.raise_for_status()  # Raise exception for 4xx/5xx
except requests.exceptions.RequestException as e:
    print(f"Request failed: {e}")
```

**API Wrapper Example:**
```python
import requests
from typing import Dict, Any

class GitHubAPI:
    def __init__(self, token: str):
        self.base_url = 'https://api.github.com'
        self.session = requests.Session()
        self.session.headers.update({
            'Authorization': f'token {token}',
            'Accept': 'application/vnd.github.v3+json'
        })
    
    def get_repos(self, org: str) -> list:
        response = self.session.get(f'{self.base_url}/orgs/{org}/repos')
        response.raise_for_status()
        return response.json()
    
    def create_issue(self, owner: str, repo: str, title: str, body: str) -> Dict:
        response = self.session.post(
            f'{self.base_url}/repos/{owner}/{repo}/issues',
            json={'title': title, 'body': body}
        )
        response.raise_for_status()
        return response.json()

# Usage
api = GitHubAPI(os.environ['GITHUB_TOKEN'])
repos = api.get_repos('mycompany')
```

</details>

<details>
<summary><strong>Q14: How do you execute shell commands from Python?</strong></summary>

**Using subprocess (recommended):**
```python
import subprocess

# Simple command
result = subprocess.run(['ls', '-la'], capture_output=True, text=True)
print(result.stdout)
print(result.returncode)

# With shell=True (be careful with user input!)
result = subprocess.run('echo $HOME', shell=True, capture_output=True, text=True)

# Check for errors
result = subprocess.run(
    ['docker', 'build', '-t', 'myapp', '.'],
    capture_output=True,
    text=True,
    check=True  # Raises exception on non-zero exit
)

# Stream output in real-time
process = subprocess.Popen(
    ['kubectl', 'logs', '-f', 'pod-name'],
    stdout=subprocess.PIPE,
    text=True
)
for line in process.stdout:
    print(line, end='')

# Run with timeout
try:
    result = subprocess.run(
        ['long_running_command'],
        timeout=60,
        capture_output=True
    )
except subprocess.TimeoutExpired:
    print("Command timed out")
```

**Practical Example - Deploy Script:**
```python
import subprocess
import sys

def run_command(cmd: list[str], description: str) -> bool:
    print(f"→ {description}")
    result = subprocess.run(cmd, capture_output=True, text=True)
    if result.returncode != 0:
        print(f"  ✗ Failed: {result.stderr}")
        return False
    print(f"  ✓ Success")
    return True

def deploy():
    steps = [
        (['docker', 'build', '-t', 'myapp:latest', '.'], 'Building Docker image'),
        (['docker', 'push', 'myapp:latest'], 'Pushing to registry'),
        (['kubectl', 'rollout', 'restart', 'deployment/myapp'], 'Restarting deployment'),
    ]
    
    for cmd, desc in steps:
        if not run_command(cmd, desc):
            sys.exit(1)
    
    print("\n✓ Deployment complete!")

if __name__ == '__main__':
    deploy()
```

</details>

<details>
<summary><strong>Q15: How do you handle exceptions in Python?</strong></summary>

**Try-Except Block:**
```python
try:
    result = risky_operation()
except ValueError as e:
    print(f"Value error: {e}")
except (TypeError, KeyError) as e:
    print(f"Type or key error: {e}")
except Exception as e:
    print(f"Unexpected error: {e}")
    raise  # Re-raise the exception
else:
    print("No exception occurred")
finally:
    cleanup()  # Always runs
```

**Custom Exceptions:**
```python
class DeploymentError(Exception):
    """Raised when deployment fails"""
    def __init__(self, message: str, stage: str):
        self.message = message
        self.stage = stage
        super().__init__(f"Deployment failed at {stage}: {message}")

def deploy(environment: str):
    try:
        build()
    except BuildError as e:
        raise DeploymentError(str(e), "build")
    
    try:
        push_to_registry()
    except RegistryError as e:
        raise DeploymentError(str(e), "push")

# Usage
try:
    deploy("production")
except DeploymentError as e:
    print(f"Failed at stage: {e.stage}")
    notify_team(e.message)
```

**Context Manager for Resource Management:**
```python
from contextlib import contextmanager

@contextmanager
def ssh_connection(host: str):
    conn = create_connection(host)
    try:
        yield conn
    finally:
        conn.close()

# Usage
with ssh_connection('server1') as conn:
    conn.execute('uptime')
```

</details>

<details>
<summary><strong>Q16: Explain Python regex for log parsing</strong></summary>

**Basic Regex Operations:**
```python
import re

text = "Error at 192.168.1.100: Connection refused at 2024-01-15 10:30:45"

# Search for pattern
match = re.search(r'\d+\.\d+\.\d+\.\d+', text)
if match:
    print(f"Found IP: {match.group()}")  # 192.168.1.100

# Find all matches
ips = re.findall(r'\d+\.\d+\.\d+\.\d+', text)

# Replace
cleaned = re.sub(r'\d+\.\d+\.\d+\.\d+', '[REDACTED]', text)

# Split
parts = re.split(r'\s+', text)

# Compile for reuse
IP_PATTERN = re.compile(r'(\d+\.\d+\.\d+\.\d+)')
match = IP_PATTERN.search(text)
```

**Log Parsing Example:**
```python
import re
from collections import Counter

# Apache log pattern
LOG_PATTERN = re.compile(
    r'(?P<ip>\d+\.\d+\.\d+\.\d+) - - '
    r'\[(?P<timestamp>[^\]]+)\] '
    r'"(?P<method>\w+) (?P<path>[^ ]+) [^"]*" '
    r'(?P<status>\d+) (?P<size>\d+)'
)

def parse_log(log_file: str):
    stats = {
        'total_requests': 0,
        'status_codes': Counter(),
        'top_ips': Counter(),
        'top_paths': Counter()
    }
    
    with open(log_file, 'r') as f:
        for line in f:
            match = LOG_PATTERN.match(line)
            if match:
                data = match.groupdict()
                stats['total_requests'] += 1
                stats['status_codes'][data['status']] += 1
                stats['top_ips'][data['ip']] += 1
                stats['top_paths'][data['path']] += 1
    
    return stats

# Usage
stats = parse_log('/var/log/apache2/access.log')
print(f"Total requests: {stats['total_requests']}")
print(f"Top 5 IPs: {stats['top_ips'].most_common(5)}")
```

</details>

<details>
<summary><strong>Q17: How do you use argparse for CLI tools?</strong></summary>

**Basic CLI Tool:**
```python
import argparse
import sys

def main():
    parser = argparse.ArgumentParser(
        description='Deploy application to environment',
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog='''
Examples:
  %(prog)s deploy --env production
  %(prog)s rollback --env staging --version v1.2.3
        '''
    )
    
    # Subcommands
    subparsers = parser.add_subparsers(dest='command', required=True)
    
    # Deploy command
    deploy_parser = subparsers.add_parser('deploy', help='Deploy application')
    deploy_parser.add_argument(
        '--env', '-e',
        required=True,
        choices=['dev', 'staging', 'production'],
        help='Target environment'
    )
    deploy_parser.add_argument(
        '--dry-run',
        action='store_true',
        help='Simulate deployment without making changes'
    )
    deploy_parser.add_argument(
        '--version', '-v',
        default='latest',
        help='Version to deploy (default: latest)'
    )
    
    # Rollback command
    rollback_parser = subparsers.add_parser('rollback', help='Rollback to previous version')
    rollback_parser.add_argument('--env', '-e', required=True)
    rollback_parser.add_argument('--version', '-v', required=True)
    
    args = parser.parse_args()
    
    if args.command == 'deploy':
        deploy(args.env, args.version, args.dry_run)
    elif args.command == 'rollback':
        rollback(args.env, args.version)

def deploy(env: str, version: str, dry_run: bool):
    print(f"Deploying {version} to {env}")
    if dry_run:
        print("(Dry run - no changes made)")

def rollback(env: str, version: str):
    print(f"Rolling back {env} to {version}")

if __name__ == '__main__':
    main()
```

**Using Click (alternative):**
```python
import click

@click.group()
def cli():
    """Deployment CLI tool"""
    pass

@cli.command()
@click.option('--env', '-e', required=True, type=click.Choice(['dev', 'staging', 'prod']))
@click.option('--version', '-v', default='latest')
@click.option('--dry-run', is_flag=True)
def deploy(env, version, dry_run):
    """Deploy application to environment"""
    click.echo(f"Deploying {version} to {env}")

@cli.command()
@click.option('--env', '-e', required=True)
@click.option('--version', '-v', required=True)
def rollback(env, version):
    """Rollback to previous version"""
    click.echo(f"Rolling back {env} to {version}")

if __name__ == '__main__':
    cli()
```

</details>

<details>
<summary><strong>Q18: How do you set up logging in Python?</strong></summary>

**Basic Logging:**
```python
import logging

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('app.log'),
        logging.StreamHandler()  # Console output
    ]
)

logger = logging.getLogger(__name__)

# Usage
logger.debug('Debug message')
logger.info('Info message')
logger.warning('Warning message')
logger.error('Error message')
logger.exception('Exception with traceback')  # Use in except block
```

**Production-Ready Logging:**
```python
import logging
import logging.handlers
import json
from datetime import datetime

class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_data = {
            'timestamp': datetime.utcnow().isoformat(),
            'level': record.levelname,
            'logger': record.name,
            'message': record.getMessage(),
            'module': record.module,
            'function': record.funcName,
            'line': record.lineno
        }
        if record.exc_info:
            log_data['exception'] = self.formatException(record.exc_info)
        return json.dumps(log_data)

def setup_logging(app_name: str, log_level: str = 'INFO'):
    logger = logging.getLogger(app_name)
    logger.setLevel(getattr(logging, log_level))
    
    # Console handler
    console = logging.StreamHandler()
    console.setFormatter(JSONFormatter())
    logger.addHandler(console)
    
    # File handler with rotation
    file_handler = logging.handlers.RotatingFileHandler(
        f'{app_name}.log',
        maxBytes=10_000_000,  # 10MB
        backupCount=5
    )
    file_handler.setFormatter(JSONFormatter())
    logger.addHandler(file_handler)
    
    return logger

# Usage
logger = setup_logging('myapp', 'INFO')
logger.info('Application started', extra={'version': '1.0.0'})
```

</details>

<details>
<summary><strong>Q19: Explain Python virtual environments and package management</strong></summary>

**Creating Virtual Environments:**
```bash
# Using venv (built-in)
python -m venv myenv
source myenv/bin/activate  # Linux/macOS
myenv\Scripts\activate     # Windows

# Deactivate
deactivate
```

**Package Management with pip:**
```bash
# Install package
pip install requests

# Install specific version
pip install requests==2.28.0

# Install from requirements
pip install -r requirements.txt

# Generate requirements
pip freeze > requirements.txt

# Upgrade package
pip install --upgrade requests

# Uninstall
pip uninstall requests
```

**requirements.txt Best Practices:**
```
# Pin exact versions for reproducibility
requests==2.28.1
boto3==1.26.0
pyyaml==6.0

# Or use ranges
requests>=2.28,<3.0

# Development dependencies (use separate file)
# requirements-dev.txt
pytest>=7.0
black>=22.0
mypy>=0.990
```

**Using Poetry (modern approach):**
```bash
# Initialize project
poetry init

# Add dependency
poetry add requests
poetry add pytest --group dev

# Install all dependencies
poetry install

# Run script
poetry run python myscript.py

# Build and publish
poetry build
poetry publish
```

**pyproject.toml Example:**
```toml
[tool.poetry]
name = "mydevops-tool"
version = "1.0.0"
description = "DevOps automation tool"

[tool.poetry.dependencies]
python = "^3.9"
requests = "^2.28"
boto3 = "^1.26"
click = "^8.0"

[tool.poetry.group.dev.dependencies]
pytest = "^7.0"
black = "^22.0"

[tool.poetry.scripts]
mytool = "mydevops_tool.cli:main"
```

</details>

<details>
<summary><strong>Q20: Practical Python DevOps Scripts</strong></summary>

**AWS EC2 Instance Manager:**
```python
import boto3
from typing import List, Dict

class EC2Manager:
    def __init__(self, region: str = 'us-east-1'):
        self.ec2 = boto3.client('ec2', region_name=region)
    
    def list_instances(self, state: str = None) -> List[Dict]:
        filters = []
        if state:
            filters.append({'Name': 'instance-state-name', 'Values': [state]})
        
        response = self.ec2.describe_instances(Filters=filters)
        
        instances = []
        for reservation in response['Reservations']:
            for instance in reservation['Instances']:
                name = next(
                    (t['Value'] for t in instance.get('Tags', []) if t['Key'] == 'Name'),
                    'N/A'
                )
                instances.append({
                    'id': instance['InstanceId'],
                    'name': name,
                    'state': instance['State']['Name'],
                    'type': instance['InstanceType'],
                    'public_ip': instance.get('PublicIpAddress', 'N/A')
                })
        return instances
    
    def start_instances(self, instance_ids: List[str]):
        self.ec2.start_instances(InstanceIds=instance_ids)
    
    def stop_instances(self, instance_ids: List[str]):
        self.ec2.stop_instances(InstanceIds=instance_ids)
```

**Kubernetes Pod Monitor:**
```python
from kubernetes import client, config
from kubernetes.client.rest import ApiException
import time

def monitor_pods(namespace: str = 'default'):
    config.load_kube_config()
    v1 = client.CoreV1Api()
    
    while True:
        try:
            pods = v1.list_namespaced_pod(namespace)
            print(f"\n{'Pod Name':<40} {'Status':<15} {'Restarts':<10}")
            print("-" * 65)
            
            for pod in pods.items:
                name = pod.metadata.name
                status = pod.status.phase
                restarts = sum(
                    cs.restart_count 
                    for cs in (pod.status.container_statuses or [])
                )
                
                if status != 'Running':
                    status = f"⚠️ {status}"
                elif restarts > 0:
                    status = f"⚠️ {status} ({restarts} restarts)"
                else:
                    status = f"✓ {status}"
                
                print(f"{name:<40} {status:<15} {restarts:<10}")
            
            time.sleep(10)
            
        except ApiException as e:
            print(f"Error: {e}")
            break
        except KeyboardInterrupt:
            break
```

**Slack Alert Sender:**
```python
import requests
import os
from datetime import datetime

class SlackNotifier:
    def __init__(self, webhook_url: str = None):
        self.webhook_url = webhook_url or os.environ.get('SLACK_WEBHOOK_URL')
    
    def send_message(self, message: str, channel: str = None):
        payload = {'text': message}
        if channel:
            payload['channel'] = channel
        
        response = requests.post(self.webhook_url, json=payload)
        response.raise_for_status()
    
    def send_deployment_notification(
        self, 
        app: str, 
        version: str, 
        environment: str,
        status: str
    ):
        color = '#36a64f' if status == 'success' else '#ff0000'
        
        payload = {
            'attachments': [{
                'color': color,
                'title': f'Deployment {status.upper()}',
                'fields': [
                    {'title': 'Application', 'value': app, 'short': True},
                    {'title': 'Version', 'value': version, 'short': True},
                    {'title': 'Environment', 'value': environment, 'short': True},
                    {'title': 'Time', 'value': datetime.now().strftime('%Y-%m-%d %H:%M:%S'), 'short': True}
                ]
            }]
        }
        
        requests.post(self.webhook_url, json=payload)

# Usage
notifier = SlackNotifier()
notifier.send_deployment_notification('myapp', 'v1.2.3', 'production', 'success')
```

</details>

---

## Behavioral Questions

<!-- Add your questions and answers here -->
