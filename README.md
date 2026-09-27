# Certbot TLS Guide

A minimal guide for issuing and automatically renewing **Let's Encrypt TLS certificates** with Docker Certbot.

The project is intentionally focused on two tasks:

- issuing a certificate for a domain;
- renewing existing certificates automatically.

## Requirements

- Linux server
- Docker
- Docker Compose
- A domain pointing to the server
- Port `80` available during the initial HTTP-01 validation

## 1. Install Certbot

Create a working directory:

```bash
mkdir -p /opt/certbot
cd /opt/certbot
```

Create `/opt/certbot/docker-compose.yml`:

```yaml
services:
  certbot:
    container_name: certbot
    image: certbot/certbot
    network_mode: host
    volumes:
      - ./certs:/etc/letsencrypt
```

## 2. Issue the first certificate

Make sure port `80` is available.

Replace `your-domain.com` and `admin@your-domain.com` with your own values:

```bash
docker run --rm \
  -v $(pwd)/certs:/etc/letsencrypt \
  -v $(pwd)/var-lib-letsencrypt:/var/lib/letsencrypt \
  --network host \
  certbot/certbot certonly --standalone \
  --non-interactive --agree-tos \
  --email admin@your-domain.com \
  -d your-domain.com
```

After a successful run, the certificate files are stored in the `certs` directory.

## 3. Renew certificates

Run:

```bash
docker compose run --rm certbot renew
```

Certbot checks the certificates and renews only those that are close to expiration.

You can safely run this command periodically.

## 4. Automatic renewal

Open your cron configuration:

```bash
crontab -e
```

For example, run the renewal check every day at midnight:

```cron
0 0 * * * cd /opt/certbot && docker compose run --rm certbot renew
```

Running `renew` regularly is preferable to trying to calculate the exact renewal date yourself. Certbot decides whether a certificate actually needs renewal.

## Web guide

The repository includes a standalone `index.html` with the same instructions and a Russian/English language switcher.

For GitHub Pages, enable **Settings → Pages** and select the branch containing `index.html`.

## License

Use and modify this guide freely for your own infrastructure.
