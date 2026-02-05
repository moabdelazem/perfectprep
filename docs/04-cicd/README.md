# CI/CD Pipelines

[← Back to Index](../../README.md)

This section covers Continuous Integration and Continuous Deployment including Jenkins, Azure DevOps, pipeline configuration, and best practices.

---

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

