
# 🎧 Terraform Spotify Playlist Creator

This project automates the creation of a Spotify playlist using **Terraform** and the **Spotify Provider**. It allows you to define playlist names and pre-load them with tracks directly from Terraform configuration.

---

## 📌 Project Overview

Using Terraform's Infrastructure as Code approach, this setup creates a playlist called **Sigmasongs** and adds one or more tracks automatically using their Spotify track IDs.

---

## 📁 Project Structure

```
terraform-spotify/
├── .env                   # Contains Spotify Client ID and Secret
├── provider.tf            # Spotify provider configuration
├── playlist.tf            # Playlist resource and tracks
├── variables.tf           # api_key variable declaration
├── terraform.tfvars       # Value for api_key
├── terraform.tfstate      # Terraform state tracking
├── .terraform.lock.hcl    # Dependency lock file
```

---

## ✅ Prerequisites

- Terraform installed: [Download](https://developer.hashicorp.com/terraform/downloads)
- Docker installed: [Download](https://www.docker.com/products/docker-desktop)
- Spotify account (free is fine)
- Spotify Developer App (for API access)

---

## 🔐 Step 1: Create Spotify Developer App

1. Go to [Spotify Developer Dashboard](https://developer.spotify.com/dashboard/).
2. Create a new app.
3. Add a Redirect URI:
   ```
   https://localhost:27228/spotify_callback
   ```
4. Copy the `Client ID` and `Client Secret`.
5. Create a `.env` file in your project directory:

   ```env
   SPOTIFY_CLIENT_ID=your_client_id
   SPOTIFY_CLIENT_SECRET=your_client_secret
   ```

---
 
 Use HTTPS on localhost with Self-Signed Certificate (Recommended)

Generate a self-signed SSL certificate:
 ```
mkdir -p ~/ssl && cd ~/ssl

openssl req -x509 -newkey rsa:4096 -sha256 -days 365 \
  -nodes -keyout key.pem -out cert.pem \
  -subj "/CN=localhost"
 ```
Create an Nginx config to proxy HTTPS to your Docker app:
 ```
sudo nano /etc/nginx/conf.d/spotify-local.conf
 ```
Paste this:

 ```
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

Start Nginx:
 ```
sudo nginx -t && sudo systemctl restart nginx
 ```
Add this redirect URI in Spotify Dashboard:
 ```
https://localhost/spotify_callback
 ```


Update your .env file:

 ```
REDIRECT_URI=https://localhost/spotify_callback
 ```


Trust the certificate in your browser (Advanced → Proceed to localhost).

--- 
🐳 Step 2: Run Auth Proxy with Docker

docker run --rm -it -p 27228:27228 --env-file ./.env ghcr.io/conradludgate/spotify-auth-proxy

After authentication, copy the access token shown in the terminal and use it in terraform.tfvars.
## 🐳 Step 2: Run Auth Proxy with Docker

```bash
docker run --rm -it -p 27228:27228 --env-file ./.env ghcr.io/conradludgate/spotify-auth-proxy
```

After authentication, copy the access token shown in the terminal and use it in `terraform.tfvars`.

---

## 🛠️ Step 3: Project Configuration

### provider.tf

```hcl
provider "spotify" {
  api_key = var.api_key
}
```

### variables.tf

```hcl
variable "api_key" {
  type = string
}
```

### terraform.tfvars

```hcl
api_key = "your_generated_spotify_api_key"
```

### playlist.tf

```hcl
resource "spotify_playlist" "SigmaSongs" {
  name   = "Sigmasongs"
  tracks = [
    "72Z2D7jpKevicRkyL45mbw"
  ]
}
```

---

## 🚀 Step 4: Deploy with Terraform

```bash
terraform init
terraform apply
```

Type `yes` when prompted.

---

## ✅ Result

Log in to your Spotify account and confirm the playlist **Sigmasongs** is created with the specified track(s).

---

## 🧠 Bonus Tips

- Add more tracks by including additional Spotify track IDs in the `tracks` list.
- Use Terraform variables to dynamically control playlist names and content.
