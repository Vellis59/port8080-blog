---
title: "Install Ollama + Open WebUI (Docker Compose)"
date: 2026-02-06
description: "A step-by-step Docker Compose setup for running Ollama with Open WebUI on your own machine."
categories: ["Tutorials"]
tags: ["ollama", "open-webui", "docker", "docker-compose", "self-hosting", "local-ai"]
featured: true
# Keep the current published URL stable.
slug: "install-ollama-and-open-webui--compose"
---

## Introduction

Running large language models locally has become significantly easier thanks to modern tooling. By combining **Ollama** with **Open WebUI** and deploying them using **Docker Compose**, you can create a clean, reproducible, and user-friendly local AI environment in minutes.

This guide walks through the complete process step by step, focusing on clarity and practical setup rather than theory.

<!--more-->

### What is Ollama?

Ollama is a local model runtime that allows you to download, manage, and run large language models directly on your machine. It abstracts away much of the complexity involved in model execution and provides a simple API and CLI for interacting with models like LLaMA, Mistral, and others.

Key features of Ollama:

- Simple model management (`pull`, `run`, `list`)
- Local-first execution
- REST API for integration with other tools
- Support for CPU and GPU workloads

### What is Open WebUI?

Open WebUI is a web-based interface designed to interact with local or remote LLM backends. When paired with Ollama, it provides a clean chat-style UI, user management, conversation history, and model selection without requiring any coding.

Key features of Open WebUI:

- Browser-based chat interface
- User authentication
- Model switching
- Conversation history
- Backend-agnostic design

### Why Use Docker Compose?

Docker Compose allows you to define and run multi-container applications using a single configuration file. For Ollama and Open WebUI, this means:

- One command to start everything
- Consistent setup across systems
- Easy updates and maintenance
- Clean separation of services

### Who This Guide Is For

This guide is ideal for:

- Developers running local AI models
- Researchers experimenting with LLMs
- Self-hosting enthusiasts
- Anyone who wants a UI on top of Ollama without manual setup

No prior experience with Ollama or Open WebUI is required.

---

## Prerequisites

### System Requirements

Minimum recommended specifications:

- 8 GB RAM (16 GB preferred for larger models)
- 20+ GB free disk space
- Modern CPU with AVX support
- Optional: NVIDIA GPU for accelerated inference

### Supported Operating Systems

- Linux (Ubuntu, Debian, Arch, etc.)
- macOS (Intel or Apple Silicon)
- Windows 10/11 with WSL2

### Required Software

#### Docker

Docker is required to run containers for both services.

#### Docker Compose

Docker Compose is used to orchestrate multiple containers together.

> Note: Docker Desktop includes Docker Compose by default.

### Basic Command-Line Knowledge

You should be comfortable with:

- Navigating directories
- Running basic terminal commands
- Editing text files

---

## Architecture Overview

### How Ollama and Open WebUI Work Together

Ollama runs as a backend service that hosts and executes models. Open WebUI acts as a frontend that communicates with Ollama over HTTP.

The two services are independent but connected through Docker networking.

### Data Flow Between Services

1. User sends a prompt via Open WebUI
2. Open WebUI forwards the request to Ollama
3. Ollama processes the request using the selected model
4. Response is returned to Open WebUI
5. Output is displayed in the browser

### Ports and Networking

| Service       | Default Port | Purpose                  |
|---------------|-------------|--------------------------|
| Ollama        | 11434       | Model API endpoint       |
| Open WebUI    | 3000        | Web user interface       |

Docker Compose creates a shared network so services can reference each other by name.

---

## Step 1: Install Docker and Docker Compose

### Install Docker on Linux

On Ubuntu-based systems:

```bash
sudo apt update
sudo apt install docker.io docker-compose-plugin
sudo systemctl enable docker
sudo systemctl start docker
```

Add your user to the Docker group (optional but recommended):

```bash
sudo usermod -aG docker $USER
```

### Install Docker on macOS

- Download Docker Desktop from the official Docker website
- Install and launch the application
- Ensure Docker is running from the menu bar

### Install Docker on Windows

- Install Docker Desktop
- Enable WSL2 during setup
- Restart when prompted

### Verify Installation

Run the following command:

```bash
docker compose version
```

If a version number is returned, Docker Compose is ready.

---

## Step 2: Prepare the Project Directory

### Create a Working Directory

Choose a location and create a project directory:

```bash
mkdir ollama-openwebui
cd ollama-openwebui
```

### Recommended Folder Structure

```text
ollama-openwebui/
├── docker-compose.yml
├── ollama/
│   └── data/
└── openwebui/
    └── data/
```

This structure helps keep persistent data organized.

### Environment Variables Overview

Environment variables allow you to:

- Define backend URLs
- Control authentication
- Tune performance settings

These will be defined inside `docker-compose.yml`.

---

## Step 3: Create the Docker Compose File

### Overview of docker-compose.yml

Create a file named `docker-compose.yml` in the project directory.

This file defines two services:

- `ollama`
- `open-webui`

### Ollama Service Configuration

#### Image and Version

Use the official Ollama image:

```yaml
image: ollama/ollama:latest
```

#### Volumes for Model Storage

Persist downloaded models:

```yaml
volumes:
  - ./ollama/data:/root/.ollama
```

#### Port Mapping

Expose the Ollama API:

```yaml
ports:
  - "11434:11434"
```

#### Resource Considerations (CPU/GPU)

By default, Ollama runs on CPU. GPU support is covered later.

### Open WebUI Service Configuration

#### Image and Version

```yaml
image: ghcr.io/open-webui/open-webui:latest
```

#### Environment Variables

Set the Ollama backend URL:

```yaml
environment:
  - OLLAMA_BASE_URL=http://ollama:11434
```

#### Connecting to Ollama

Docker Compose automatically resolves the `ollama` hostname.

#### Port Mapping

```yaml
ports:
  - "3000:8080"
```

### Optional: GPU Support Configuration

For NVIDIA GPUs on Linux:

```yaml
deploy:
  resources:
    reservations:
      devices:
        - capabilities: [gpu]
```

Ensure `nvidia-container-toolkit` is installed.

---

## Step 4: Start Ollama and Open WebUI

### Running Docker Compose

From the project directory:

```bash
docker compose up -d
```

### First-Time Startup Behavior

- Images are downloaded
- Containers are created
- Ollama initializes model storage
- Open WebUI sets up its database

This may take a few minutes.

### Verifying Running Containers

Check container status:

```bash
docker compose ps
```

Both services should show as `running`.

---

## Step 5: Access Open WebUI

### Opening the Web Interface

Open a browser and navigate to:

```text
http://localhost:3000
```

### Initial Setup and User Account

On first access, you will:

- Create an admin account
- Set login credentials
- Confirm basic settings

### Connecting to Ollama Backend

Open WebUI should automatically detect Ollama. If not:

- Go to settings
- Verify the backend URL
- Test the connection

---

## Step 6: Download and Run Models in Ollama

### Popular Models to Get Started

Recommended starter models:

- `llama3`
- `mistral`
- `codellama`
- `phi`

### Pulling Models via CLI

Download a model:

```bash
docker exec -it ollama ollama pull llama3
```

Run it directly:

```bash
docker exec -it ollama ollama run llama3
```

### Managing Models

List installed models:

```bash
docker exec -it ollama ollama list
```

Remove unused models:

```bash
docker exec -it ollama ollama rm model-name
```

### Setting Default Models in Open WebUI

Inside Open WebUI:

- Open model selector
- Choose a default model
- Save preferences per user or globally

---

## Configuration and Customization

### Persisting Data with Volumes

Both services use volumes to retain:

- Models
- User accounts
- Chat history
- Settings

This ensures data survives container restarts.

### Changing Ports

To change exposed ports, edit `docker-compose.yml`:

```yaml
ports:
  - "8080:8080"
```

Restart services afterward.

### Updating Images

Pull the latest images:

```bash
docker compose pull
docker compose up -d
```

### Environment Variable Tuning

Examples:

- Increase request timeouts
- Enable experimental features
- Adjust logging levels

Refer to official documentation for supported variables.

---

## Security Considerations

### Local-Only vs Network Exposure

By default, services are accessible only on `localhost`. Avoid exposing ports publicly unless necessary.

### Authentication in Open WebUI

Always enable authentication:

- Use strong passwords
- Disable open registration if not needed

### Reverse Proxy and HTTPS (Optional)

For advanced setups:

- Use Nginx or Traefik
- Terminate HTTPS
- Add basic access control

---

## Troubleshooting

### Containers Not Starting

- Check logs: `docker compose logs`
- Verify ports are not already in use
- Ensure Docker has enough resources

### Open WebUI Cannot Connect to Ollama

- Confirm `OLLAMA_BASE_URL`
- Ensure both containers are on the same network
- Check Ollama logs

### Model Download Issues

- Verify disk space
- Check internet connectivity
- Retry pull command

### Performance and Memory Problems

- Use smaller models
- Increase Docker memory limits
- Enable GPU acceleration if available

---

## Updating and Maintenance

### Updating Ollama

```bash
docker compose pull ollama
docker compose up -d
```

### Updating Open WebUI

```bash
docker compose pull open-webui
docker compose up -d
```

### Backing Up Data

Backup the `ollama/data` and `openwebui/data` directories regularly.

### Cleaning Up Unused Images and Volumes

```bash
docker system prune
```

Use with caution.

---

## Uninstalling Ollama

To completely remove the setup:

```bash
docker compose down -v
```

Then delete the project directory.

---

## FAQ

**Can I run Ollama without Open WebUI?**  
Yes. Ollama works independently via CLI or API.

**Does this setup work offline?**  
Once models are downloaded, inference works offline.

**Can I use multiple models simultaneously?**  
Yes, Open WebUI allows switching models per conversation.

---

## Conclusion

Installing Ollama and Open WebUI with Docker Compose is one of the cleanest ways to run local language models with a modern interface. This setup provides reproducibility, easy updates, and a clear separation between backend and frontend services.

**Next step:** download a model that fits your hardware and start experimenting with prompts, system messages, and multi-user workflows inside Open WebUI.