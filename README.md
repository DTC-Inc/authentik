# Authentik Identity Provider Docker Stack

This Docker Compose setup provides a complete Authentik Identity Provider stack following enterprise-grade patterns for authentication and authorization services.

## 🏗️ Architecture

The stack includes the following services:

- **Authentik Server**: Main web interface and API server
- **Authentik Worker**: Background task processor
- **PostgreSQL**: Primary database for configuration and user data
- **Redis**: Session storage and caching
- **GeoIP Update**: Location-based policy support (optional)
- **Cloudflared**: Cloudflare tunnel integration (optional)

## 🏢 Multi-Tenancy Support

This Authentik deployment is configured with **multi-tenancy enabled**, allowing you to manage multiple isolated organizations within a single Authentik instance. Each tenant maintains complete separation of:

- **Users and Groups**: Isolated user bases per tenant
- **Applications**: Tenant-specific SSO applications
- **Policies and Flows**: Custom authentication flows per tenant
- **Branding**: Individual tenant customization and theming

### DTC Use Case

**DTC Inc.** is actively using this multi-tenant Authentik setup to provide centralized SSO services for all client organizations. The initial implementation focuses on **ZeroTier network management**, where each client tenant can:

- Manage their own ZeroTier network access
- Control user authentication for network resources
- Maintain isolated security policies
- Customize their authentication experience

This architecture scales efficiently as DTC adds new clients and services, providing enterprise-grade identity management without requiring separate Authentik instances per client.

## 📋 Prerequisites

- Docker Engine 20.10+ and Docker Compose v2
- At least 2 CPU cores and 2 GB RAM
- Valid domain name (for production use)
- Traefik reverse proxy (or modify labels for your proxy)

## 🚀 Quick Start

### 1. Environment Setup

Create a `.env` file with your configuration:

```bash
# Copy the template environment file
cp authentik.env.template .env

# Generate secure passwords and update your .env file
sed -i '' 's/YOUR_SECRET_KEY_HERE/'$(openssl rand -base64 60 | tr -d '\n')'/g' .env
sed -i '' 's/YOUR_DB_PASSWORD_HERE/'$(openssl rand -base64 32 | tr -d '\n')'/g' .env

# Edit the .env file to customize domains and other settings
nano .env
```

### 2. Configure Environment Variables

Edit your `.env` file with the following required variables:

```env
# General Configuration
CONTAINER_NAME_PREFIX=homelab
TZ=America/New_York
PUID=1000
PGID=1000

# Authentik Configuration
AUTHENTIK_VERSION=2025.2
AUTHENTIK_SECRET_KEY=your_secure_secret_key_here
AUTHENTIK_ERROR_REPORTING=false

# Multi-Tenancy Configuration
AUTHENTIK_TENANTS_ENABLED=true

# Domain Configuration
AUTHENTIK_PUBLIC_DOMAIN=auth.yourdomain.com
AUTHENTIK_PRIVATE_DOMAIN=auth.local.yourdomain.com
AUTHENTIK_CERT_RESOLVER=letsencrypt

# Database Configuration
AUTHENTIK_DB_USERNAME=authentik
AUTHENTIK_DB_PASSWORD=your_secure_password_here
AUTHENTIK_DB_DATABASE_NAME=authentik

# Network Configuration
AUTHENTIK_APP_NETWORK=authentik_app
AUTHENTIK_DB_NETWORK=authentik_db
```

### 3. Network Setup

The stack uses two Docker networks:

- **authentik_app**: Main application network with internet access
- **authentik_db**: Internal database network (isolated)

Networks are automatically created when the stack starts. No manual setup required.

### 4. Deploy the Stack

```bash
# Start the services
docker compose up -d

# Check service status
docker compose ps

# View logs
docker compose logs -f authentik-server
```

### 5. Initial Setup

1. Navigate to `https://your-domain.com/if/flow/initial-setup/`
2. Set a password for the default `akadmin` user
3. Complete the initial configuration wizard

**Multi-Tenancy Setup**: After initial setup, navigate to the Admin interface to create and configure additional tenants. Each tenant will have its own unique subdomain or path for isolated access.

## 📧 Email Configuration (Recommended)

Configure SMTP for notifications and user verification:

```env
# Email Configuration
AUTHENTIK_EMAIL_HOST=smtp.gmail.com
AUTHENTIK_EMAIL_PORT=587
AUTHENTIK_EMAIL_USERNAME=your-email@gmail.com
AUTHENTIK_EMAIL_PASSWORD=your-app-password
AUTHENTIK_EMAIL_USE_TLS=true
AUTHENTIK_EMAIL_USE_SSL=false
AUTHENTIK_EMAIL_TIMEOUT=10
AUTHENTIK_EMAIL_FROM=authentik@yourdomain.com
```

## 🌍 GeoIP Configuration (Optional)

Enable location-based policies:

```env
# Enable GeoIP updates
ENABLE_GEOIP_UPDATE=1

# MaxMind account (optional, for enhanced data)
GEOIP_ACCOUNT_ID=your_account_id
GEOIP_LICENSE_KEY=your_license_key
```

Sign up for a free MaxMind account at: https://www.maxmind.com/en/geolite2/signup

## ☁️ Cloudflare Tunnel (Optional)

Expose Authentik through Cloudflare tunnels:

```env
# Enable Cloudflare tunnel
ENABLE_AUTHENTIK_CLOUDFLARED=1
AUTHENTIK_CLOUDFLARE_TUNNEL_TOKEN=your_tunnel_token
```

## 💾 Volume Configuration

### Local Volumes (Default)

Volumes are created locally by default. No additional configuration needed.

### Network Storage (NFS/CIFS)

For network storage, configure the volume options:

```env
# NFS Example
AUTHENTIK_DB_VOLUME_TYPE=nfs
AUTHENTIK_DB_VOLUME_OPTIONS=addr=192.168.1.10,rw,nfsvers=4
AUTHENTIK_DB_BASE=/mnt/nas/authentik/database

# CIFS Example
AUTHENTIK_MEDIA_VOLUME_TYPE=cifs
AUTHENTIK_MEDIA_VOLUME_OPTIONS=username=user,password=pass,uid=1000,gid=1000
AUTHENTIK_MEDIA_BASE=//nas.local/authentik/media
```

## 🔧 Configuration

### Environment Variables Reference

| Category | Variable | Description | Default |
|----------|----------|-------------|---------|
| **General** | `CONTAINER_NAME_PREFIX` | Prefix for container names | `homelab` |
| | `TZ` | System timezone | `UTC` |
| | `PUID`/`PGID` | User/Group IDs | `1000` |
| **Authentik** | `AUTHENTIK_VERSION` | Authentik version tag | `2025.2` |
| | `AUTHENTIK_SECRET_KEY` | Secret key (required) | - |
| | `AUTHENTIK_ERROR_REPORTING` | Enable error reporting | `false` |
| | `AUTHENTIK_TENANTS_ENABLED` | Enable multi-tenancy | `true` |
| **Domains** | `AUTHENTIK_PUBLIC_DOMAIN` | Public domain name | - |
| | `AUTHENTIK_PRIVATE_DOMAIN` | Private domain name | - |
| | `AUTHENTIK_CERT_RESOLVER` | TLS certificate resolver | `letsencrypt` |
| **Database** | `AUTHENTIK_DB_USERNAME` | Database username | `authentik` |
| | `AUTHENTIK_DB_PASSWORD` | Database password (required) | - |
| | `AUTHENTIK_DB_DATABASE_NAME` | Database name | `authentik` |

### Traefik Configuration

The stack includes Traefik labels for both public and private domains with HTTP/HTTPS routing. Ensure your Traefik instance is configured with:

- `web` entrypoint for HTTP (port 80)
- `websecure` entrypoint for HTTPS (port 443)
- Certificate resolver matching `AUTHENTIK_CERT_RESOLVER`

### Health Checks

All services include health checks:

- **Authentik Server**: HTTP health endpoint
- **Authentik Worker**: Built-in health check
- **PostgreSQL**: Database connectivity
- **Redis**: Ping response

## 🔒 Security Considerations

### Required Security Steps

1. **Generate Strong Secrets**: Use cryptographically secure random generators
2. **Use HTTPS Only**: Never run Authentik over HTTP in production
3. **Secure Database**: Use strong passwords and restrict network access
4. **Regular Updates**: Keep all components updated
5. **Monitor Logs**: Review authentication logs regularly

### Network Security

- **authentik_app**: Application network with controlled internet access
- **authentik_db**: Internal database network (fully isolated)
- Database and Redis are not accessible from outside the stack
- Only Authentik server service is exposed via reverse proxy

### Volume Security

- Ensure proper file permissions on mounted volumes
- Use encrypted storage for sensitive data
- Regular backups of database and configuration

## 🔄 Maintenance

### Updates

```bash
# Update to latest version
docker compose pull
docker compose up -d

# View update logs
docker compose logs -f authentik-server
```

### Backups

```bash
# Backup database
docker compose exec database pg_dump -U authentik authentik > authentik_backup.sql

# Backup volumes
docker run --rm -v authentik_media_volume:/data -v $(pwd):/backup alpine tar czf /backup/media_backup.tar.gz /data
```

### Monitoring

```bash
# Check service health
docker compose ps

# View service logs
docker compose logs authentik-server
docker compose logs authentik-worker

# Monitor resource usage
docker stats
```

## 🐛 Troubleshooting

### Common Issues

1. **Initial Setup 404**: Ensure the URL includes trailing slash: `/if/flow/initial-setup/`
2. **Database Connection**: Check database credentials and network connectivity
3. **Email Issues**: Verify SMTP credentials and firewall settings
4. **Certificate Problems**: Check domain DNS and certificate resolver configuration

### Debugging

Enable debug logging:

```env
AUTHENTIK_WEB_DEBUG=true
AUTHENTIK_WORKER_DEBUG=true
AUTHENTIK_ERROR_REPORTING=true
```

### Log Locations

```bash
# Application logs
docker compose logs authentik-server

# Database logs
docker compose logs database

# Worker logs
docker compose logs authentik-worker
```

## 📚 Additional Resources

- [Authentik Documentation](https://goauthentik.io/docs/)
- [Authentik GitHub Repository](https://github.com/goauthentik/authentik)
- [Traefik Documentation](https://doc.traefik.io/traefik/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)

## 🤝 Support

For issues specific to this Docker stack configuration, please check:

1. Environment variable configuration
2. Network connectivity
3. Volume permissions
4. Service health checks

For Authentik-specific issues, consult the [official documentation](https://goauthentik.io/docs/) and [community forums](https://github.com/goauthentik/authentik/discussions).

## 📄 License

This Docker Compose configuration is provided as-is under the MIT License. Authentik itself is licensed under the [Authentik License](https://github.com/goauthentik/authentik/blob/main/LICENSE). 