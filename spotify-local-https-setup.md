
# 🎧 Spotify Local HTTPS Setup using Nginx and Self-Signed Certificate

This guide helps you expose your local Spotify auth proxy running in Docker via HTTPS on `https://localhost/spotify_callback` using a self-signed SSL certificate and Nginx reverse proxy.

---

## 🚀 Prerequisites

- RHEL or any Linux system
- Docker installed
- Nginx installed (`sudo dnf install nginx -y`)
- Spotify Developer account

---

## 🔐 Step 1: Generate a Self-Signed SSL Certificate

```bash
mkdir -p ~/ssl && cd ~/ssl

openssl req -x509 -newkey rsa:4096 -sha256 -days 365 \
  -nodes -keyout key.pem -out cert.pem \
  -subj "/CN=localhost"
```

This creates:

- `key.pem` (private key)
- `cert.pem` (public certificate)

---

## 🌐 Step 2: Configure Nginx for HTTPS

Create a new config file:

```bash
sudo nano /etc/nginx/conf.d/spotify-local.conf
```

Paste the following content:

```nginx
server {
    listen 443 ssl;
    server_name localhost;

    ssl_certificate     /home/<your-username>/ssl/cert.pem;
    ssl_certificate_key /home/<your-username>/ssl/key.pem;

    location / {
        proxy_pass http://localhost:27228;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

> 🔁 Replace `<your-username>` with your Linux username.

---

## 🐳 Step 3: Run Docker Spotify Auth Proxy

```bash
docker run --rm -it -p 27228:27228 --env-file ./.env ghcr.io/conradludgate/spotify-auth-proxy
```

---

## 🔁 Step 4: Restart Nginx

```bash
sudo nginx -t && sudo systemctl restart nginx
```

---

## 🎛️ Step 5: Update Spotify Developer App

1. Go to: [Spotify Developer Dashboard](https://developer.spotify.com/dashboard)
2. Edit your app
3. Add this to the **Redirect URIs**:
   ```
   https://localhost/spotify_callback
   ```

---

## ⚙️ Step 6: Update `.env` File

Update the redirect URI to match your local HTTPS proxy:

```env
REDIRECT_URI=https://localhost/spotify_callback
```

---

## 🛡️ Optional: Trust the Certificate (Browser Warning)

When you visit `https://localhost`, your browser may show a warning.

- Click **Advanced → Proceed to localhost**
- To permanently trust the cert, you can import `cert.pem` into your browser/system’s trusted certificate store

---

## ✅ Done!

You're now running a secure local HTTPS proxy that Spotify will accept for OAuth redirect.

---
**Author:** Gaurav Sidharth Bharane  
**Use Case:** Local Spotify Dev on RHEL with HTTPS Compliance
