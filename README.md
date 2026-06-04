## BuildSpec.yml Structure Reference

AWS CodeBuild uses a `buildspec.yml` file to define the build lifecycle and artifact generation process.

### Complete BuildSpec Structure

```yaml
version: 0.2

env:
  variables:
  parameter-store:
  secrets-manager:

phases:

  install:
    runtime-versions:
    commands:
    finally:

  pre_build:
    commands:
    finally:

  build:
    commands:
    finally:

  post_build:
    commands:
    finally:

artifacts:
  files:
  name:
  discard-paths:
  base-directory:
  exclude-paths:
  enable-symlinks:

cache:
  key:
  paths:

reports:
  report-group-name:
    files:
    base-directory:
    file-format:

proxy:
  upload-artifacts:
  logs:
```
---

# version

Specifies the BuildSpec version.

```yaml
version: 0.2
```

---

# env

Used to define environment variables.

```yaml
env:
  variables:
    APP_NAME: portfolio-app
    ENV: dev
```

Usage:

```bash
echo $APP_NAME
```

---

# phases

Defines the build lifecycle.

Execution order:

```text
install
   ↓
pre_build
   ↓
build
   ↓
post_build
```

---

# install

Used to install dependencies and configure the environment.

```yaml
install:
  commands:
    - echo "Installing dependencies"
```

Example:

```yaml
install:
  commands:
    - mvn --version
    - java -version
```

---

# runtime-versions

Used inside install phase.

Specifies language/runtime versions.

```yaml
install:
  runtime-versions:
    java: corretto21
```

Example:

```yaml
install:
  runtime-versions:
    java: corretto21
    nodejs: 20
```

---

# commands

List of shell commands to execute.

```yaml
build:
  commands:
    - echo "Building Application"
    - mvn clean package
```

---

# finally

Runs even if a command fails.

```yaml
build:
  commands:
    - mvn clean package

  finally:
    - echo "Build phase completed"
```

Useful for cleanup or logging.

---

# pre_build

Runs before build phase.

Commonly used for:

* Login to ECR
* Download dependencies
* Run validations

```yaml
pre_build:
  commands:
    - echo "Running validation"
```

---

# build

Main build phase.

```yaml
build:
  commands:
    - mvn clean package
```

Docker Example:

```yaml
build:
  commands:
    - docker build -t portfolio-app .
```

---

# post_build

Runs after build phase.

Commonly used for:

* Push Docker images
* Create deployment files
* Notifications

```yaml
post_build:
  commands:
    - echo "Build completed"
```

---

# artifacts

Defines what files CodeBuild should generate and pass to the next stage.

```yaml
artifacts:
  files:
    - '**/*'
```

---

# base-directory

Artifact root folder.

Example project:

```text
project
├── src
├── target
│   └── app.jar
```

BuildSpec:

```yaml
artifacts:
  base-directory: target
  files:
    - '*.jar'
```

Result:

```text
app.jar
```

becomes the artifact.

---

# files

Defines which files are included in artifacts.

```yaml
artifacts:
  files:
    - '**/*'
```

Meaning:

```text
Include all files recursively.
```

Examples:

```text
index.html
css/style.css
js/app.js
images/logo.png
```

---

# cache

Used to speed up builds.

Maven Example:

```yaml
cache:
  paths:
    - '/root/.m2/**/*'
```

NodeJS Example:

```yaml
cache:
  paths:
    - 'node_modules/**/*'
```

---
### Build Execution Sequence

CodeBuild executes phases in the following order:

```text
install
   ↓
pre_build
   ↓
build
   ↓
post_build
```

### Most Commonly Used Keys

```yaml
version: 0.2

env:
  variables:

phases:
  install:
    commands:

  pre_build:
    commands:

  build:
    commands:

  post_build:
    commands:

artifacts:
  base-directory:
  files:

cache:
  paths:
```

### Common Artifact Patterns

```yaml
artifacts:
  files:
    - '**/*'
```

Meaning:

```text
Include ALL files from ALL folders recursively.
```

Examples:

```text
index.html
css/style.css
js/app.js
images/logo.png
```

### Glob Pattern Reference

```text
*         → Files in current directory

/*        → Files/Folders one level below root

**        → All directories recursively

**/       → All folders recursively

**/*      → All files recursively

*.html    → All HTML files in current directory

**/*.html → All HTML files recursively

**/*.css  → All CSS files recursively

**/*.js   → All JavaScript files recursively
```

### Static Website Example

```yaml
version: 0.2

phases:
  build:
    commands:
      - echo "Preparing website files"

artifacts:
  base-directory: src/main/webapp
  files:
    - '**/*'
```

### Spring Boot Maven Example

```yaml
version: 0.2

phases:
  install:
    commands:
      - mvn --version

  build:
    commands:
      - mvn clean package

artifacts:
  base-directory: target
  files:
    - '*.jar'

cache:
  paths:
    - '/root/.m2/**/*'
```
---

```yaml
version: 0.2

phases:
  build:
    commands:
      - echo "Preparing website"

artifacts:
  base-directory: src/main/webapp
  files:
    - '**/*'
```
---

```yaml
version: 0.2

phases:

  install:
    runtime-versions:
      java: corretto21

    commands:
      - mvn --version

  build:
    commands:
      - mvn clean package

artifacts:
  base-directory: target
  files:
    - '*.jar'

cache:
  paths:
    - '/root/.m2/**/*'
```
---
```yaml
version: 0.2

# Environment variables
env:
  variables:
    APP_NAME: prashant-portfolio-app
    ENVIRONMENT: dev

  # Pull values from AWS Systems Manager Parameter Store
  parameter-store:
    DB_USERNAME: /myapp/db/username

  # Pull secrets from AWS Secrets Manager
  secrets-manager:
    DB_PASSWORD: my-db-secret:password

phases:

  # Install dependencies and runtimes
  install:

    # Runtime versions
    runtime-versions:
      java: corretto21
      nodejs: 20

    commands:
      - echo "Install phase started"
      - java -version
      - mvn --version

    # Runs even if commands fail
    finally:
      - echo "Install phase completed"

  # Runs before build
  pre_build:

    commands:
      - echo "Pre-build phase started"
      - echo "Running validations"

    finally:
      - echo "Pre-build phase completed"

  # Main build phase
  build:

    commands:
      - echo "Build phase started"
      - mvn clean package

    finally:
      - echo "Build phase completed"

  # Runs after build
  post_build:

    commands:
      - echo "Post-build phase started"
      - ls -la target

    finally:
      - echo "Post-build phase completed"

# Files to pass to next stage
artifacts:

  # Root folder for artifacts
  base-directory: target

  # Files to include
  files:
    - '*.jar'

  # Artifact name
  name: ${APP_NAME}

  # Keep folder structure
  discard-paths: no

  # Exclude files
  exclude-paths:
    - '**/*.tmp'

  # Preserve symlinks
  enable-symlinks: yes

# Cache dependencies between builds
cache:

  # Cache identifier
  key: maven-cache

  paths:
    - '/root/.m2/**/*'

# Publish test reports
reports:

  junit-reports:

    files:
      - '**/*.xml'

    base-directory: target/surefire-reports

    file-format: JUNITXML

# Proxy settings
proxy:

  upload-artifacts: yes
  logs: yes
```
---
```text
CodeBuild Starts
       │
       ▼
Install Phase
       │
       ▼
Pre-Build Phase
       │
       ▼
Build Phase
       │
       ▼
Post-Build Phase
       │
       ▼
Artifacts Generated
       │
       ▼
Reports Published
       │
       ▼
Build Completed
```

