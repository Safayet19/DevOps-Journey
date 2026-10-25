<h1 align="center">GitHub Actions - Practical CI/CD Guide</h1>

<p align="center">A practical reference for workflow YAML, automated testing, Docker builds, deployments, and troubleshooting.</p>

---

## Table of Contents

1. [GitHub Actions Basics](#1-github-actions-basics)
2. [Workflow Structure](#2-workflow-structure)
3. [Workflow Triggers](#3-workflow-triggers)
4. [Jobs and Steps](#4-jobs-and-steps)
5. [Variables, Secrets and Permissions](#5-variables-secrets-and-permissions)
6. [Build and Test Pipelines](#6-build-and-test-pipelines)
7. [Docker Automation](#7-docker-automation)
8. [Artifacts and Caching](#8-artifacts-and-caching)
9. [CI/CD Deployment](#9-cicd-deployment)
10. [Troubleshooting](#10-troubleshooting)

---

## 1. GitHub Actions Basics

GitHub Actions automates tasks when repository events occur. **CI (Continuous Integration)** usually means automatically checking, building, and testing code. **CD (Continuous Delivery/Deployment)** prepares or releases tested changes.

| Term | Meaning | Practical example |
|---|---|---|
| Workflow | Automation described in a YAML file | `node-ci.yml` |
| Event/trigger | What starts a workflow | A push to `main` |
| Job | A collection of steps on one runner | `test` |
| Step | One action or shell command | `run: npm ci` |
| Runner | Computer that executes a job | `ubuntu-latest` |
| Action | Reusable automation component | `actions/checkout@v7` |
| Artifact | File saved from a workflow run | Compiled `dist/` directory |
| Cache | Reusable dependencies for faster jobs | npm download cache |
| Secret | Encrypted value made available to workflows | `secrets.DEPLOY_TOKEN` |

**Workflow location:** GitHub only runs YAML workflow files placed inside `.github/workflows/` in a repository. A README is documentation; code blocks inside a README do **not** execute.

### How to use the examples

1. Select an example suitable for your actual project.
2. Create its named file inside `.github/workflows/` (the folder is **required** for execution).
3. Commit and push the file to GitHub.
4. Open the repository's **Actions** tab, select the workflow, and inspect the run's jobs and logs.

**Do not add all examples to an unrelated project.** The Node.js, Spring Boot, Docker, and GitHub Pages examples each have different prerequisites, listed below.

## 2. Workflow Structure

A minimal complete workflow, matching the first kind of workflow we practiced:

**File:** `.github/workflows/hello.yml`

```yaml
name: Hello GitHub Actions

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read

jobs:
  greeting:
    runs-on: ubuntu-latest
    steps:
      - name: Say hello
        run: echo "Hello from GitHub Actions"
      - name: Show runner system
        run: uname -a
```

| YAML field | Exact purpose |
|---|---|
| `name: Hello GitHub Actions` | Display name in the Actions tab. |
| `on.push.branches: [main]` | Start when changes are pushed to `main`. |
| `workflow_dispatch:` | Allow manual runs from the Actions tab (workflow must exist on the default branch). |
| `permissions.contents: read` | Give `GITHUB_TOKEN` read-only access to repository contents. |
| `jobs.greeting` | Define a job named `greeting`. |
| `runs-on: ubuntu-latest` | Use a GitHub-hosted Ubuntu runner. |
| `steps` | List the steps executed in order within this job. |
| `run: echo ...` | Execute a shell command on the runner. |

**Important YAML rule:** One `-` begins one step. `name` and `run` must be properties of the **same** step when they belong together:

```yaml
# Correct: ONE step with a name and a command
steps:
  - name: Say hello
    run: echo "Hello"
```

```yaml
# Wrong: TWO separate steps; the first has no run or uses
steps:
  - name: Say hello
  - run: echo "Hello"
```

## 3. Workflow Triggers

A workflow can run automatically or manually. The following is a **complete `on:` block** that you may substitute into a workflow such as `hello.yml` above (only keep triggers you need):

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:
  schedule:
    - cron: '17 3 * * 1'
```

| Trigger | What it does | Concrete example |
|---|---|---|
| `push` | Starts when commits are pushed | `branches: [main]` restricts it to `main`. |
| `pull_request` | Checks proposed changes targeting a branch | `branches: [main]` checks PRs targeting `main`. |
| `workflow_dispatch` | Enables the **Run workflow** button | `workflow_dispatch:` needs no extra fields. |
| `schedule` | Runs using a cron schedule | `17 3 * * 1` is every Monday at 03:17 UTC by default. |

Scheduled runs use the default branch and can be delayed. Avoid scheduling at minute `0` if reliable timing matters. Use `branches: [main]` only when your branch really is named `main`.

## 4. Jobs and Steps

| Field/option | Complete usage example | Meaning |
|---|---|---|
| `runs-on` | `runs-on: ubuntu-latest` | Select the runner OS. |
| `uses` | `uses: actions/checkout@v7` | Download the current repository code onto the runner. |
| `run` | `run: node --version` | Execute a command. |
| `with` | `node-version: '24'` under `with:` for `actions/setup-node@v7` | Pass configuration to an action. |
| `needs` | `needs: build` | Wait for the `build` job to succeed before starting this job. |
| `if` | `if: github.event_name == 'push'` | Run a step/job only if its condition is true. |
| `env` | `CI: 'true'` under `env:` | Set an environment variable. |
| `timeout-minutes` | `timeout-minutes: 15` | Stop a job after 15 minutes. |
| `working-directory` | `working-directory: backend` under `run` | Run that step in the `backend/` folder. |

**`uses` vs. `run`:** `uses` invokes an existing Action; `run` executes commands. A single step should use one or the other, not both.

**Multiple shell commands in one step:**

```yaml
- name: Inspect files
  run: |
    pwd
    ls -la
    git status --short
```

This snippet is one step to place under `jobs.<job-name>.steps`, not a standalone workflow.

## 5. Variables, Secrets and Permissions

| Feature | Exact syntax | Explanation |
|---|---|---|
| Workflow variable | `env: { APP_ENV: test }` | Makes `APP_ENV` available to steps in scope. |
| Repository variable | `${{ vars.API_URL }}` | Reads a value configured in Actions **Variables**. |
| Repository secret | `${{ secrets.DEPLOY_TOKEN }}` | Reads an encrypted secret configured in Actions **Secrets**. |
| Shell variable | `echo "$APP_ENV"` | Accesses `APP_ENV` inside a Linux `run` script. |
| Built-in token | `${{ secrets.GITHUB_TOKEN }}` | GitHub-generated token, with workflow-defined permissions. |
| Read-only source access | `permissions: { contents: read }` | Minimum permission for normal checkout. |
| Publish container | `permissions: { contents: read, packages: write }` | Allows GHCR publishing. |

Set a custom secret at **Repository > Settings > Secrets and variables > Actions > New repository secret**. For example, create a secret named `DEPLOY_TOKEN`; do **not** put its actual value in YAML or commit it to Git.

To check that a secret exists without printing it, put this complete step under a job's `steps:` list:

```yaml
- name: Check deployment secret
  env:
    DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
  run: |
    if [ -z "$DEPLOY_TOKEN" ]; then
      echo "DEPLOY_TOKEN is missing"
      exit 1
    fi
    echo "DEPLOY_TOKEN is configured"
```

**Security:** Use the fewest possible `GITHUB_TOKEN` permissions. A token available to a workflow is not automatically safe to expose to untrusted pull-request code. Do not print secret values in logs, and pin third-party Actions to reviewed commit SHAs in security-sensitive production workflows. Version tags are used below to keep the examples readable.

## 6. Build and Test Pipelines

### Example 2 - Node.js CI

**Use when your repository contains:** `package.json`, `package-lock.json`, an `index.js` entry file, and a working `npm test` script in `package.json`. For another entry filename, update the `node --check` command.

**File:** `.github/workflows/node-ci.yml`

```yaml
name: Node.js CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read

jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - name: Checkout source
        uses: actions/checkout@v7

      - name: Set up Node.js
        uses: actions/setup-node@v7
        with:
          node-version: '24'
          cache: npm

      - name: Install locked dependencies
        run: npm ci

      - name: Check JavaScript syntax
        run: node --check index.js

      - name: Run tests
        run: npm test
```

| Step | Result | If it fails |
|---|---|---|
| `npm ci` | Installs dependencies from the lockfile | Commit a matching `package-lock.json`. |
| `node --check index.js` | Parses the entry file without running the app | Fix syntax errors, or change the entry filename. |
| `npm test` | Executes your project's test script | Define a valid `test` script and fix failing tests. |
| `cache: npm` | Caches npm's downloaded packages | Requires a supported lockfile, usually `package-lock.json`. |

### Example 3 - Spring Boot / Maven CI

**Use when your repository contains:** `pom.xml`, `mvnw`, and the `.mvn/wrapper/` files. Choose the JDK version matching your project (Java 21 is used here). The tests must be runnable in CI without missing databases or external credentials.

**File:** `.github/workflows/spring-boot-ci.yml`

```yaml
name: Spring Boot CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    timeout-minutes: 20
    steps:
      - name: Checkout source
        uses: actions/checkout@v7

      - name: Set up Java 21
        uses: actions/setup-java@v6
        with:
          distribution: temurin
          java-version: '21'
          cache: maven

      - name: Build and test using Maven Wrapper
        run: bash ./mvnw -B -ntp verify
```

| Command/option | Purpose |
|---|---|
| `bash ./mvnw` | Execute the project's committed Maven Wrapper. |
| `-B` | Maven batch mode, useful for CI logs. |
| `-ntp` | Disable Maven transfer-progress output. |
| `verify` | Run the Maven lifecycle through verification, including tests unless the project disables them. |
| `cache: maven` | Cache dependencies downloaded by Maven. |

If integration tests require MongoDB or other services, configure those prerequisites rather than skipping the tests or adding passwords to the workflow.

## 7. Docker Automation

### Example 4 - Build on pull requests, publish to GitHub Container Registry on push

**Use when:** the repository has a working `Dockerfile` in its root. It builds on both push and pull requests, but **publishes only when pushing to `main`**. It does not deploy a container to a server.

**File:** `.github/workflows/docker.yml`

```yaml
name: Docker Build and Publish

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read
  packages: write

jobs:
  docker:
    runs-on: ubuntu-latest
    timeout-minutes: 30
    steps:
      - name: Checkout source
        uses: actions/checkout@v7

      - name: Set lowercase image name
        shell: bash
        run: echo "IMAGE_NAME=ghcr.io/${GITHUB_REPOSITORY,,}" >> "$GITHUB_ENV"

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v4

      - name: Log in to GitHub Container Registry
        if: github.event_name == 'push'
        uses: docker/login-action@v4
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and optionally push image
        uses: docker/build-push-action@v7
        with:
          context: .
          push: ${{ github.event_name == 'push' }}
          tags: ${{ env.IMAGE_NAME }}:latest
```

| Setting | Exact behavior |
|---|---|
| `${GITHUB_REPOSITORY,,}` | Lowercases the `owner/repo` name for the container tag. |
| `packages: write` | Allows the workflow token to publish to GHCR. |
| `docker/setup-buildx-action@v4` | Sets up the Docker builder. |
| `push: ${{ github.event_name == 'push' }}` | `true` on push; `false` on pull requests. |
| `tags: ...:latest` | Tags the image as `ghcr.io/owner/repo:latest` (lowercase). |

**Expected result:** On a successful push to `main`, inspect your GitHub repository/user **Packages** area for the newly published container. Some organizations or pre-existing packages may require package access configuration. Do not assume the image is automatically public.

## 8. Artifacts and Caching

**Artifacts** store outputs from a run (for example, the compiled website). **Caching** reduces repeated dependency downloads; it is not a replacement for saved build artifacts.

| Need | Concrete configuration | Effect |
|---|---|---|
| Cache npm downloads | `cache: npm` in `actions/setup-node@v7` | Reuses npm cache across runs based on lockfile information. |
| Cache Maven downloads | `cache: maven` in `actions/setup-java@v6` | Reuses Maven dependency downloads. |
| Upload a build folder | `actions/upload-artifact@v7` with `path: dist/` | Stores `dist/` from the current job. |
| Retrieve it in another job | `actions/download-artifact@v8` with the same artifact name | Downloads the saved build output. |
| Artifact retention | `retention-days: 7` for upload-artifact | Retains that artifact for seven days, subject to repository policy. |

The complete pipeline in the next section demonstrates actual upload and download of an artifact across two jobs. In the Actions tab, open the workflow run and look for its **Artifacts** section.

## 9. CI/CD Deployment

### Example 5 - Build, test and deploy a static Node.js website to GitHub Pages

**Use when your repository contains:** a Node.js static website project with `package.json`, `package-lock.json`, a working `npm test` script, and an `npm run build` script that creates `dist/index.html` (such as a configured Vite project). This deploys **static files only**; it does not host a Spring Boot or Express server.

**One-time setup:** Open **Repository > Settings > Pages > Build and deployment > Source**, and choose **GitHub Actions**. If the website is served under a repository subpath, configure your build tool's base path accordingly.

**File:** `.github/workflows/deploy-pages.yml`

```yaml
name: Test, Build and Deploy Pages

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - name: Checkout source
        uses: actions/checkout@v7

      - name: Set up Node.js
        uses: actions/setup-node@v7
        with:
          node-version: '24'
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Build static website
        run: npm run build

      - name: Verify build output
        run: test -f dist/index.html

      - name: Save build output
        uses: actions/upload-artifact@v7
        with:
          name: site-dist
          path: dist/
          if-no-files-found: error
          retention-days: 7

  deploy:
    needs: build
    if: github.ref == 'refs/heads/main' && github.event_name != 'pull_request'
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Download tested build
        uses: actions/download-artifact@v8
        with:
          name: site-dist
          path: dist

      - name: Configure GitHub Pages
        uses: actions/configure-pages@v5

      - name: Package files for Pages
        uses: actions/upload-pages-artifact@v4
        with:
          path: dist/

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

| Part | Why it is there |
|---|---|
| `build` job | Installs dependencies, runs tests, builds website and checks the output. |
| `actions/upload-artifact@v7` | Saves the tested `dist/` folder so another job can use it. |
| `needs: build` | Deployment starts only if the build succeeds. |
| `if: ...main...` | Prevents PR branches from deploying and limits deployments to `main`. |
| `actions/download-artifact@v8` | Restores exactly the build output produced in the earlier job. |
| `pages: write` and `id-token: write` | Allows GitHub Pages deployment. |
| `actions/deploy-pages@v4` | Publishes the static site to GitHub Pages. |

**Expected result:** Pull requests are tested and built but not deployed. Pushes to `main` and manual runs from `main` build, test, then deploy. When successful, the deploy job displays the Pages URL.

## 10. Troubleshooting

| Problem | Likely reason | What to check or fix |
|---|---|---|
| Workflow not appearing in Actions | File is only in README or in the wrong folder | Put actual YAML at `.github/workflows/name.yml`. |
| Manual **Run workflow** button missing | No `workflow_dispatch`, or workflow absent from default branch | Add `workflow_dispatch:` and commit the workflow to the default branch. |
| Workflow did not run on push | Wrong branch name or event filter | Check the `on.push.branches` entry against the pushed branch. |
| `Invalid workflow file` | YAML spacing, wrong indentation or invalid schema | Align `name`, `run`, and `uses` correctly; see Section 2. |
| Step has neither `run` nor `uses` | A `- name:` step was separated from its command | Put `run:` under the same `- name:` item. |
| `npm ci` failed | No lockfile or mismatched dependencies | Commit `package-lock.json` and regenerate it if needed. |
| `npm test` failed | Test script missing or test failures | Inspect `package.json` scripts and the test output. |
| `mvnw: No such file` | Maven Wrapper absent from the repository | Commit `mvnw` and `.mvn/wrapper`, or adapt to `mvn` if appropriate. |
| `Permission denied` while publishing | Workflow token lacks permissions or package access | Verify `packages: write` and GHCR package access. |
| Missing secret | Secret name does not exist or is not available to this run | Configure the exact name in Settings; don't print the value. |
| Pages deploy fails | Pages source not set to Actions, bad permissions, or no build output | Select GitHub Actions in Pages settings and check `dist/index.html`. |
| Job skipped unexpectedly | `if` expression evaluated to false | Inspect event name, branch ref, and dependent job results. |

### Fast debugging workflow

1. Open **Actions > workflow run > failed job > failed step**.
2. Read the error **above** the final `Process completed with exit code 1` line.
3. Confirm the necessary source files exist in GitHub, not only on your computer.
4. Check whether the workflow was triggered by `push`, `pull_request`, or `workflow_dispatch`.
5. Fix one issue, commit, and run again. Avoid adding secrets or disabling tests just to make the build green.

### Common GitHub Actions keywords

| Keyword | Example | Meaning |
|---|---|---|
| `continue-on-error` | `continue-on-error: true` | Do not fail the job when this step fails; use sparingly. |
| `if` | `if: always()` | Run even when earlier steps failed, subject to job state and conditions. |
| `env` | `env: { APP_ENV: test }` | Set environment variables. |
| `needs` | `needs: build` | Run after the named job finishes successfully. |
| `with` | `with: { node-version: '24' }` | Supply parameters to a reusable action. |

---

## Practice Checklist

- [ ] Add `hello.yml` and verify the first workflow succeeds.
- [ ] Fix deliberately incorrect YAML indentation and compare the result.
- [ ] Trigger one workflow manually and another with a push.
- [ ] Run the Node.js CI workflow on a project with working tests.
- [ ] Run the Spring Boot CI workflow on a project with a Maven Wrapper.
- [ ] Build a Docker image on a PR and publish it from `main`.
- [ ] Upload and download a build artifact using two separate jobs.
- [ ] Deploy a **static** project to GitHub Pages after tests pass.

**References:** [GitHub Actions documentation](https://docs.github.com/en/actions) | [Workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax) | [GitHub Pages custom workflows](https://docs.github.com/en/pages/getting-started-with-github-pages/using-custom-workflows-with-github-pages) | [Container registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
