<h1 align="center">Jenkins - Practical CI/CD & Pipeline Guide</h1>

<p align="center">A practical reference for Jenkins setup, pipelines, GitHub integration, automated testing, Docker, deployment, and troubleshooting.</p>

---

This guide uses **complete commands and Jenkinsfiles**, with clear prerequisites. It covers both **Windows and Linux** because their Pipeline shell commands differ.

> **Important:** This is documentation. Jenkins executes a file named `Jenkinsfile` when it is configured in a job, not the code blocks inside this README. Start with the first pipeline and use only the examples that match your project.

## Table of Contents

1. [Jenkins Basics](#1-jenkins-basics)
2. [Installation and Service Management](#2-installation-and-service-management)
3. [Jobs and Your First Pipeline](#3-jobs-and-your-first-pipeline)
4. [Jenkinsfile Syntax and OS Commands](#4-jenkinsfile-syntax-and-os-commands)
5. [GitHub Integration and Triggers](#5-github-integration-and-triggers)
6. [Variables, Parameters and Credentials](#6-variables-parameters-and-credentials)
7. [Node.js CI Pipeline](#7-nodejs-ci-pipeline)
8. [Spring Boot Maven CI Pipeline](#8-spring-boot-maven-ci-pipeline)
9. [Docker Build and Publish Pipeline](#9-docker-build-and-publish-pipeline)
10. [Complete CI/CD Deployment Pipeline](#10-complete-cicd-deployment-pipeline)
11. [Agents, Plugins, Artifacts and Test Reports](#11-agents-plugins-artifacts-and-test-reports)
12. [Troubleshooting and Maintenance](#12-troubleshooting-and-maintenance)

---

## 1. Jenkins Basics

Jenkins is an automation server. It can fetch code, run builds and tests, package software, and deploy successful builds.

| Term | Explanation | Practical example |
|---|---|---|
| Controller | Manages Jenkins configuration, jobs, and scheduling | Your Jenkins web server |
| Agent | Machine that executes a Pipeline's steps | A Windows or Linux build machine |
| Executor | An execution slot for running builds on a node | One executor runs one build at a time |
| Job | Saved Jenkins task | `spring-boot-ci` |
| Pipeline | Job defined as stages and steps | Checkout -> Test -> Build -> Deploy |
| Stage | Named phase of a Pipeline | `Test` |
| Step | Individual action inside a stage | `bat 'mvnw.cmd test'` |
| Jenkinsfile | Pipeline-as-code file, usually in the repository root | `Jenkinsfile` |
| Workspace | Files checked out and used for a job on an agent | Source code and build output |
| Plugin | Extension that provides integration or steps | Git, Pipeline, JUnit |
| Credentials | Jenkins-managed authentication information | Git token or SSH private key |

**CI** means automatically validating code changes (such as build and test). **CD** means preparing or deploying validated changes. A successful `echo` step alone is **not** a real build or deployment.

## 2. Installation and Service Management

### Windows (PowerShell)

For a current Jenkins release, install a **supported Java 21 or later runtime**, then use the official **Jenkins LTS Windows installer**. The installer can run Jenkins as a Windows service. The service account must have the **Log on as a service** right. Prefer a limited-purpose account instead of `LocalSystem`.

| Task | Complete PowerShell command | Explanation |
|---|---|---|
| Check Java | `java -version` | Confirm a supported Java runtime is available. |
| Check Jenkins service | `Get-Service -Name Jenkins` | Show current service status. |
| Check service identity | `Get-CimInstance Win32_Service -Filter "Name='jenkins'" \| Select-Object Name, StartName, State, PathName` | Show the Windows account and executable path. |
| Start Jenkins | `Start-Service -Name Jenkins` | Start the service; elevated permissions may be required. |
| Stop Jenkins | `Stop-Service -Name Jenkins` | Stop the service and interrupt running work. |
| Restart Jenkins | `Restart-Service -Name Jenkins` | Restart Jenkins; use after configuration/plugin changes when needed. |
| Check port 8080 | `Get-NetTCPConnection -LocalPort 8080 -State Listen -ErrorAction SilentlyContinue` | See if something is listening on the usual web port. |
| Open local Jenkins | `Start-Process 'http://localhost:8080'` | Open Jenkins in your browser. |

**First login:** Open `http://localhost:8080` (or your chosen port), locate `initialAdminPassword` under your installation's **JENKINS_HOME/secrets** or the path shown by the setup wizard, unlock Jenkins, install suggested plugins, and create an administrator account. Paths can differ between Windows installations; do not assume your current PowerShell session has the service's `JENKINS_HOME` environment variable.

### Ubuntu / Debian (Bash)

Use the Jenkins **LTS** package repository. The official installation instructions currently use the 2026 repository key; check the linked documentation if its location changes.

```bash
sudo apt update
sudo apt install -y fontconfig openjdk-21-jre wget
sudo install -d -m 0755 /etc/apt/keyrings
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
echo 'deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/' | \
  sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install -y jenkins
sudo systemctl enable --now jenkins
sudo systemctl status jenkins
```

| Task | Complete Bash command | Explanation |
|---|---|---|
| Check installed Java | `java -version` | Verify Java is available. |
| Check Jenkins status | `sudo systemctl status jenkins` | Show whether the service is running. |
| Start Jenkins | `sudo systemctl start jenkins` | Start the service. |
| Stop Jenkins | `sudo systemctl stop jenkins` | Stop the service. |
| Restart Jenkins | `sudo systemctl restart jenkins` | Restart the service. |
| Enable on boot | `sudo systemctl enable jenkins` | Start Jenkins automatically after reboot. |
| Inspect service logs | `sudo journalctl -u jenkins -n 100 --no-pager` | Show the last 100 Jenkins service log entries. |
| Follow service logs | `sudo journalctl -u jenkins -f` | Stream new Jenkins service logs. |
| Read initial password | `sudo cat /var/lib/jenkins/secrets/initialAdminPassword` | Standard Linux package path; handle the password privately. |

> Jenkins usually listens on port **8080**. `localhost` works only from the same computer. Do not expose an unprotected Jenkins admin interface to the public internet.

## 3. Jobs and Your First Pipeline

### Create a Pipeline job

1. Open **Jenkins Dashboard -> New Item**.
2. Enter `hello-jenkins`; choose **Pipeline**; click **OK**.
3. Under **Pipeline -> Definition**, select **Pipeline script**.
4. Paste the complete example below, select **Save**, then **Build Now**.
5. Open the build -> **Console Output** to confirm success.

### Example 1 - First Pipeline (works on Windows or Linux)

**Requirement:** A connected Jenkins agent or controller executor. No Git repository, Node.js, or Maven needed for this demonstration.

```groovy
pipeline {
    agent any

    options {
        timeout(time: 10, unit: 'MINUTES')
    }

    stages {
        stage('Information') {
            steps {
                echo "Job: ${env.JOB_NAME}"
                echo "Build number: ${env.BUILD_NUMBER}"
                script {
                    if (isUnix()) {
                        sh 'uname -s'
                    } else {
                        bat 'ver'
                    }
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Demo build step only - no application compiled.'
            }
        }

        stage('Test') {
            steps {
                echo 'Demo test step only - no tests executed.'
            }
        }
    }

    post {
        success { echo 'Pipeline completed successfully.' }
        failure { echo 'Pipeline failed. Check Console Output.' }
    }
}
```

**Expected result:** Three stages appear and Jenkins reports `SUCCESS`. To build a real project, use the Node.js or Spring Boot examples below.

### Keep Jenkinsfile in Git (recommended)

For a real project, add **one file named `Jenkinsfile` in the root** of that project, commit it, then configure the Jenkins job with **Pipeline -> Definition -> Pipeline script from SCM**. The README itself is not a runnable Jenkinsfile.

## 4. Jenkinsfile Syntax and OS Commands

### Declarative Pipeline essentials

| Keyword / step | Complete usage example | Meaning |
|---|---|---|
| `pipeline` | See **Example 1** for a complete `pipeline { ... }` | Encloses the complete Declarative Pipeline. |
| `agent` | `agent any` | Use any available executor. |
| `agent` with label | `agent { label 'linux' }` | Require an agent carrying the `linux` label. |
| `stages` | `stages { stage('Test') { steps { echo 'Testing' } } }` | Contains named phases. |
| `steps` | `steps { echo 'Hello' }` | Contains actions inside a stage. |
| `sh` | `sh 'pwd'` | Run a shell command on **Linux/macOS** agents. |
| `bat` | `bat 'cd'` | Run a CMD command on **Windows** agents. |
| `powershell` | `powershell 'Get-Location'` | Run Windows PowerShell on a Windows agent. |
| `echo` | `echo "Build #${env.BUILD_NUMBER}"` | Print text to the Jenkins console. |
| `script` | `script { if (isUnix()) { sh 'pwd' } else { bat 'cd' } }` | Use Groovy control flow when required. |
| `when` | `when { branch 'main' }` | Execute a stage only on `main` in a **Multibranch Pipeline**. |
| `post` | `post { always { echo 'Finished' } }` | Run actions after a Pipeline/stage, regardless of outcome. |
| `timeout` | `options { timeout(time: 20, unit: 'MINUTES') }` | Stop a build that exceeds the time limit. |
| `disableConcurrentBuilds` | `options { disableConcurrentBuilds() }` | Prevent overlapping builds of the same job. |
| `checkout` | `checkout scm` | Check out the repository configured for this SCM-based job. |
| `error` | `error 'Missing required file'` | Mark the build failed with a clear reason. |

**Checkout note:** The CI/CD examples use `skipDefaultCheckout(true)` and an explicit `checkout scm` stage so Jenkins does not fetch the same source twice.

**Use the right OS-specific step:** `sh` needs a Unix-like shell. On Windows, use `bat` or `powershell`, unless you deliberately configured a compatible shell. A local Git Bash installation does not make `sh` automatically work for a Jenkins Windows service.

### Declarative vs Scripted

| Style | Structure | Recommendation |
|---|---|---|
| Declarative | `pipeline { agent ...; stages { ... } }` | Preferred for the examples in this guide. |
| Scripted | `node { stage('...') { ... } }` | Useful when custom Groovy control flow is needed. |

Complete **Scripted** Hello World example:

```groovy
node {
    stage('Hello') {
        echo 'Hello from a Scripted Pipeline'
    }
}
```

Do not mix Scripted `node { ... }` as the outer wrapper with Declarative `pipeline { ... }` for the same Jenkinsfile.

## 5. GitHub Integration and Triggers

### Connect an existing GitHub repository

1. Install/verify **Pipeline**, **Git**, and **GitHub** plugins from **Manage Jenkins -> Plugins**.
2. In the repository being built, commit a root-level file named `Jenkinsfile`.
3. Create a Jenkins **Pipeline** job.
4. Choose **Pipeline script from SCM -> Git**.
5. Enter the real GitHub clone URL, such as `https://github.com/OWNER/REPO.git`.
6. Set **Branch Specifier** to `*/main` if your branch is `main`.
7. Set **Script Path** to `Jenkinsfile`, then **Save -> Build Now**.
8. For private repositories, add an authorized Jenkins credential instead of a password in the URL.

### Choose ONE automatic trigger

| Trigger type | Jenkins configuration / syntax | Exact behavior |
|---|---|---|
| Manual | Click **Build Now** | Runs only when requested. |
| Periodic build | `triggers { cron('H 2 * * 1-5') }` | Run once during the 02:00 hour on weekdays; `H` picks a stable minute. |
| Poll Git changes | `triggers { pollSCM('H/5 * * * *') }` | Check SCM roughly every five minutes; build only if changes are found. |
| GitHub webhook | GitHub trigger in job + `https://YOUR-JENKINS-HOST/github-webhook/` | GitHub notifies Jenkins after a push. |

The `triggers { ... }` examples belong **once at the top level inside a Declarative `pipeline` block**, not in a `steps` block. Do not use cron and polling together unless you intentionally need both.

**Important:** GitHub cannot send a webhook to `http://localhost:8080` on your laptop. For a local Jenkins installation, use **SCM polling** first. For webhooks, use a properly secured reachable Jenkins endpoint or an approved webhook forwarding service. The GitHub webhook URL normally ends with `/github-webhook/` (including the trailing slash).

### Jenkins cron basics

| Pattern | Meaning |
|---|---|
| `H/5 * * * *` | Approximately every 5 minutes, spread across jobs. |
| `H H * * *` | Once per day at a stable, Jenkins-chosen time. |
| `H 9 * * 1-5` | Once during the 09:00 hour, Monday-Friday. |
| `H 2 * * 0` | Once during the 02:00 hour on Sunday. |

`H` is Jenkins' **hashed scheduling value**, not a normal Unix cron minute. The configured Jenkins timezone controls schedule interpretation.

## 6. Variables, Parameters and Credentials

| Feature | Complete Jenkinsfile syntax | Explanation |
|---|---|---|
| Environment variable | `environment { APP_ENV = 'test' }` | Makes a variable available to pipeline steps. |
| Read Jenkins variable | `echo "Build #${env.BUILD_NUMBER}"` | Accesses a built-in Jenkins environment value. |
| Linux shell variable | `sh 'echo "$APP_ENV"'` | Expands the variable in the shell. |
| Windows CMD variable | `bat 'echo %APP_ENV%'` | Expands the variable in CMD. |
| Windows PowerShell variable | `powershell 'Write-Output $env:APP_ENV'` | Accesses an environment variable in PowerShell. |
| Boolean parameter | `parameters { booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests') }` | Adds a checkbox to **Build with Parameters**. |
| Check a parameter | `when { expression { params.RUN_TESTS } }` | Execute a stage only when the checkbox is selected. |
| Simple secret binding | `withCredentials([string(credentialsId: 'api-token', variable: 'API_TOKEN')]) { ... }` | Exposes a Jenkins secret temporarily to wrapped steps. |

### Create a credential

Go to **Manage Jenkins -> Credentials**, select an appropriate store and scope, and **Add Credentials**. Common kinds are **Secret text**, **Username with password**, and **SSH Username with private key**. Give credentials meaningful IDs, such as `dockerhub-account` or `deploy-ssh-key`.

Example of a complete Pipeline that checks whether a configured secret is nonempty **without printing its value**:

```groovy
pipeline {
    agent any
    stages {
        stage('Check credential') {
            steps {
                withCredentials([string(credentialsId: 'api-token', variable: 'API_TOKEN')]) {
                    script {
                        if (!env.API_TOKEN?.trim()) {
                            error 'The api-token secret is empty.'
                        }
                    }
                    echo 'The credential was provided.'
                }
            }
        }
    }
}
```

**Prerequisite:** Create a Secret text credential with ID `api-token` first. Never paste real tokens or private keys into a Jenkinsfile, log, README, or Git commit. Avoid making deployment credentials available to untrusted pull-request builds.

## 7. Node.js CI Pipeline

### Example 2 - Install dependencies and run tests

**Requirements:** A Git repository with `package.json`, `package-lock.json`, and a valid `npm test` script. The selected Jenkins agent must have `node` and `npm` on its **service/agent PATH**. This Jenkinsfile works on either Windows or Linux.

**File in application repository:** `Jenkinsfile`

```groovy
pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        timeout(time: 15, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Node.js') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'node --version && npm --version'
                    } else {
                        bat 'node --version && npm --version'
                    }
                }
            }
        }

        stage('Install') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'npm ci'
                    } else {
                        bat 'npm ci'
                    }
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'npm test'
                    } else {
                        bat 'npm test'
                    }
                }
            }
        }
    }

    post {
        success { echo 'Node.js CI passed.' }
        failure { echo 'Node.js CI failed. Inspect the failed stage.' }
    }
}
```

| Step | What success confirms | Common failure |
|---|---|---|
| `node --version` | Node.js is available to the Jenkins agent | `node` not on the service account's PATH |
| `npm ci` | Dependencies match the committed lockfile | Missing or inconsistent `package-lock.json` |
| `npm test` | The project's configured tests pass | Missing test script or failed assertions |

For a project with a real `build` script, an additional stage can run `npm run build`; do **not** add that command to an Express project without such a script.

## 8. Spring Boot Maven CI Pipeline

### Example 3 - Build, run tests, and archive JAR

**Requirements:** Java matching your project (Java 21 in this example); Git repository containing `pom.xml`, `mvnw`, `mvnw.cmd`, `.mvn/wrapper/`; and tests that can run with available CI configuration. No global Maven tool is required because this uses the committed Maven Wrapper.

**File in Spring Boot repository:** `Jenkinsfile`

```groovy
pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        timeout(time: 25, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Java') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'java -version'
                    } else {
                        bat 'java -version'
                    }
                }
            }
        }

        stage('Build and Test') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'bash ./mvnw -B -ntp verify'
                    } else {
                        bat 'mvnw.cmd -B -ntp verify'
                    }
                }
            }
        }
    }

    post {
        always {
            junit testResults: 'target/surefire-reports/TEST-*.xml', allowEmptyResults: true
        }
        success {
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            echo 'Spring Boot JAR archived.'
        }
        failure {
            echo 'Maven failed. Read the error earlier in Console Output.'
        }
    }
}
```

| Command / step | Meaning |
|---|---|
| `bash ./mvnw -B -ntp verify` | Linux: use Maven Wrapper in batch mode without transfer progress, running Maven through `verify`. |
| `mvnw.cmd -B -ntp verify` | Windows equivalent of the Maven Wrapper command. |
| `junit ...` | Publish XML test reports when available; `allowEmptyResults: true` avoids a second error if the build stops before tests create reports. |
| `archiveArtifacts ...` | Save packaged JAR files with the successful Jenkins build. |

If the project needs MongoDB, environment variables, or integration services, configure those prerequisites in Jenkins rather than disabling tests. **A successful build does not mean deployment.**

## 9. Docker Build and Publish Pipeline

### Example 4 - Build, inspect, and optionally publish an image

**Requirements:** A Linux Jenkins agent labelled `docker-linux`, Docker CLI + working Docker daemon available to that agent, and a project-root `Dockerfile`. If publishing, create a Jenkins **Username with password** credential with ID `dockerhub-account`; use your Docker Hub username and a suitable access token.

Before running, replace `YOUR_DOCKERHUB_USERNAME/my-app` with the actual lowercase Docker Hub image name. The **Publish** stage is off unless you explicitly select `PUBLISH_IMAGE` in **Build with Parameters**. Run this job only with trusted source code.

**File in Dockerized project:** `Jenkinsfile`

```groovy
pipeline {
    agent { label 'docker-linux' }

    parameters {
        booleanParam(name: 'PUBLISH_IMAGE', defaultValue: false,
                     description: 'Push to Docker Hub after successful build')
    }

    environment {
        IMAGE_NAME = 'YOUR_DOCKERHUB_USERNAME/my-app'
    }

    options {
        skipDefaultCheckout(true)
        timeout(time: 30, unit: 'MINUTES')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Docker') {
            steps {
                sh 'docker version'
            }
        }

        stage('Build Image') {
            steps {
                sh 'docker build -t "$IMAGE_NAME:$BUILD_NUMBER" .'
            }
        }

        stage('Verify Image') {
            steps {
                sh 'docker image inspect "$IMAGE_NAME:$BUILD_NUMBER" >/dev/null'
            }
        }

        stage('Publish Image') {
            when {
                expression { params.PUBLISH_IMAGE }
            }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-account',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        set +x
                        trap 'docker logout >/dev/null 2>&1 || true' EXIT
                        printf '%s' "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push "$IMAGE_NAME:$BUILD_NUMBER"
                    '''
                }
            }
        }
    }

    post {
        success { echo 'Docker pipeline completed.' }
        failure { echo 'Docker pipeline failed. Inspect Console Output.' }
    }
}
```

| Command / setting | Actual result |
|---|---|
| `docker version` | Checks both Docker CLI and server connectivity. |
| `docker build -t "$IMAGE_NAME:$BUILD_NUMBER" .` | Builds a uniquely numbered image from `Dockerfile`. |
| `docker image inspect ...` | Verifies the built image exists; **does not test application behavior**. |
| `PUBLISH_IMAGE = false` | Skips Docker Hub login and push. |
| `docker login ... --password-stdin` | Authenticates without putting a token directly in the command argument. |
| `docker push ...` | Publishes the tagged image to Docker Hub. |

**Windows note:** A Windows Jenkins service often cannot access Docker Desktop using the same settings as an interactive user. Check the actual service account, Docker daemon access, and agent OS; do not blindly replace `sh` with `bat` in this Linux-only example. Docker daemon access is highly privileged.

## 10. Complete CI/CD Deployment Pipeline

### Example 5 - Test, build and deploy a static website to a Linux server

**Requirements:** A **Multibranch Pipeline** job building a trusted `main` branch, a Jenkins Linux agent labelled `linux` with Node.js, npm, `rsync`, `ssh`, `ssh-agent`; a static web project with `package.json`, `package-lock.json`, working `npm test` and `npm run build`, producing `dist/index.html`; and an SSH private key stored as Jenkins credential **`deploy-ssh-key`**. Install the **SSH Agent** plugin.

Before running, replace **`deploy@192.0.2.10`** (documentation-only IP) and **`/var/www/site/`** with your real authorized server account and writable destination. Configure the server's public-key authorization and verify/add its trusted SSH host key to the Jenkins agent's `known_hosts`. Do **not** disable host-key verification. The server must already be configured to serve this directory (for example via Nginx).

**File in static website repository:** `Jenkinsfile`

```groovy
pipeline {
    agent { label 'linux' }

    options {
        skipDefaultCheckout(true)
        timeout(time: 25, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
                sh 'test -f dist/index.html'
                archiveArtifacts artifacts: 'dist/**', fingerprint: true
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sshagent(credentials: ['deploy-ssh-key']) {
                    sh '''
                        rsync -av \
                          -e 'ssh -o BatchMode=yes -o StrictHostKeyChecking=yes' \
                          dist/ deploy@192.0.2.10:/var/www/site/
                    '''
                }
            }
        }
    }

    post {
        success { echo 'Pipeline passed; main branch deployed when applicable.' }
        failure { echo 'Pipeline failed or deployment was unsuccessful.' }
    }
}
```

| Stage | Exact purpose |
|---|---|
| Checkout | Retrieves the commit being built. |
| Install | Installs locked Node.js dependencies. |
| Test | Runs the project's real automated tests. |
| Build | Creates static output and verifies `dist/index.html`. |
| Deploy | On **main** only, copies tested static files to the configured Linux web directory. |

**Important:** `when { branch 'main' }` is for **Multibranch Pipeline** jobs. This example deploys **static files**, not a Spring Boot or Express process, and it does not automatically configure Nginx. `rsync -av` updates files but does not delete stale remote files; deletion policies should be designed separately.

## 11. Agents, Plugins, Artifacts and Test Reports

### Essential plugins

| Plugin | When you need it | What it provides |
|---|---|---|
| Pipeline / Pipeline: Declarative | For all Jenkinsfiles in this guide | Pipeline definitions, stages, and steps. |
| Git | GitHub source checkout | Git SCM integration and `checkout scm`. |
| GitHub | GitHub webhook integration | GitHub project integrations and hook trigger. |
| Credentials Binding | Pipelines using tokens/passwords | `withCredentials`. |
| JUnit | Maven test reporting | `junit` Pipeline step. |
| SSH Agent | Remote deployment example | `sshagent` credential forwarding. |
| Docker Pipeline | Docker-based `agent { docker { ... } }` syntax | Running Pipeline stages inside containers (not required for plain `sh 'docker build ...'`). |

Install plugins from **Manage Jenkins -> Plugins**. Keep Jenkins core and plugins compatible; do not install plugins just because they exist.

### Artifacts and test reports

| Purpose | Complete Pipeline step | Meaning |
|---|---|---|
| Store Maven JARs | `archiveArtifacts artifacts: 'target/*.jar', fingerprint: true` | Make compiled JARs available from the build page. |
| Store built website | `archiveArtifacts artifacts: 'dist/**', fingerprint: true` | Archive generated static files. |
| Publish JUnit results | `junit 'target/surefire-reports/TEST-*.xml'` | Show test history and failures using XML test reports. |
| Save small build files temporarily | `stash name: 'site', includes: 'dist/**'` | Keep selected files for other stages in the same run. |
| Restore files in another stage | `unstash 'site'` | Retrieve files saved with `stash`. |

**Artifact vs stash:** Archived artifacts are retained with build history. `stash/unstash` transfers files between stages of the **same build** and is intended for smaller outputs.

### Node / agent checks

| Problem / task | Location or command | What to verify |
|---|---|---|
| No agent available | **Manage Jenkins -> Nodes** | Agent is online and has an executor. |
| Required label missing | `agent { label 'linux' }` | At least one available agent actually has label `linux`. |
| Check Linux tools | `sh 'git --version && java -version'` | Tools exist on the Linux agent's PATH. |
| Check Windows tools | `bat 'git --version && java -version'` | Tools exist on the Windows service/agent PATH. |
| Agent workspace | `echo "Workspace: ${env.WORKSPACE}"` | Prints the job's assigned workspace path. |
| Build logs | **Job -> Build number -> Console Output** | Start with the earliest meaningful error. |

For production installations, use dedicated agents rather than running untrusted builds on the controller. The user running the agent needs the filesystem, Git, Docker, Java, Node.js, and deployment permissions required by the specific job.

## 12. Troubleshooting and Maintenance

| Error / symptom | Common cause | Exact check or next step |
|---|---|---|
| `Jenkins service failed to start` | Invalid Java runtime, service identity, or occupied port | `java -version`; `Get-Service Jenkins`; inspect service logs and port 8080. |
| `Service logon failure` on Windows | Incorrect service account password or missing **Log on as a service** right | Inspect **Services -> Jenkins -> Properties -> Log On**; correct account permissions. |
| Unlock page does not open | Jenkins not running or wrong configured port | `Get-Service Jenkins` / `sudo systemctl status jenkins`; confirm the URL. |
| Plugin installation keeps failing | Outdated core, missing dependency, incompatible plugin version, or download failure | **Manage Jenkins -> Plugins**; read the exact dependency/version error; install compatible versions. |
| `No such DSL method` | Missing plugin or wrong Pipeline step name | Open **Jenkins URL/pipeline-syntax**; check plugin and step syntax. |
| `No nodes with the label ...` | Agent label does not exist or node offline | **Manage Jenkins -> Nodes**; verify `agent { label '...' }`. |
| `sh: not found` or shell step fails on Windows | `sh` is for Unix-like agents | Use `bat '...'` or `powershell '...'`. |
| `node` / `npm` / `mvn` not found | Tool missing from the **Jenkins service account** environment | Run version commands inside a Jenkins build, not only in your desktop terminal. |
| `npm ci` failed | Lockfile missing/mismatched | Commit correct `package-lock.json`; verify `npm ci` locally. |
| `mvnw.cmd` failed | Wrapper missing or Java/project configuration issue | Confirm `mvnw.cmd`, `.mvn/wrapper/`, and `pom.xml` exist. |
| `docker: command not found` | Docker CLI missing on selected agent | Run `docker version` on that Jenkins agent. |
| Docker daemon permission error | Jenkins agent account cannot access Docker daemon | Check Docker daemon and permissions; do not grant broad Docker access casually. |
| GitHub push does not trigger Jenkins | Webhook unreachable, wrong URL, or trigger not configured | Check webhook deliveries, job trigger, and the trailing `/github-webhook/`. |
| Localhost webhook never arrives | GitHub cannot reach a laptop's localhost | Use polling or a secured reachable endpoint. |
| Credentials not found | Credential ID/scope mismatch | Check the exact ID under **Manage Jenkins -> Credentials**. |
| SSH deployment fails | Wrong key, host key, server address, or file permissions | Verify key authorization, `known_hosts`, and destination write permission. |
| `junit` report missing | Tests did not produce XML or reports use different path | Inspect `target/surefire-reports/` in the project build. |
| Pipeline hangs | External process waiting for input or blocked agent | Add `timeout`, check the current step, and investigate logs. |

### Safe maintenance practices

- Back up **JENKINS_HOME** using a controlled, secure process. It includes job configuration and sensitive Jenkins material; restrict access to backups.
- Prefer **Jenkins LTS**, keep the controller and plugins compatible, and review plugin dependency errors before upgrades.
- Use **least-privilege** service accounts, agents, repository credentials, and deployment accounts.
- Protect deployment branches and credentials; do not let untrusted PR code access production secrets.
- Do not expose an administrative Jenkins endpoint without authentication and appropriate network protection.
- Prefer **Pipeline script from SCM** over storing large Pipelines only in the Jenkins UI.

### Practice checklist

- [ ] Install Jenkins and verify the Windows/Linux service is running.
- [ ] Create the first Pipeline job and inspect Console Output.
- [ ] Save a real `Jenkinsfile` in GitHub and load it using **Pipeline script from SCM**.
- [ ] Run a Pipeline manually and then configure either SCM polling or a webhook.
- [ ] Run an actual Node.js CI pipeline with a lockfile and tests.
- [ ] Build and test a Spring Boot project and archive its JAR.
- [ ] Build a Docker image and verify the image exists.
- [ ] Add credentials securely and test deployment only on an authorized target.
- [ ] Diagnose one failed build using its first relevant error.

---

## Official References

- [Jenkins documentation](https://www.jenkins.io/doc/)
- [Windows installation](https://www.jenkins.io/doc/book/installing/windows/)
- [Linux installation](https://www.jenkins.io/doc/book/installing/linux/)
- [Pipeline syntax](https://www.jenkins.io/doc/book/pipeline/syntax/)
- [Getting started with Pipeline](https://www.jenkins.io/doc/book/pipeline/getting-started/)
- [GitHub plugin and webhook](https://plugins.jenkins.io/github/)
- [Jenkins credentials](https://www.jenkins.io/doc/book/security/credentials/)
- [Managing Jenkins plugins](https://www.jenkins.io/doc/book/managing/plugins/)

*This README is a practical reference. Configure commands, agent labels, repositories, and deployment destinations for your actual environment before running them.*
