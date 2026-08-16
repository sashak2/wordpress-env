# WordPress Multi-Site Docker Environment

A self-hosted environment running multiple WordPress sites using Docker on Ubuntu Server (VMware VM), with the whole setup version-controlled in Git for portability to other machines.

## Goal

- Run up to 3 independent WordPress sites on one Ubuntu Server VM
- Use Docker/Docker Compose to isolate each site's app + database
- Use an Nginx reverse proxy to route different local domains to each site
- Track configuration in Git/GitHub so the environment can be rebuilt elsewhere

## Progress Log

### Step 1: Install Docker & Docker Compose ✅
- Installed Docker Engine + Compose plugin via Docker's official apt repo
- Added user to `docker` group to run without `sudo`
- Verified with `docker run hello-world`

### Step 2: Set up project as a Git repo ✅
- Initialized Git repo in `~/wordpress-env`
- Added `.gitignore` to exclude `.env`, logs, and data folders

### Step 3: Plan folder structure ✅
- Created `nginx-proxy/conf.d/` to hold per-site Nginx reverse proxy configs

### Step 4: Create environment variables file ✅
- Created `.env` with database credentials for all 3 sites (gitignored)
- Generated `.env.example` as a safe template for GitHub

### Step 5: Create docker-compose.yml (Site 1 only) ✅
- Defined `db1` (MariaDB) and `wordpress1` services, connected via `wp_net` network
- Defined `nginx-proxy` service to handle incoming traffic on port 80
- Named volumes `db1_data` and `wp1_data` persist database and WordPress files

## Requirements
- Ubuntu Server (tested on 26)
- Docker Engine + Docker Compose plugin
- VMware VM with bridged networking (for LAN access from other devices)

## Setup Instructions (for rebuilding on a new machine)
_(to be filled in as steps are completed)_
