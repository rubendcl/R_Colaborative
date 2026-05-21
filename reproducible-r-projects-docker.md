# Reproducible R Projects with Docker Across Different PCs and Operating Systems

This guide focuses on using **Docker** to achieve the highest level of reproducibility for R projects across different machines and operating systems.

---

## 1. Why Docker for R Reproducibility?

Docker is the **most reproducible** approach for R projects because it encapsulates:

- The **exact operating system** (e.g., Ubuntu 22.04)
- The **exact R version**
- All **system libraries** and dependencies
- All **R packages** with exact versions
- **Compilers** and build tools
- **LaTeX** installations
- **Fonts** and locale settings

### What Docker Solves That renv Alone Cannot

| Problem | renv | Docker |
|---------|------|--------|
| Package versions | ✅ Yes | ✅ Yes |
| R version | ❌ No | ✅ Yes |
| System libraries | ❌ No | ✅ Yes |
| OS differences | ❌ No | ✅ Yes |
| Compilers | ❌ No | ✅ Yes |
| LaTeX | ❌ No | ✅ Yes |
| Fonts | ❌ No | ✅ Yes |
| BLAS/LAPACK | ❌ No | ✅ Yes |

---

## 2. General Architecture with Docker

A typical reproducible R project with Docker looks like this:

```
project/
│
├── .devcontainer/
│   └── devcontainer.json       (VS Code Dev Container config)
├── docker-compose.yml          (optional, for multi-service)
├── Dockerfile                  (defines the R environment)
├── .dockerignore               (files to exclude from Docker build)
├── renv/
├── renv.lock
├── .Rprofile
├── analysis.R
├── report.Rmd
├── data/
├── output/
└── README.md
```

---

## 3. Installing Docker

### 3.1 On Linux (Ubuntu/Debian)

```bash
# Update package index
sudo apt update

# Install prerequisites
sudo apt install -y ca-certificates curl

# Add Docker's official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add your user to the docker group (avoids needing sudo)
sudo usermod -aG docker $USER

# Log out and back in, or run:
newgrp docker

# Verify installation
docker --version
docker run hello-world
```

### 3.2 On Windows

**Option A — Docker Desktop (Recommended)**

1. Go to [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop/)
2. Download **Docker Desktop for Windows**
3. Run the installer (requires WSL 2)
4. Follow the setup wizard
5. Restart your computer

**Enable WSL 2 (if not already enabled):**

Open PowerShell as Administrator:

```powershell
# Enable WSL
wsl --install

# Set WSL 2 as default
wsl --set-default-version 2

# Check WSL status
wsl --status
```

**Verify Docker installation:**

```powershell
docker --version
docker run hello-world
```

**Option B — Docker Engine via WSL 2 (without Docker Desktop)**

```powershell
# Install Ubuntu WSL
wsl --install -d Ubuntu

# Inside WSL, install Docker Engine (see Linux instructions above)
```

### 3.3 On macOS

```bash
# Option 1: Docker Desktop for Mac
# Download from https://www.docker.com/products/docker-desktop/

# Option 2: Using Homebrew
brew install --cask docker

# Verify
docker --version
docker run hello-world
```

### 3.4 Verify Docker Works on All OS

After installation, run this test:

```bash
docker run --rm hello-world
```

Expected output includes:

```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

---

## 4. Dockerfile for R Projects

The `Dockerfile` is the heart of reproducibility. It defines the exact environment.

### 4.1 Minimal Dockerfile for R

```dockerfile
# Use a specific R version image (NOT "latest")
FROM rocker/r-ver:4.5.3

# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    libcurl4-openssl-dev \
    libssl-dev \
    libxml2-dev \
    libfontconfig1-dev \
    libharfbuzz-dev \
    libfribidi-dev \
    libfreetype6-dev \
    libpng-dev \
    libtiff5-dev \
    libjpeg-dev \
    && rm -rf /var/lib/apt/lists/*

# Install R packages
RUN R -e "install.packages(c('rmarkdown', 'knitr', 'ggplot2', 'dplyr', 'readr', 'renv', 'here'), repos = 'https://cloud.r-project.org/')"

# Set working directory
WORKDIR /project

# Copy project files
COPY . .

# Default command: render the report
CMD ["Rscript", "-e", "rmarkdown::render('report.Rmd')"]
```

### 4.2 Dockerfile with renv Integration

```dockerfile
# Use specific R version
FROM rocker/r-ver:4.5.3

# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    libcurl4-openssl-dev \
    libssl-dev \
    libxml2-dev \
    libfontconfig1-dev \
    libharfbuzz-dev \
    libfribidi-dev \
    libfreetype6-dev \
    libpng-dev \
    libtiff5-dev \
    libjpeg-dev \
    && rm -rf /var/lib/apt/lists/*

# Install renv
RUN R -e "install.packages('renv', repos = 'https://cloud.r-project.org/')"

# Set working directory
WORKDIR /project

# Copy lockfile first (for Docker layer caching)
COPY renv.lock .
COPY .Rprofile .
COPY renv/activate.R renv/activate.R

# Restore R packages using renv
RUN R -e "renv::restore()"

# Copy the rest of the project
COPY . .

# Default command
CMD ["Rscript", "-e", "rmarkdown::render('report.Rmd')"]
```

### 4.3 Dockerfile with LaTeX for PDF Output

```dockerfile
FROM rocker/r-ver:4.5.3

# Install system dependencies including LaTeX
RUN apt-get update && apt-get install -y --no-install-recommends \
    libcurl4-openssl-dev \
    libssl-dev \
    libxml2-dev \
    libfontconfig1-dev \
    libharfbuzz-dev \
    libfribidi-dev \
    libfreetype6-dev \
    libpng-dev \
    libtiff5-dev \
    libjpeg-dev \
    texlive-xetex \
    texlive-fonts-recommended \
    texlive-fonts-extra \
    texlive-latex-extra \
    lmodern \
    && rm -rf /var/lib/apt/lists/*

# Install R packages
RUN R -e "install.packages(c('rmarkdown', 'knitr', 'tinytex', 'renv'), repos = 'https://cloud.r-project.org/')"

# Install TinyTeX (alternative to texlive)
RUN R -e "tinytex::install_tinytex()"

WORKDIR /project
COPY . .

CMD ["Rscript", "-e", "rmarkdown::render('report.Rmd', output_format = 'pdf_document')"]
```

---

## 5. .dockerignore File

Create `.dockerignore` to exclude unnecessary files from the Docker build context:

```
# R
.Rhistory
.RData
.Ruserdata

# renv local library (will be restored inside container)
renv/library/
renv/staging/
renv/python/
renv/sandbox/

# Output (will be generated inside container)
output/

# Git
.git
.gitignore

# Docker
.dockerignore

# OS
.DS_Store
Thumbs.db

# IDE
.vscode/
.Rproj.user
```

---

## 6. Building and Running the Docker Container

### 6.1 Build the Docker Image

```bash
# Build with a tag
docker build -t my-r-project .

# Build without cache (for clean rebuild)
docker build --no-cache -t my-r-project .
```

### 6.2 Run the Container

```bash
# Run and render the report
docker run --rm my-r-project

# Run interactively (open R shell)
docker run --rm -it my-r-project R

# Run with a specific command
docker run --rm my-r-project Rscript -e "rmarkdown::render('report.Rmd', output_format = 'all')"
```

### 6.3 Mount Local Directory for Development

This allows you to edit files locally and run them in the container:

```bash
# Mount current directory to /project in container
docker run --rm -v "$(pwd):/project" my-r-project Rscript -e "rmarkdown::render('report.Rmd')"
```

**On Windows (PowerShell):**

```powershell
docker run --rm -v "${PWD}:/project" my-r-project Rscript -e "rmarkdown::render('report.Rmd')"
```

**On Windows (CMD):**

```cmd
docker run --rm -v "%CD%:/project" my-r-project Rscript -e "rmarkdown::render('report.Rmd')"
```

### 6.4 Get Output Files from Container

```bash
# Run and copy output to local machine
docker run --rm -v "$(pwd)/output:/project/output" my-r-project
```

---

## 7. Using Docker Compose

For more complex setups, use `docker-compose.yml`:

```yaml
version: '3.8'

services:
  r-project:
    build: .
    container_name: r-reproducible-project
    volumes:
      - .:/project
      - ./output:/project/output
    environment:
      - RENV_PATHS_ROOT=/project/renv
    command: >
      Rscript -e "rmarkdown::render('report.Rmd', output_format = 'all')"
```

Then run:

```bash
docker compose up
```

---

## 8. Complete Workflow: From Linux to Windows Using Docker

### Step 1 — On Linux (Source Machine)

```bash
# 1. Create project and set up renv
# (see the companion guide: reproducible-r-projects-renv-github.md)

# 2. Create Dockerfile
cat > Dockerfile << 'EOF'
FROM rocker/r-ver:4.5.3
RUN apt-get update && apt-get install -y --no-install-recommends \
    libcurl4-openssl-dev libssl-dev libxml2-dev \
    && rm -rf /var/lib/apt/lists/*
RUN R -e "install.packages('renv', repos = 'https://cloud.r-project.org/')"
WORKDIR /project
COPY renv.lock .Rprofile renv/activate.R ./
RUN R -e "renv::restore()"
COPY . .
CMD ["Rscript", "-e", "rmarkdown::render('report.Rmd')"]
EOF

# 3. Create .dockerignore
cat > .dockerignore << 'EOF'
renv/library/
renv/staging/
output/
.git
.DS_Store
EOF

# 4. Build the Docker image
docker build -t my-r-project .

# 5. Test locally
docker run --rm my-r-project

# 6. Save or push the image
# Option A: Save as tar file
docker save my-r-project | gzip > my-r-project.tar.gz

# Option B: Push to Docker Hub
docker tag my-r-project yourusername/my-r-project:latest
docker push yourusername/my-r-project:latest

# Option C: Push to GitHub Container Registry
docker tag my-r-project ghcr.io/yourusername/my-r-project:latest
docker push ghcr.io/yourusername/my-r-project:latest
```

### Step 2 — On Windows (Target Machine)

```bash
# 1. Install Docker (see section 3.2)

# 2. Get the Docker image
# Option A: Load from tar file
docker load < my-r-project.tar.gz

# Option B: Pull from Docker Hub
docker pull yourusername/my-r-project:latest

# Option C: Pull from GitHub Container Registry
docker pull ghcr.io/yourusername/my-r-project:latest

# 3. Clone the project (if not already done)
git clone REPO_URL
cd project

# 4. Run the container with mounted project
docker run --rm -v "$(pwd):/project" my-r-project

# 5. Or run interactively
docker run --rm -it my-r-project R
```

### Step 3 — Verify Reproducibility

```bash
# Compare outputs between Linux and Windows
# The outputs should be IDENTICAL

# On Linux:
docker run --rm -v "$(pwd)/output:/project/output" my-r-project

# On Windows:
docker run --rm -v "${PWD}/output:/project/output" my-r-project
```

---

## 9. Using Rocker Images (Pre-built R Docker Images)

The [Rocker Project](https://www.rocker-project.org/) provides pre-built Docker images for R.

### Available Rocker Images

| Image | Description | Use Case |
|-------|-------------|----------|
| `rocker/r-ver` | R only | Minimal R environment |
| `rocker/rstudio` | R + RStudio Server | Interactive development |
| `rocker/tidyverse` | R + tidyverse packages | Data analysis |
| `rocker/verse` | R + tidyverse + LaTeX | Reports and PDF |
| `rocker/geospatial` | R + geospatial packages | GIS and mapping |
| `rocker/shiny` | R + Shiny Server | Web applications |
| `rocker/shiny-verse` | R + Shiny + tidyverse | Full Shiny apps |

### Example: Using rocker/tidyverse

```dockerfile
FROM rocker/tidyverse:4.5.3

# Already includes: dplyr, ggplot2, tidyr, readr, purrr, stringr, etc.

# Install additional packages
RUN R -e "install.packages(c('rmarkdown', 'knitr', 'renv'), repos = 'https://cloud.r-project.org/')"

WORKDIR /project
COPY . .
CMD ["Rscript", "-e", "rmarkdown::render('report.Rmd')"]
```

### Example: Using rocker/verse (with LaTeX)

```dockerfile
FROM rocker/verse:4.5.3

# Already includes: R + tidyverse + LaTeX + rmarkdown

WORKDIR /project
COPY . .
CMD ["Rscript", "-e", "rmarkdown::render('report.Rmd', output_format = 'pdf_document')"]
```

---

## 10. VS Code Dev Containers for R

VS Code Dev Containers allow you to develop inside a Docker container seamlessly.

### 10.1 Install Dev Containers Extension

1. Open VS Code
2. Press `Ctrl+Shift+X`
3. Search for **Dev Containers** (by Microsoft)
4. Click **Install**

### 10.2 Create .devcontainer Configuration

Create `.devcontainer/devcontainer.json`:

```json
{
  "name": "R Reproducible Project",
  "build": {
    "dockerfile": "../Dockerfile",
    "context": ".."
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "reditorsupport.r",
        "yukiyudi.r-markdown",
        "davidanson.markdownlint",
        "ms-vscode-remote.remote-containers"
      ],
      "settings": {
        "r.rpath.linux": "/usr/local/bin/R",
        "r.alwaysUseActiveTerminal": true,
        "r.bracketedPaste": true,
        "r.sessionWatcher": true
      }
    }
  },
  "postCreateCommand": "R -e 'renv::restore()'",
  "mounts": [
    "source=${localWorkspaceFolder}/output,target=/project/output,type=bind"
  ],
  "remoteUser": "rstudio"
}
```

### 10.3 Using Dev Containers

1. Open your project folder in VS Code
2. Press `Ctrl+Shift+P`
3. Type **Dev Containers: Reopen in Container**
4. VS Code will build the Docker image and open the project inside the container
5. All terminal commands, extensions, and the Knit button work inside the container

**Benefits of Dev Containers:**

- No need to install R locally
- Same environment on every machine
- VS Code extensions work inside the container
- Terminal automatically runs inside the container
- Git integration works seamlessly

---

## 11. Docker + renv: Best Practices

### 11.1 Use renv Inside Docker

```dockerfile
FROM rocker/r-ver:4.5.3

# Install system deps
RUN apt-get update && apt-get install -y --no-install-recommends \
    libcurl4-openssl-dev libssl-dev libxml2-dev \
    && rm -rf /var/lib/apt/lists/*

# Install renv
RUN R -e "install.packages('renv', repos = 'https://cloud.r-project.org/')"

WORKDIR /project

# Copy renv files first (for layer caching)
COPY renv.lock .Rprofile renv/activate.R ./
COPY renv/ renv/

# Restore packages (cached unless renv.lock changes)
RUN R -e "renv::restore()"

# Copy rest of project
COPY . .
```

### 11.2 Docker Layer Caching Strategy

Order your Dockerfile commands from **least to most frequently changing**:

```dockerfile
# 1. Base image (rarely changes)
FROM rocker/r-ver:4.5.3

# 2. System dependencies (rarely changes)
RUN apt-get update && apt-get install -y ...

# 3. R package installation (changes when renv.lock changes)
COPY renv.lock ./
RUN R -e "renv::restore()"

# 4. Project files (changes frequently)
COPY . .
```

This way, rebuilding is fast when only project files change.

### 11.3 Never Install Packages Directly in Dockerfile

**BAD:**

```dockerfile
RUN R -e "install.packages(c('ggplot2', 'dplyr', 'tidyr'))"
```

**GOOD (use renv.lock):**

```dockerfile
COPY renv.lock ./
RUN R -e "renv::restore()"
```

This ensures exact version reproducibility.

---

## 12. Sharing Docker Images

### 12.1 Docker Hub

```bash
# Login
docker login

# Tag
docker tag my-r-project username/my-r-project:1.0

# Push
docker push username/my-r-project:1.0

# Pull on another machine
docker pull username/my-r-project:1.0
```

### 12.2 GitHub Container Registry (GHCR)

```bash
# Login to GHCR
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin

# Tag
docker tag my-r-project ghcr.io/username/my-r-project:1.0

# Push
docker push ghcr.io/username/my-r-project:1.0

# Pull
docker pull ghcr.io/username/my-r-project:1.0
```

### 12.3 Save/Load as tar File

```bash
# Save (portable file)
docker save my-r-project | gzip > my-r-project.tar.gz

# Transfer via USB, email, or cloud storage
# Then load on target machine
gunzip -c my-r-project.tar.gz | docker load
```

---

## 13. Docker on Different Operating Systems

### 13.1 Key Differences

| Aspect | Linux | Windows | macOS |
|--------|-------|---------|-------|
| Docker runs | Natively | Via WSL 2 | Via HyperKit/Apple Virtualization |
| Performance | Native | Good | Good |
| File sharing | Native bind mount | Via WSL 2 | osxfs (slower) |
| Volume paths | `/path/to/dir` | `C:\path` or `/mnt/c/path` | `/path/to/dir` |
| Line endings | LF | CRLF (may need conversion) | LF |

### 13.2 Handling Line Endings (Windows CRLF Problem)

Windows uses CRLF (`\r\n`) while Linux uses LF (`\n`). This can break R scripts.

**Solution 1 — Configure Git:**

```bash
# In your project
git config core.autocrlf input
```

**Solution 2 — Use .gitattributes:**

Create `.gitattributes` in your project root:

```
# Auto detect text files and force LF
* text=auto eol=lf

# R specific
*.R text eol=lf
*.Rmd text eol=lf
*.r text eol=lf
*.rmd text eol=lf
*.yaml text eol=lf
*.yml text eol=lf
*.json text eol=lf
*.md text eol=lf

# Binary files
*.png binary
*.jpg binary
*.pdf binary
```

**Solution 3 — Convert in Dockerfile:**

```dockerfile
RUN apt-get install -y dos2unix && \
    find /project -name "*.R" -o -name "*.Rmd" | xargs dos2unix
```

### 13.3 Windows Path Mounting

**PowerShell:**

```powershell
docker run --rm -v "${PWD}:/project" my-r-project
```

**CMD:**

```cmd
docker run --rm -v "%CD%:/project" my-r-project
```

**Git Bash (on Windows):**

```bash
docker run --rm -v "/$PWD:/project" my-r-project
```

---

## 14. Complete Example Project

### 14.1 Project Structure

```
docker-r-project/
│
├── .devcontainer/
│   └── devcontainer.json
├── .dockerignore
├── Dockerfile
├── .gitattributes
├── .gitignore
├── .Rprofile
├── renv.lock
├── renv/
├── README.md
├── report.Rmd
├── analysis.R
├── data/
│   └── sample.csv
└── output/
```

### 14.2 Dockerfile

```dockerfile
FROM rocker/verse:4.5.3

# Install additional system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    libcurl4-openssl-dev \
    libssl-dev \
    libxml2-dev \
    && rm -rf /var/lib/apt/lists/*

# Install renv
RUN R -e "install.packages('renv', repos = 'https://cloud.r-project.org/')"

WORKDIR /project

# Copy renv files
COPY renv.lock .Rprofile renv/activate.R ./
COPY renv/ renv/

# Restore packages
RUN R -e "renv::restore()"

# Copy project files
COPY . .

# Render all output formats
CMD ["Rscript", "-e", "rmarkdown::render('report.Rmd', output_format = 'all')"]
```

### 14.3 .devcontainer/devcontainer.json

```json
{
  "name": "R Docker Project",
  "build": {
    "dockerfile": "../Dockerfile",
    "context": ".."
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "reditorsupport.r",
        "yukiyudi.r-markdown",
        "davidanson.markdownlint"
      ],
      "settings": {
        "r.rpath.linux": "/usr/local/bin/R",
        "r.alwaysUseActiveTerminal": true
      }
    }
  },
  "postCreateCommand": "R -e 'renv::restore()'",
  "mounts": [
    "source=${localWorkspaceFolder}/output,target=/project/output,type=bind"
  ]
}
```

### 14.4 README.md

```markdown
# Docker R Reproducible Project

## Prerequisites

- Docker installed on your machine

## Quick Start

### Build the Docker image

```bash
docker build -t docker-r-project .
```

### Run the report

```bash
docker run --rm -v "$(pwd)/output:/project/output" docker-r-project
```

### Run interactively

```bash
docker run --rm -it docker-r-project R
```

### Using VS Code Dev Containers

1. Install the **Dev Containers** extension
2. Press `Ctrl+Shift+P` → **Dev Containers: Reopen in Container**
3. All tools work inside the container

## Output

Reports will be generated in the `output/` directory.
```

---

## 15. Troubleshooting Docker for R

### 15.1 Permission Denied

```bash
# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker
```

### 15.2 Docker Daemon Not Running

```bash
# Linux
sudo systemctl start docker
sudo systemctl enable docker

# Windows: Start Docker Desktop from Start Menu
# macOS: Start Docker Desktop from Applications
```

### 15.3 Out of Disk Space

```bash
# Clean unused images, containers, and volumes
docker system prune -a --volumes
```

### 15.4 R Package Installation Fails in Docker

```dockerfile
# Common fix: install system dependencies first
RUN apt-get update && apt-get install -y \
    libcurl4-openssl-dev \
    libssl-dev \
    libxml2-dev \
    libfontconfig1-dev \
    libharfbuzz-dev \
    libfribidi-dev
```

### 15.5 LaTeX Errors in Docker

```dockerfile
# Install full LaTeX distribution
RUN apt-get update && apt-get install -y \
    texlive-xetex \
    texlive-fonts-recommended \
    texlive-latex-extra

# Or use TinyTeX
RUN R -e "install.packages('tinytex'); tinytex::install_tinytex()"
```

### 15.6 Encoding Issues

```dockerfile
# Set locale in Dockerfile
RUN apt-get update && apt-get install -y locales && \
    locale-gen en_US.UTF-8 && \
    update-locale LANG=en_US.UTF-8

ENV LANG=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8
```

### 15.7 Slow Builds

```dockerfile
# Optimize layer caching: copy renv.lock before project files
COPY renv.lock ./
RUN R -e "renv::restore()"
COPY . .
```

---

## 16. Docker vs Other Approaches

| Aspect | renv Only | Docker | Docker + renv |
|--------|-----------|--------|---------------|
| Package versions | ✅ | ✅ | ✅ |
| R version | ❌ | ✅ | ✅ |
| System libraries | ❌ | ✅ | ✅ |
| OS consistency | ❌ | ✅ | ✅ |
| LaTeX | ❌ | ✅ | ✅ |
| Ease of setup | ✅ Easy | ⚠️ Medium | ⚠️ Medium |
| Build time | Fast | Slow (first time) | Slow (first time) |
| Image size | N/A | Large (1-3 GB) | Large (1-3 GB) |
| Collaboration | ✅ Git-friendly | ⚠️ Need registry | ✅ Git + registry |
| CI/CD | ✅ Easy | ✅ Possible | ✅ Best |

**Recommendation:** Use **Docker + renv** together for maximum reproducibility. Use renv for package management and Docker for the complete environment.

---

## 17. CI/CD with Docker and GitHub Actions

```yaml
# .github/workflows/render-report.yml
name: Render Report

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  render:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Build Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          load: true
          tags: report:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max
      
      - name: Render report
        run: |
          mkdir -p output
          docker run --rm \
            -v "$(pwd)/output:/project/output" \
            report:latest
      
      - name: Upload artifacts
        uses: actions/upload-artifact@v4
        with:
          name: report-output
          path: output/
```

---

## 18. Summary: Best Practices

### DO:

- ✅ Use **specific R version tags** (e.g., `rocker/r-ver:4.5.3`), never `latest`
- ✅ Use **renv inside Docker** for package management
- ✅ Use **.dockerignore** to keep builds fast
- ✅ Use **Docker layer caching** (copy renv.lock before project files)
- ✅ Use **VS Code Dev Containers** for seamless development
- ✅ Use **.gitattributes** to handle line endings
- ✅ Use **GitHub Container Registry** or **Docker Hub** for sharing images
- ✅ Use **GitHub Actions** for CI/CD automation

### DON'T:

- ❌ Don't use `latest` tag for R images
- ❌ Don't install packages directly in Dockerfile (use renv.lock)
- ❌ Don't ignore CRLF/LF line ending issues
- ❌ Don't build images without cache when not needed
- ❌ Don't store large data files in Docker images (use volumes)

---

## 19. Quick Reference Commands

```bash
# Build
docker build -t my-project .

# Build without cache
docker build --no-cache -t my-project .

# Run and render
docker run --rm my-project

# Run with mounted output
docker run --rm -v "$(pwd)/output:/project/output" my-project

# Run interactively
docker run --rm -it my-project R

# Run specific command
docker run --rm my-project Rscript analysis.R

# List images
docker images

# Remove image
docker rmi my-project

# Save image to file
docker save my-project | gzip > my-project.tar.gz

# Load image from file
gunzip -c my-project.tar.gz | docker load

# Push to Docker Hub
docker push username/my-project:tag

# Pull from Docker Hub
docker pull username/my-project:tag

# Clean up
docker system prune -a --volumes
```

---

## 20. Final Recommendations

### For Maximum Reproducibility

Use **Docker + renv + GitHub** together:

1. **Docker** — guarantees the same OS, R version, system libraries, and LaTeX
2. **renv** — guarantees the same R package versions
3. **GitHub** — provides version control and collaboration
4. **VS Code Dev Containers** — provides a seamless development experience

### For Simple Projects

If Docker is too complex for your workflow:

- Use **renv** alone (see the companion guide)
- Accept that R version and OS differences may cause minor variations
- Document the expected R version in README.md

### For Professional/Scientific Work

Docker is **strongly recommended** for:

- Clinical trials and regulatory submissions
- Published research with numerical results
- Multi-site collaborative projects
- Production pipelines
- Any project where "it works on my machine" is unacceptable

---

*Companion guide: [reproducible-r-projects-renv-github.md](./reproducible-r-projects-renv-github.md)*
