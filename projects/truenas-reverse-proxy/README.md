# TrueNAS Reverse Proxy

A Caddy reverse proxy configured for internal services on the TrueNAS network.

## Features

- Automatic SSL certificate provisioning via Cloudflare DNS challenge
- Reverse proxy for internal services
- Docker-based deployment

## Prerequisites

1. Docker and Docker Compose installed
2. Cloudflare API token with DNS edit permissions
3. DNS records configured in Cloudflare pointing to your NAS IP

## Docker Image

A pre-built Docker image is available at `ghcr.io/glitchassassin/truenas-reverse-proxy`.

The image is automatically built and published via GitHub Actions when changes are made to files in this directory. To use the pre-built image instead of building locally, update `docker-compose.yml`:

```yaml
services:
  caddy:
    image: ghcr.io/glitchassassin/truenas-reverse-proxy:latest
    # Remove the build section
```

## Setup

1. **Create Cloudflare API Token**
   - Go to https://dash.cloudflare.com/profile/api-tokens
   - Create a token with:
     - Zone:Zone:Read
     - Zone:DNS:Edit
   - Copy the token

2. **Configure Environment**
   ```bash
   cp .env.example .env
   # Edit .env and add your Cloudflare API token
   ```

3. **Update Caddyfile**
   - Edit `Caddyfile` and replace `your-email@example.com` with your email address
   - Add any additional routes as needed

4. **Build and Start the Service**
   ```bash
   docker-compose up -d --build
   ```
   
   Note: The Caddyfile is baked into the Docker image, so you'll need to rebuild the image after making changes to it.

## Adding New Routes

To add a new route, add a new block to `Caddyfile`:

```
newservice.jonwinsley.com {
    reverse_proxy 10.0.0.3:PORT {
        header_up Host {host}
        header_up X-Real-IP {remote}
        header_up X-Forwarded-For {remote}
        header_up X-Forwarded-Proto {scheme}
    }
}
```

Then rebuild and restart the container:
```bash
docker-compose up -d --build
```

Note: Since the Caddyfile is baked into the image, you must rebuild the image after making changes. If using the pre-built image from GitHub Container Registry, push your changes and wait for the GitHub Actions workflow to build and publish a new image.

## Current Routes

- `https://truenas.jonwinsley.com/` → `10.0.0.3:444`
- `https://pihole.jonwinsley.com/` → `10.0.0.3:20720`

## Notes

- SSL certificates are automatically provisioned via Cloudflare DNS challenge
- This works even for internal-only services since the DNS challenge doesn't require public access
- Certificates are stored in the `caddy_data` volume and persist across container restarts
