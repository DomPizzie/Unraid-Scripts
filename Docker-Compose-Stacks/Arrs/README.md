# Media Automation Stack (The "Arrs")

A modular Docker Compose deployment for a complete media automation pipeline, specifically tuned for Unraid but compatible with most Linux distribution with slight modifications.

## Description

This project manages the automated acquisition and management of media. It utilizes the "Arr" stack (Radarr, Sonarr, Prowlarr) for library management, qBittorrent and SABnzbd for downloads, and Gluetun as a VPN gateway to ensure secure traffic. The configuration is designed to be Linux-agnostic by utilizing environment variables for paths and permissions, ensuring it works seamlessly whether you are on Unraid, or another Linux distribution.

## Getting Started

### Dependencies

* **Operating System:** Most Linux distributions
* **Docker and Docker Compose:** Required to orchestrate the containers. If running Unraid, install [Docker Compose Manager](https://forums.unraid.net/topic/114415-plugin-docker-compose-manager).
* **VPN Subscription:** Required if using the Gluetun service (supports OpenVPN or Wireguard).

### Installing

1. **Clone the Repository (if not already done):**
```bash
git clone git@github.com:DomPizzie/Unraid-Scripts.git
cd Unraid-Scripts/Docker-Compose-Stacks/Arrs

```


2. **Prepare the Environment File:**
Copy the `env.sample` to a new file named `.env`.
```bash
cp env.sample .env

```


3. **Configure Variables:**
Edit the `.env` file to match your system. Ensure you update the following:
* `<hostname>`: Set your system's network name.
* `<your provider>`: Your VPN service (e.g., mullvad, nordvpn, private internet access).
* `<username>` / `<password>`: Your VPN credentials.
* `CONFIG_PATH` and `DATA_PATH`: Ensure these point to your desired local directories (no trailing slash).
* `PUID/PGID/UMASK`: Define the User ID and Group ID (Unraid defaults are 99/100) and the file mode mask to prevent permission issues between the host and containers.
* **Ports:** The default ports should not have any conflicts, but you can modify them to your environment's needs.


### Executing program

#### Unraid (Docker Compose Manager)

1. **Add New Stack:** Go to the **Docker** tab, scroll to the bottom, and click **Add New Stack**.
2. **Folder Name:** Name it `Arr`
3. **Configuration:** Copy `docker-compose.yml` and `.env` files using GUI and click **Update Stack** to save.
4. **Deploy:** Click **Compose Up** to launch the containers.


#### Linux (Terminal)

1. **Start the Stack:**
Run the following command to download the images and start the containers in the background:
```bash
docker compose up -d

```


2. **Verify Status:**
Ensure all containers are running correctly:
```bash
docker compose ps

```


#### Access Web Interfaces
Open your browser and navigate to the following ports:
* **Radarr:** `http://<your-host>:7878`
* **Sonarr:** `http://<your-host>:8989`
* **Prowlarr:** `http://<your-host>:9696`
* **qBittorrent:** `http://<your-host>:8081` (routed through Gluetun)
* **SABnzbd:** `http://<your-host>:8080` (routed through Gluetun)


## Post-Installation & Networking

### Internal Container Communication

Since these services share the same Docker network, they should communicate using their **Service Names**. Because the default configuration uses a prefix (`arrs_`), the DNS names within the Docker network will include that prefix.

* **Best Practice:** When configuring applications, use the service name as the host:
  * **Radarr:** `http://arrs_radarr:7878`
  * **Sonarr:** `http://arrs_sonarr:8989`
  * **Prowlarr:** `http://arrs_prowlarr:9696`

* **Why:** This ensures communication remains stable even if the underlying container IP addresses change.

### Configuring VPN-Routed Traffic

Services like **qBittorrent** and **SABnzbd** are routed through the `gluetun` network namespace (`network_mode: service:gluetun`).

* **Connection Tip (Not Required):** To connect **Prowlarr** to **qBittorrent**, use `http://arrs_gluetun:8081`.
* **Port Mapping:** Remember that for these two services, the ports are actually being managed and exposed by the `arrs_gluetun` container.


### Local Network Authentication

Most "Arr" services allow you to bypass the login screen when accessing them from a local IP range (e.g., `192.168.1.0/24`).
  * **Setting:** This is usually found under **Settings > General > Security > Authentication**.
  * **⚠️ Security Warning:** Only enable this if you fully trust your local network and have proper segmentation in place. Proceeding with "None" or "Disabled for Local Addresses" means anyone on your network can modify your library.


## Help

* **SABnzbd Access:** On the first boot, you may need to modify `sabnzbd.ini` on the host to accept local URLs or change the internal port if there are conflicts.
* **qBittorrent UI:** If you want a custom UI, you can uncomment the `DOCKER_MODS` line in the `docker-compose.yml` after the initial setup is verified.
* **Permissions:** If containers cannot write to folders, verify that `PUID` and `PGID` in your `.env` match the owner of your local data folders.
* **Additional Configs and Documentation:** See inline comments in files.
* **Check logs:** `docker logs -f`

## Authors

[@DomPizzie](https://github.com/DomPizzie)

## Version History

* 0.1
  * Initial Release: Added Radarr, Sonarr, Prowlarr, qBittorrent, SABnzbd, FlareSolverr, and Gluetun.



## License

This project is licensed under the MIT License - see the LICENSE.md file in the root of the repository for details.

## Acknowledgments

* [LinuxServer.io](https://www.linuxserver.io/) - For providing the high-quality Docker images.
* [TRaSH Guides](https://trash-guides.info/) - For the definitive guides on hardlinks and atomic moves.
* [Servarr Wiki](https://wiki.servarr.com/) - For documentation on the Arr stack.
* [Docker Compose Generator](https://github.com/ajnart/dcm) - For generating sane Docker Compose Files
