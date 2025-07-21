# n8n Docker Compose Setup

This repository contains a Docker Compose configuration for running n8n, a powerful workflow automation tool, along with its dependencies.

## Overview

n8n is a free and open-source workflow automation tool that allows you to connect different services and automate tasks through a visual interface. This setup includes:

- **n8n**: The main workflow automation platform with web interface
- **n8n-worker**: Dedicated worker service for queue-based workflow execution
- **PostgreSQL**: Database for storing workflow data and execution history
- **Redis**: Queue management and caching for scalable execution
- **Docker Networks**: Isolated network for secure service communication

## Prerequisites

- Docker Engine 20.10+
- Docker Compose 2.0+
- At least 2GB of available RAM
- 10GB of available disk space

## Quick Start

1. **Clone this repository:**
   ```bash
   git clone <repository-url>
   cd docker-compose-n8n
   ```

2. **Create environment file (optional):**
   ```bash
   cp .env.example .env
   ```

3. **Configure environment variables (optional):**
   Edit the `.env` file with your desired settings, or use Portainer's environment variable interface.

4. **Deploy in Portainer:**
   - Go to Stacks in Portainer
   - Create a new stack from Git repository
   - Use this repository URL
   - Configure environment variables as needed
   - Deploy the stack

5. **Access n8n:**
   Open your browser and navigate to `http://localhost:5678` (or your configured domain and port).

## Configuration

### Environment Variables

Create a `.env` file in the project root with the following variables:

```env
# n8n Configuration
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=changeme
N8N_HOST=localhost
N8N_PORT=5678
N8N_PROTOCOL=http
WEBHOOK_URL=http://localhost:5678/

# Execution Configuration
EXECUTIONS_MODE=queue
EXECUTIONS_TIMEOUT=3600
EXECUTIONS_TIMEOUT_MAX=7200
EXECUTIONS_DATA_SAVE_ON_ERROR=all
EXECUTIONS_DATA_SAVE_ON_SUCCESS=all
EXECUTIONS_DATA_SAVE_MANUAL_EXECUTIONS=true

# Database Configuration
POSTGRES_DB=n8n
POSTGRES_USER=n8n
POSTGRES_PASSWORD=changeme
POSTGRES_NON_ROOT_USER=n8n
POSTGRES_NON_ROOT_PASSWORD=changeme

# Redis Configuration
REDIS_PASSWORD=changeme

# Generic timezone for containers
GENERIC_TIMEZONE=UTC

# Logging Configuration
N8N_LOG_LEVEL=info

# Performance Configuration
N8N_PAYLOAD_SIZE_MAX=16
N8N_METRICS=false
N8N_DEFAULT_BINARY_DATA_MODE=default
N8N_SECURE_COOKIE=false
```

### Security Recommendations

- **Change default passwords**: Always change the default passwords in your `.env` file
- **Use strong passwords**: Generate strong, unique passwords for all services
- **Enable HTTPS**: Use Traefik or another reverse proxy for SSL termination in production
- **Restrict network access**: Configure firewall rules to limit access to necessary ports only
- **Regular backups**: Implement automated backup strategies for your data

## Docker Compose Services

### n8n Service
- **Image**: `n8nio/n8n:latest`
- **Ports**: `5678:5678`
- **Volumes**: Persistent data storage for workflows and settings
- **Environment**: Configuration through environment variables
- **Health checks**: Built-in health monitoring
- **Resource limits**: 2GB memory limit, 512MB reserved

### n8n Worker Service
- **Image**: `n8nio/n8n:latest`
- **Command**: `worker`
- **Purpose**: Dedicated worker for queue-based workflow execution
- **Environment**: Shares database and Redis configuration with main service
- **Resource limits**: 1GB memory limit, 256MB reserved

### PostgreSQL Database
- **Image**: `postgres:15-alpine`
- **Ports**: `5432:5432` (internal only)
- **Volumes**: Persistent database storage
- **Environment**: Database credentials and configuration
- **Health checks**: PostgreSQL readiness checks

### Redis Cache
- **Image**: `redis:7-alpine`
- **Ports**: `6379:6379` (internal only)
- **Volumes**: Persistent cache storage
- **Environment**: Redis password configuration
- **Health checks**: Redis connectivity checks

## Data Persistence

All important data is persisted using Docker volumes:

- `n8n_data`: n8n workflows, credentials, and settings
- `postgres_data`: PostgreSQL database files
- `redis_data`: Redis cache data

## Backup and Restore

### Backup

1. **Stop the services:**
   ```bash
   docker-compose down
   ```

2. **Create backup archive:**
   ```bash
   docker run --rm -v n8n_data:/source -v postgres_data:/postgres -v redis_data:/redis -v $(pwd):/backup alpine tar czf /backup/n8n-backup-$(date +%Y%m%d-%H%M%S).tar.gz -C / source postgres redis
   ```

### Restore

1. **Stop the services:**
   ```bash
   docker-compose down
   ```

2. **Remove existing data:**
   ```bash
   rm -rf n8n_data postgres_data redis_data
   ```

3. **Extract backup:**
   ```bash
   tar -xzf n8n-backup-YYYYMMDD-HHMMSS.tar.gz
   ```

4. **Start the services:**
   ```bash
   docker-compose up -d
   ```

## Updating

To update n8n and other services:

1. **Pull latest images:**
   ```bash
   docker-compose pull
   ```

2. **Restart services:**
   ```bash
   docker-compose down
   docker-compose up -d
   ```

## Monitoring and Logs

### View logs
```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f n8n
docker-compose logs -f postgres
docker-compose logs -f redis
```

### Check service status
```bash
docker-compose ps
```

### Monitor resource usage
```bash
docker stats
```

## Troubleshooting

### Common Issues

1. **Port already in use:**
   - Change the port mapping in `docker-compose.yml`
   - Or stop the conflicting service

2. **Database connection errors:**
   - Ensure PostgreSQL is running: `docker-compose ps postgres`
   - Check database credentials in `.env` file
   - Verify network connectivity between containers

3. **Permission issues:**
   - Ensure proper ownership of data directories:
     ```bash
     sudo chown -R 1000:1000 n8n_data
     ```

4. **Memory issues:**
   - Increase Docker memory allocation
   - Add swap space to your system
   - Consider scaling down other services

### Reset Everything

If you need to start fresh:

```bash
docker-compose down -v
docker system prune -f
rm -rf n8n_data postgres_data redis_data
docker-compose up -d
```

**⚠️ Warning**: This will delete all your workflows and data!

## Production Considerations

For production deployments, consider:

1. **Use external database**: Configure external PostgreSQL instance
2. **Implement SSL**: Use Traefik, Nginx, or Cloudflare for HTTPS
3. **Set up monitoring**: Use Prometheus, Grafana, or similar tools
4. **Configure logging**: Centralized logging with ELK stack or similar
5. **Backup strategy**: Automated, regular backups to remote storage
6. **Resource limits**: Set memory and CPU limits in docker-compose.yml
7. **Health checks**: Implement proper health checks for all services
8. **Secrets management**: Use Docker secrets or external secret management

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

- **n8n Documentation**: https://docs.n8n.io/
- **n8n Community**: https://community.n8n.io/
- **Docker Documentation**: https://docs.docker.com/
- **Issues**: Create an issue in this repository for setup-specific problems

## Changelog

### v1.0.0
- Initial Docker Compose setup for n8n
- PostgreSQL database integration
- Redis cache support
- Basic authentication configuration
- Data persistence and backup instructions