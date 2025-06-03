# Harbor with Built-in Trivy (HTTP Only)

✅ **Trivy is automatically integrated**  
✅ **No need to set up Redis manually**  
✅ **Simpler and cleaner than external setup**
![HarborTrivy](https://github.com/user-attachments/assets/89141ad9-ce7a-4669-91b7-9fa11c901796)

---

##  Assumptions

- OS: Ubuntu 24.04 (clean or ready system)
- Harbor will run over HTTP only (no TLS)
- Trivy will be installed *automatically* with Harbor
- Server IP: `192.168.55.23`

---

##  Step 1: Update the System

```bash
sudo apt update && sudo apt upgrade -y
```

---

##  Step 2: Install Docker

```bash
curl -s https://get.docker.com/ | sh
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
```

> ⚠️ Log out and back in (or run `newgrp docker`) to apply the Docker group change.

---

##  Step 3: Download and Extract Harbor

```bash
cd /opt/
wget https://github.com/goharbor/harbor/releases/download/v2.13.1/harbor-online-installer-v2.13.1.tgz
tar xvf harbor-online-installer-v2.13.1.tgz
cd harbor
```

---

##  Step 4: Configure Harbor for HTTP

```bash
cp harbor.yml.tmpl harbor.yml
vim harbor.yml
```

Edit these lines in `harbor.yml`:

```yaml
hostname: 192.168.55.23

http:
  port: 80

# Comment out or remove the following:
# https:
#   port: 443
#   certificate: /your/cert.pem
#   private_key: /your/key.pem
```

---

##  Step 5: Install Harbor with Trivy Support

⚠️ **This step is critical**: Install Harbor **with built-in Trivy**

```bash
sudo ./install.sh --with-trivy
```

This command:
- Deploys Harbor and all core services
- Automatically runs the Trivy adapter
- Integrates Trivy as a vulnerability scanner (with SBOM support)
- Registers Trivy inside the Harbor UI
- Connects Trivy to the internal Redis (no manual setup needed)

---

##  Step 6: Verify Services

```bash
docker ps
```

You should see containers like:
- `harbor-core`
- `harbor-jobservice`
- `nginx`
- `harbor-portal`
- `trivy-adapter`
- `redis`
- `registry`

---

##  Step 7: Access Harbor UI

Open your browser and go to:

```
http://192.168.55.23
```

Default login:

- Username: `admin`
- Password: `Harbor12345`

> Change the password after first login!

---

##  Step 8: Configure Docker Daemon to Trust Harbor (Insecure Registry)

```bash
sudo vim /etc/docker/daemon.json
```

Add the following:

```json
{
  "insecure-registries": ["192.168.55.23"]
}
```

Restart Docker:

```bash
sudo systemctl restart docker
```

---

##  Step 9: Push a Test Image

```bash
docker login 192.168.55.23
docker pull nginx
docker tag nginx 192.168.55.23/library/nginx:latest
docker push 192.168.55.23/library/nginx:latest
```

---

##  Step 10: Test Trivy Integration (Scan + SBOM)

1. In Harbor UI, go to any project.
2. Click on an image (e.g. nginx).
3. Click **Scan**.
4. Once completed, click on the image again → go to **SBOM** tab.

You should see:
- Vulnerability report
- Software Bill of Materials (SBOM) list

---

## 🎉 Done!

You now have:

✅ Harbor running over HTTP  
✅ Built-in Trivy scanner fully integrated  
✅ No need for manual Redis or config overrides  
✅ Vulnerability scanning + SBOM working out of the box
