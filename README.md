# CUPS Print Server - Multi-Architecture Docker Image 🖨️🐳

![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/CaTeIM/docker-cups/cups.yml?branch=main&style=for-the-badge)
![Docker Hub Pulls](https://img.shields.io/docker/pulls/cateim/cups?style=for-the-badge)
![Docker Image Size](https://img.shields.io/docker/image-size/cateim/cups/latest?style=for-the-badge)

*[🇧🇷 Leia em Português](README.pt-br.md)*

This is a multi-architecture Docker image of **[CUPS (Common Unix Printing System)](https://github.com/OpenPrinting/cups)**, built upon the latest **Ubuntu (Rolling)** and **Debian (Testing)** bases. The goal is to provide a print server with the latest CUPS versions, ready to use and easy to deploy in containerized environments.

## 📚 Source Code

This is an open-source project. The `Dockerfile`, startup script, and GitHub Actions build workflow are all available in the project repository.

➡️ **[GitHub Repository: CaTeIM/docker-cups](https://github.com/CaTeIM/docker-cups)**

## 🐳 Available Tags

This repository builds two image "tracks". The `latest` tag always points to the Ubuntu base.

| Tag | Distro Base | CUPS Version |
| :--- | :--- | :--- |
| `latest`, `ubuntu`, `[version]` | Ubuntu (Rolling) | Dynamically updated |
| `debian`, `[version]` | Debian 13 (Trixie) | Dynamically updated |

*💡 **Note:** The `[version]` tags (e.g., `2.4.12`) are dynamically extracted from the upstream OS package manager. The images are automatically rebuilt every Sunday to ensure the latest CUPS version and security patches.*

## ✨ Why use this image?

-   ✅ **Always Up-to-Date**: Uses the `apt-get` installation method from the official Ubuntu Rolling and Debian 13 repositories. The image is automatically rebuilt every week to include the latest OS security patches and CUPS updates.

-   ✅ **Multi-Distro**: Choose between an Ubuntu (`latest`) or Debian (`debian`) base, depending on your preference.

-   🔒 **Secure**: The build process includes applying all available security updates (`apt-get upgrade`).

-   🖨️ **Ready to Use**: Includes a comprehensive set of print drivers (`printer-driver-all`, `hplip`, `openprinting-ppds`), making most printers plug-and-play.

-   🚀 **Multi-Architecture**: Built to run natively on `linux/amd64` (PCs, Intel/AMD Servers) and `linux/arm64` (Raspberry Pi, Orange Pi 5, etc.).

-   🔧 **Smart Configuration**: Features a startup script that sets up an admin user and prepares CUPS for remote access on the first run.

## ⚙️ How to Use (Example with `docker-compose.yml`)

The recommended way to use this image is with Portainer Stacks or `docker-compose`. Create a `docker-compose.yml` file with the following content.

> **Note:** For USB printers to work properly (detect out-of-paper, reconnection, etc.), mapping the `/run/udev` and `/run/dbus` volumes is **essential**.

```yaml
version: "3.8"

services:
  cups:
    # Use 'latest' (Ubuntu), 'debian', or dynamic version tags like '2.4.x'
    image: cateim/cups:latest
    container_name: cups
    # Gives the container full access to system devices (mandatory for USB)
    privileged: true
    restart: unless-stopped
    environment:
      # Define a secure password for the 'admin' user on the web interface
      - ADMIN_PASSWORD=your_strong_password
      # Set your timezone
      - TZ=America/Sao_Paulo
    volumes:
      # --- Configuration and Data ---
      - /srv/cups/config:/etc/cups
      - /srv/cups/logs:/var/log/cups
      - /srv/cups/spool:/var/spool/cups
      
      # --- Hardware and System (CRITICAL FOR USB) ---
      # Physical access to USB ports
      - /dev/bus/usb:/dev/bus/usb
      # Allows communication with system services (fixes ColorManager/DBus errors)
      - /run/dbus:/run/dbus:ro
      # Allows CUPS to detect hardware events (e.g., reloading paper, opening cover)
      - /run/udev:/run/udev:ro
      
      # Synchronize the clock with the Host
      - /etc/localtime:/etc/localtime:ro
      
    # 'host' is the easiest way to ensure printer discovery on the network (AirPrint/Bonjour)
    # If you prefer 'bridge', make sure to expose port 631:631
    network_mode: host
```

### 🔑 Administration

  - To access the web interface, use the address: `https://<YOUR_SERVER_IP>:631`
  - To access the **Administration** area, use the login `admin` and the password you defined in the `ADMIN_PASSWORD` variable.

## 🐛 Troubleshooting (USB and "Host-Based" Printers)

If you use USB printers that rely on host-loaded firmware (like HP LaserJet P1102, P1005, 1020 series, etc.), you might notice that **turning the printer off and on** causes CUPS to stop responding or leaves jobs as "Held".

This happens because the Container loses the USB device reference when the electrical connection drops.

### Definitive Solution (Host UDEV Rule)

For the Container to automatically reconnect whenever the printer is restarted or the cable is reconnected, create a `udev` rule on your host system (not in the container).

1.  Find your printer's ID with the `lsusb` command (e.g., `03f0:002a`).
2.  Create the `/etc/udev/rules.d/99-fix-cups-usb.rules` file with the content below (replacing with your ID):

```bash
# Automatically restarts the CUPS container upon detecting the printer connection
ACTION=="add", SUBSYSTEM=="usb", ATTR{idVendor}=="YOUR_VENDOR_ID", ATTR{idProduct}=="YOUR_PRODUCT_ID", RUN+="/usr/bin/docker restart cups"
```

3.  Reload the rules: `udevadm control --reload-rules && udevadm trigger`

### Operational Tip (Out of Paper)

If you run out of paper and the job is held, avoid turning off the printer.

1.  Load the paper.
2.  **Open and close the toner/cartridge cover.**
3.  The mapped `/run/udev` volume in the container will detect the event and CUPS will automatically resume printing.

---

*This project is not officially affiliated with OpenPrinting. All credit for CUPS goes to its respective developers.*
