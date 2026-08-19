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
5. Visit `http://<vm_ip>:8001` (site1), `http://<vm_ip>:8002` (site2) etc. in a browser and complete the WordPress install
   - No hosts file editing needed — sites are distinguished by port, not hostname

## Daily Start/Stop Routine

**Start up:**
```bash
# 1. Power on the VM (VMware), then SSH in
ssh your_username@<vm_ip>

# 2. Start containers
cd ~/wordpress-env
docker compose up -d
docker compose ps   # confirm all services are "Up"
```

**Shut down:**
```bash
# 1. Stop containers (keeps data, cleanly closes DB connections)
cd ~/wordpress-env
docker compose down

# 2. Shut down the VM safely (never just close VMware)
sudo shutdown now
```

## Useful Docker Commands

| Purpose | Command |
|---|---|
| Start all containers | `docker compose up -d` |
| Stop & remove containers (keep data) | `docker compose down` |
| Stop & remove containers + **delete data** | `docker compose down -v` ⚠️ |
| Pause without removing | `docker compose stop` / `docker compose start` |
| View status | `docker compose ps` |
| View logs (all / one service) | `docker compose logs -f` / `docker compose logs -f wordpress1` |
| Restart one service | `docker compose restart nginx-proxy` |
| Shell into a container | `docker exec -it wordpress1 bash` |
| Connect to a database | `docker exec -it db1 mariadb -u root -p` |
| Check disk usage | `docker system df` |
| Clean up unused images/data | `docker system prune` |

**Note:** `docker compose up -d` is always safe to re-run — it creates what's missing, starts what's stopped, and leaves unchanged running services alone. This is how new sites (site2, site3) get added without disrupting existing ones.

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

### Switched to port-based access (no hosts file needed) ✅
- Changed Nginx configs from hostname-based (`server_name site1.local`) to port-based routing
- `nginx-proxy` now listens on 8001 (site1) and 8002 (site2), mapped via `docker-compose.yml` ports
- Access via `http://<vm_ip>:8001` and `http://<vm_ip>:8002` — no laptop hosts file editing required

### Fixed: WordPress install redirect losing the port ✅
- **Issue:** Visiting `http://<vm_ip>:8001` redirected to `http://<vm_ip>/wp-admin/install.php` (no port) → "refused to connect"
- **Cause:** Nginx's `proxy_set_header Host $host;` strips the port number before forwarding to WordPress, so WordPress generated URLs without it
- **Fix:** Changed `$host` → `$http_host` in both site configs, which preserves the port from the original request
- Rebuilt both sites clean (`docker compose down -v` + `up -d`) after the fix to clear broken install state

## Access URLs
- Site 1: `http://<vm_ip>:8001`
- Site 2: `http://<vm_ip>:8002`
