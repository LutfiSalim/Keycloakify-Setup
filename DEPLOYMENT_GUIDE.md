# Keycloak with PostgreSQL Deployment Guide

This guide provides detailed instructions for deploying Keycloak with PostgreSQL using Docker. This setup provides a robust authentication and user management system suitable for development, testing, and production environments.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Deployment Options](#deployment-options)
3. [Basic Deployment (with Hardcoded Credentials)](#basic-deployment-with-hardcoded-credentials)
4. [Advanced Deployment (with Environment Variables)](#advanced-deployment-with-environment-variables)
5. [Automated Setup](#automated-setup)
6. [Post-Deployment Configuration](#post-deployment-configuration)
7. [Security Considerations](#security-considerations)
8. [Maintenance and Operations](#maintenance-and-operations)
9. [Troubleshooting](#troubleshooting)
10. [Upgrading](#upgrading)

## Prerequisites

Before deploying Keycloak with PostgreSQL, ensure you have the following:

- **Docker**: Version 20.10.0 or higher
- **Docker Compose**: Version 2.0.0 or higher
- **Network Access**: Ports 8080 (Keycloak) and 5432 (PostgreSQL) available
- **Storage**: Sufficient disk space for PostgreSQL data persistence
- **Git**: For cloning the repository (optional)

You can verify your Docker and Docker Compose installations with:

```bash
docker --version
docker-compose --version
```

## Deployment Options

This repository provides two deployment options:

1. **Basic Deployment**: Uses `docker-compose.yml` with hardcoded credentials
2. **Advanced Deployment**: Uses `docker-compose-with-env.yml` with environment variables from `.env` file

Choose the option that best suits your needs based on your security requirements and deployment environment.

## Basic Deployment (with Hardcoded Credentials)

This approach is simpler but less secure for production environments.

### Step 1: Clone or Download the Repository

```bash
git clone https://your-repository-url.git
cd keycloak-postgres-docker
```

### Step 2: Review the Configuration

Open `docker-compose.yml` to review the default configuration:

- PostgreSQL database name: `keycloak`
- PostgreSQL username: `keycloak`
- PostgreSQL password: `password`
- Keycloak admin username: `admin`
- Keycloak admin password: `admin`

### Step 3: Deploy the Services

```bash
docker-compose up -d
```

This command starts both PostgreSQL and Keycloak services in detached mode.

### Step 4: Verify Deployment

Check that both services are running:

```bash
docker-compose ps
```

You should see both `keycloak-postgres` and `keycloak` containers in the "Up" state.

## Advanced Deployment (with Environment Variables)

This approach is more secure and flexible, especially for production environments.

### Step 1: Clone or Download the Repository

```bash
git clone https://your-repository-url.git
cd keycloak-postgres-docker
```

### Step 2: Configure Environment Variables

Create a `.env` file from the provided example:

```bash
cp .env.example .env
```

Edit the `.env` file to customize your configuration:

```
# PostgreSQL Configuration
POSTGRES_DB=keycloak
POSTGRES_USER=keycloak
POSTGRES_PASSWORD=your_secure_password

# Keycloak Configuration
KEYCLOAK_ADMIN=admin
KEYCLOAK_ADMIN_PASSWORD=your_secure_admin_password

# Keycloak Database Connection
KC_DB=postgres
KC_DB_URL=jdbc:postgresql://postgres:5432/keycloak
KC_DB_USERNAME=keycloak
KC_DB_PASSWORD=your_secure_password

# Keycloak Host Configuration
KC_HOSTNAME=localhost
KC_HOSTNAME_PORT=8080
KC_HTTP_ENABLED=true
KC_HOSTNAME_STRICT_HTTPS=false
KC_HEALTH_ENABLED=true
```

### Step 3: Deploy the Services

```bash
docker-compose -f docker-compose-with-env.yml up -d
```

This command starts both PostgreSQL and Keycloak services using the environment variables from your `.env` file.

### Step 4: Verify Deployment

Check that both services are running:

```bash
docker-compose -f docker-compose-with-env.yml ps
```

You should see both `keycloak-postgres` and `keycloak` containers in the "Up" state.

## Automated Setup

This repository includes a setup script that automates the deployment process.

### Step 1: Make the Script Executable

```bash
chmod +x setup.sh
```

### Step 2: Run the Setup Script

```bash
./setup.sh
```

The script will:
1. Check if Docker and Docker Compose are installed
2. Create a `.env` file from `.env.example` if it doesn't exist
3. Start the containers using `docker-compose`
4. Verify that the containers are running
5. Display the Keycloak URL and admin credentials

## Post-Deployment Configuration

After deploying Keycloak, you'll need to perform some initial configuration.

### Accessing Keycloak Admin Console

1. Open a web browser and navigate to [http://localhost:8080](http://localhost:8080)
2. Click on "Administration Console"
3. Log in with the admin credentials:
   - Username: `admin` (or your custom KEYCLOAK_ADMIN value)
   - Password: `admin` (or your custom KEYCLOAK_ADMIN_PASSWORD value)

### Initial Configuration Tasks

1. **Create a New Realm**:
   - Click "Create Realm" in the dropdown menu in the top-left corner
   - Enter a name for your realm and click "Create"

2. **Create a Client**:
   - Go to "Clients" in the left sidebar
   - Click "Create client"
   - Follow the wizard to set up your client

3. **Create Users**:
   - Go to "Users" in the left sidebar
   - Click "Add user"
   - Fill in the user details and click "Create"
   - Go to the "Credentials" tab to set a password

4. **Configure User Federation** (optional):
   - Go to "User Federation" in the left sidebar
   - Configure LDAP or other user federation providers

## Security Considerations

For production deployments, consider the following security enhancements:

### 1. Use Strong Passwords

Update all default passwords in the `.env` file with strong, unique passwords.

### 2. Enable HTTPS

For production, configure Keycloak to use HTTPS by:

1. Obtaining SSL certificates
2. Updating the Keycloak environment variables:
   ```
   KC_HOSTNAME_STRICT_HTTPS=true
   KC_HTTP_ENABLED=false
   ```
3. Adding certificate configuration to the Keycloak container

### 3. Network Security

- Use a reverse proxy like Nginx or Traefik in front of Keycloak
- Implement IP restrictions for admin access
- Consider using Docker networks to isolate services

### 4. Regular Updates

Keep both PostgreSQL and Keycloak images updated to the latest stable versions to address security vulnerabilities.

## Maintenance and Operations

### Backing Up PostgreSQL Data

Create regular backups of your PostgreSQL data:

```bash
# Create a backup
docker exec -t keycloak-postgres pg_dumpall -c -U keycloak > keycloak_backup_$(date +%Y-%m-%d).sql

# Restore from backup
cat keycloak_backup_YYYY-MM-DD.sql | docker exec -i keycloak-postgres psql -U keycloak
```

### Monitoring

Monitor the health of your services:

```bash
# Check Keycloak logs
docker-compose logs keycloak

# Check PostgreSQL logs
docker-compose logs postgres

# Check Keycloak health endpoint
curl http://localhost:8080/health/ready
```

### Scaling

For high-availability setups:

1. Use a load balancer in front of multiple Keycloak instances
2. Set up PostgreSQL replication
3. Configure shared storage for Keycloak instances

## Troubleshooting

### Common Issues and Solutions

1. **Keycloak Can't Connect to PostgreSQL**:
   - Check if PostgreSQL is running: `docker-compose ps`
   - Verify network connectivity: `docker exec keycloak ping postgres`
   - Check PostgreSQL logs: `docker-compose logs postgres`

2. **"Invalid Username or Password" When Logging In**:
   - Verify the admin credentials in the `.env` file
   - Check if the credentials were properly passed to the container

3. **Database Migration Errors**:
   - Check Keycloak logs: `docker-compose logs keycloak`
   - Ensure PostgreSQL version compatibility

4. **Container Fails to Start**:
   - Check for port conflicts: `netstat -tuln | grep 8080`
   - Verify Docker has sufficient resources

### Viewing Logs

```bash
# View all logs
docker-compose logs

# View Keycloak logs
docker-compose logs keycloak

# View PostgreSQL logs
docker-compose logs postgres

# Follow logs in real-time
docker-compose logs -f
```

## Upgrading

### Upgrading Keycloak

1. Update the Keycloak image version in your Docker Compose file
2. Pull the new image: `docker-compose pull keycloak`
3. Restart the services: `docker-compose up -d`
4. Monitor logs for any migration issues: `docker-compose logs -f keycloak`

### Upgrading PostgreSQL

Upgrading PostgreSQL requires careful planning to avoid data loss:

1. Back up your database: `docker exec -t keycloak-postgres pg_dumpall -c -U keycloak > keycloak_backup_$(date +%Y-%m-%d).sql`
2. Update the PostgreSQL image version in your Docker Compose file
3. Stop the services: `docker-compose down`
4. Remove the PostgreSQL volume if necessary: `docker volume rm keycloak-postgres-docker_postgres_data`
5. Start the services with the new version: `docker-compose up -d`
6. Restore your data if needed

## Conclusion

This deployment guide provides a comprehensive approach to running Keycloak with PostgreSQL using Docker. By following these instructions, you can set up a robust authentication and user management system that can be scaled and secured according to your requirements.

For more information, refer to the official documentation:
- [Keycloak Documentation](https://www.keycloak.org/documentation)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Docker Documentation](https://docs.docker.com/)
