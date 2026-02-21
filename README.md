# Smart Mirror Docker Setup

This guide explains how to run the Smart Mirror application using Docker. You don't need to be a Docker expert to get everything running! 

Docker Compose simplifies the process of starting the Backend, Frontend, MongoDB, and AI services all at once.

## Prerequisites

1. Download and install **[Docker Desktop](https://www.docker.com/products/docker-desktop/)**.
2. Make sure Docker is running (you should see the Docker icon in your system tray/menu bar).

## Step 1: Prepare Environment Variables

Before starting the app, you need to create the configuration files (`.env` files) that the services use. We have provided templates for you to copy.

Open your terminal or command prompt in this `docker-compose` folder and run the following commands to copy the example files:

**For Windows (Command Prompt):**
```cmd
copy .env.ai.example .env.ai
copy .env.api.example .env.api
```

**For Mac / Linux / PowerShell:**
```bash
cp .env.ai.example .env.ai
cp .env.api.example .env.api
```

You can open `.env.ai` and `.env.api` in a text editor to update the values if needed, but the default copies are usually enough to start.

## Step 2: Start the Application

To start all the services, ensure your terminal is in the same folder as the `docker-compose.yaml` file, then run:

```bash
docker-compose up -d --pull always
```
*(If the above command doesn't work, try `docker compose up -d --pull always` without the dash `-`).*

The `-d` flag runs the application in the background ("detached" mode), freeing up your terminal.

The `--pull always` flag ensures that Docker always checks for the latest image versions before starting the containers, 

**What happens now?**
Docker will automatically download the necessary images (API, Dashboard, AI, MongoDB) and start them together. This might take a few minutes the first time.

## Step 3: Access the Application

Once everything is running, you can access the services in your web browser:

- **Dashboard (Frontend Web UI):** [http://localhost:8081](http://localhost:8081)
- **API Service:** [http://localhost:3000](http://localhost:3000)
- **Database (MongoDB):** `localhost:27017`
- **AI Service:** Runs in the background automatically.

## Useful Docker Commands

Here are some commands you might find helpful while working with the app. Run these in the same folder:

- **View running services:**
  ```bash
  docker-compose ps
  ```

- **See the live logs (to check for errors or see what's happening):**
  ```bash
  docker-compose logs -f
  ```
  *(Press `Ctrl + C` to exit the logs).*

- **Stop the application:**
  ```bash
  docker-compose down
  ```
  *(This stops and removes the running containers, but your data is safe).*

- **Stop and restart the application:**
  ```bash
  docker-compose restart
  ```
