# Witsml-Explorer: Complete Clean-Slate Setup & Development Guide

This guide is designed to take a **brand-new Windows laptop (with absolutely nothing pre-installed)** and walk you through every step required to run and develop **Witsml-Explorer** inside Docker Desktop and on the host system.

---

## 🗺️ Path to Success Overview

```mermaid
graph TD
    A[1. Install WSL 2 & Git] --> B[2. Install Docker Desktop]
    B --> C[3. Fork & Clone Repository]
    C --> D[4. Deploy Unified Stack]
    D --> E[5. Setup Host Development Environment]
```

---

## 💻 Step 1: Install WSL 2 (Windows Subsystem for Linux) & Git

Docker Desktop on Windows performs best and most reliably when running on the **WSL 2 backend** rather than Hyper-V.

1. **Enable WSL 2**:
   * Open **PowerShell** as **Administrator** (Right-click -> Run as Administrator).
   * Run the following command:
     ```powershell
     wsl --install
     ```
   * *Note*: This enables the WSL 2 feature and downloads the default Ubuntu Linux distribution.
   * **Reboot your laptop** when prompted or once the command completes.

2. **Install Git**:
   * Download the official Git for Windows installer from [git-scm.com/download/win](https://git-scm.com/download/win).
   * Run the `.exe` installer. You can keep all default options selected during setup.
   * Verify installation by opening a new PowerShell window and running:
     ```powershell
     git --version
     ```
   * Set up your Git global credentials:
     ```powershell
     git config --global user.name "Your Name"
     git config --global user.email "your.email@example.com"
     ```

---

## 🐳 Step 2: Install and Configure Docker Desktop

1. **Download Docker Desktop**:
   * Download the Windows installer from the [Docker Desktop Official Page](https://www.docker.com/products/docker-desktop/).
2. **Install**:
   * Launch the downloaded installer.
   * **Important**: On the Configuration screen, ensure the option **"Use WSL 2 instead of Hyper-V (recommended)"** is **checked**.
   * Let the installation run, and click **Close and restart** (or Close and log out) when it finishes.
3. **Start and Test Docker**:
   * Open the Start Menu, search for **Docker Desktop**, and open it.
   * Accept the Service Agreement if prompted.
   * Wait until the status indicator in the bottom-left corner turns **green** ("Engine running").
   * Test the connection in a standard PowerShell terminal:
     ```powershell
     docker info
     ```

---

## 🔱 Step 3: Fork and Clone the Repository

To develop features and submit changes to the main repository, you should work out of a GitHub Fork.

1. **Fork the Repository**:
   * Go to [github.com/equinor/witsml-explorer](https://github.com/equinor/witsml-explorer).
   * Click the **Fork** button in the top-right corner.
   * Choose your personal GitHub account as the destination.

2. **Clone Your Fork**:
   * Open PowerShell and navigate to a folder where you want to keep your project (e.g., `C:\Projects`):
     ```powershell
     mkdir C:\Projects
     cd C:\Projects
     ```
   * Clone your newly forked repository (replace `<your-username>` with your actual GitHub username):
     ```powershell
     git clone https://github.com/<your-username>/witsml-explorer.git
     cd witsml-explorer
     ```

---

## 🚀 Step 4: Deploy the Unified Stack (Frontend, API, and Database)

We run the entire application—Database (MongoDB), C# API Backend, and React Frontend—simultaneously using Docker Compose.

1. **Create the Unified Compose Script**:
   Create a new file named `docker-compose.yml` in the root of your cloned `witsml-explorer` directory and paste this exact content inside it:

   ```yaml
   services:
     # 1. MongoDB Database
     db:
       image: mongo:4.4.1
       container_name: witsml-explorer-db
       restart: unless-stopped
       ports:
         - 27017:27017
       volumes:
         - mongo-data:/data/db
       environment:
         - MONGO_INITDB_ROOT_USERNAME=admin
         - MONGO_INITDB_ROOT_PASSWORD=supersecretpassword

     # 2. .NET Core Web API Backend
     api:
       build:
         context: .
         dockerfile: Dockerfile-api
       container_name: witsml-explorer-api
       restart: unless-stopped
       ports:
         - 5000:5000
       depends_on:
         - db
       environment:
         - ASPNETCORE_ENVIRONMENT=Development
         - AllowedOrigin=http://localhost:3000
         - MongoDb__Name=witsml-explorer-db
         - MongoDb__ConnectionString=mongodb://admin:supersecretpassword@db:27017

     # 3. Vite React Frontend
     frontend:
       build:
         context: .
         dockerfile: Dockerfile-frontend
       container_name: witsml-explorer-frontend
       restart: unless-stopped
       ports:
         - 3000:3000
       depends_on:
         - api

   volumes:
     mongo-data:
   ```

2. **Build and Run the Containers**:
   From your root `witsml-explorer` folder in PowerShell, run:
   ```powershell
   docker compose up --build -d
   ```
   * *What this does*: It downloads MongoDB, compiles the backend C# API inside an SDK container, compiles the static React frontend assets, deploys them via an Nginx container, and links their network together.

3. **Access Witsml-Explorer**:
   * Open your web browser and go to: **[http://localhost:3000](http://localhost:3000)**
   * Access the interactive API Swagger documentation at: **[http://localhost:5000/swagger](http://localhost:5000/swagger)**

4. **Stop the Application**:
   When you are done testing, run:
   ```powershell
   docker compose down
   ```

---

## 🛠️ Step 5: Setup Host Development Environment (For Active Coding)

Building Docker containers takes 1–3 minutes every time you make a small change. To develop efficiently, run the application code directly on your Windows host machine with **instant hot-reloading**.

### 1. Install Tools on the New Laptop
* **Install .NET 8.0 SDK**: Download and install the SDK from [dotnet.microsoft.com/en-us/download/dotnet/8.0](https://dotnet.microsoft.com/en-us/download/dotnet/8.0).
* **Install Node.js (LTS)**: Download and install the latest LTS version from [nodejs.org](https://nodejs.org/).
* **Install Yarn globally**: Open PowerShell and run:
  ```powershell
  npm install --global yarn
  ```

---

### 2. Choose Your Database Option

You have two excellent choices for local database persistence when developing:

#### 🍃 Option A: Serverless LiteDB (Easiest - Zero Setup Required)
LiteDB is a serverless document database that runs entirely in a single file on your laptop. You don't need Docker or any database engine running to use it!

1. Go to `Src/WitsmlExplorer.Api/`
2. Create a copy of the configuration template and name it `mysettings.json`:
   ```powershell
   cp appsettings.json mysettings.json
   ```
3. Edit `mysettings.json` and set the content to:
   ```json
   {
     "AllowedOrigin": "http://localhost:3000",
     "LiteDb": {
       "Name": "witsml-explorer-db.db"
     }
   }
   ```
   * *Note*: This will create a file named `witsml-explorer-db.db` directly inside your `Src/WitsmlExplorer.Api` folder to store your server connections.

#### 🐳 Option B: MongoDB running in Docker Desktop
If you want to use MongoDB during development:

1. Spin up the MongoDB container in the background:
   ```powershell
   cd Docker/MongoDb
   docker compose up -d
   ```
2. Copy `appsettings.json` to `mysettings.json` inside `Src/WitsmlExplorer.Api/` and configure it:
   ```json
   {
     "AllowedOrigin": "http://localhost:3000",
     "MongoDb": {
       "Name": "witsml-explorer-db",
       "ConnectionString": "mongodb://admin:supersecretpassword@localhost:27017"
     }
   }
   ```

---

### 3. Run the Development Servers with Hot Reloading

Now, spin up both servers side-by-side:

1. **Start the API Backend**:
   Open a PowerShell terminal, and run:
   ```powershell
   cd Src/WitsmlExplorer.Api
   dotnet watch run
   ```
   * *Why this is awesome*: `dotnet watch` monitors your C# files. Every time you save a code edit, it instantly rebuilds and restarts the API server in under 2 seconds.

2. **Start the Frontend Dev Server**:
   Open a **second** PowerShell terminal, and run:
   ```powershell
   cd Src/WitsmlExplorer.Frontend
   yarn install
   yarn dev
   ```
   * *Why this is awesome*: Vite dev server starts instantly on port `3000` and features **Hot Module Replacement (HMR)**. When you edit a React component, the changes appear on your screen in milliseconds without reloading the page.

---

## 🔄 The Fork-to-PR Contribution Lifecycle

When you are ready to implement a new feature:

1. **Create a Feature Branch**:
   ```bash
   git checkout -b feature/my-new-feature
   ```
2. **Develop & Test locally**:
   Use the host development environment (Hot Reloading) to write code and verify it works.
3. **Run Unit Tests**:
   * Backend: `cd Tests/WitsmlExplorer.Api.Tests && dotnet test`
   * Frontend: `cd Src/WitsmlExplorer.Frontend && yarn test`
4. **Final Docker Verification**:
   Before merging, run the container builds using Docker Desktop to make sure container orchestration works without a hitch:
   ```bash
   # From root folder
   docker compose up --build -d
   ```
   Open `http://localhost:3000`, test the frontend against the backend API, and ensure everything is clean.
5. **Push and Open PR**:
   Commit your changes, push to your GitHub fork, and open a Pull Request (PR) on the official `equinor/witsml-explorer` main branch.
