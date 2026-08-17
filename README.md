# WordPress Multi-Site Docker Environment

A self-hosted environment running multiple WordPress sites using Docker on Ubuntu Server (VMware VM), with the whole setup version-controlled in Git for portability to other machines.

## Goal

- Run up to 3 independent WordPress sites on one Ubuntu Server VM
- Use Docker/Docker Compose to isolate each site's app + database
- Use an Nginx reverse proxy to route different local domains to each site
- Track configuration in Git/GitHub so the environment can be rebuilt elsewhere

## Requirements
- Ubuntu Server (tested on 26)
- Docker Engine + Docker Compose plugin
- VMware VM with bridged networking (for LAN access from other devices)

## Setup Instructions (for rebuilding on a new machine)

1. Install Docker & Docker Compose
2. Clone this repo: `git clone <repo-url>`
3. Copy `.env.example` to `.env` and fill in real database credentials
4. Run `docker compose up -d`
5. On the client machine, map `site1.local` (and site2/site3 once added) to the server's IP in the local hosts file
6. Visit `http://site1.local` in a browser and complete the WordPress install

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

### Step 6: Create Nginx reverse proxy config (Site 1 only) ✅
- Added `nginx-proxy/conf.d/site1.conf` routing `site1.local` → `wordpress1` container

### Step 7: Launch containers ✅
- Ran `docker compose up -d` — db1, wordpress1, nginx-proxy all started successfully
- Verified with `docker compose ps` and `docker compose logs`

### Step 8: Access from laptop browser ✅
- Added `site1.local` → VM IP mapping in laptop's hosts file
- Confirmed WordPress install screen loads at http://site1.local
- Completed WordPress installation for Site 1

### Site 2 added ✅
- Uncommented Site 2 variables in `.env`, regenerated `.env.example`
- Added `db2`/`wordpress2` services to `docker-compose.yml`
- Added `nginx-proxy/conf.d/site2.conf`
- Confirmed accessible at http://site2.local, independent from Site 1

