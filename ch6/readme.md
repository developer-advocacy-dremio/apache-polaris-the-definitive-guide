# Chapter 6- Hands-On with Polaris

Cloning the environment repo
```bash
git clone https://github.com/AlexMercedCoder/quick-test-polaris-environment
```

Then change directories into the cloned repository.
```bash
cd quick-test-polaris-environment
```

Defining Environment Variables

```bash
# Bash/ZSH
export AWS_REGION=us-east-1
export AWS_ACCESS_KEY_ID=your-aws-access-key
export AWS_SECRET_ACCESS_KEY=your-aws-secret-key
# CMD/Windows
set AWS_REGION=us-east-1
set AWS_ACCESS_KEY_ID=your-aws-access-key
set AWS_SECRET_ACCESS_KEY=your-aws-secret-key
# Powershell Windows
$env:AWS_REGION = "us-east-1"
$env:AWS_ACCESS_KEY_ID = "your-aws-access-key"
$env:AWS_SECRET_ACCESS_KEY = "your-aws-secret-key"
```

docker-compose file
```yaml
services:
  polaris:
    image: apache/polaris:1.1.0-incubating-SNAPSHOT
    container_name: polaris
    ports:
      - "8181:8181"
      - "8182"
    networks:
      polaris-quickstart:
    volumes:
      - ./icebergdata:/data
    environment:
      AWS_REGION: $AWS_REGION
      AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID
      AWS_SECRET_ACCESS_KEY: $AWS_SECRET_ACCESS_KEY
      POLARIS_BOOTSTRAP_CREDENTIALS: POLARIS,root_user,my_secret_id
      polaris.realm-context.realms: POLARIS
      quarkus.log.file.enable: "false"
      quarkus.otel.sdk.disabled: "true"
      polaris.features."DROP_WITH_PURGE_ENABLED": "true"
      polaris.features."ALLOW_INSECURE_STORAGE_TYPES": "true"
      polaris.features."SUPPORTED_CATALOG_STORAGE_TYPES": "[\"FILE\",\"S3\",\"GCS\",\"AZURE\"]"
      polaris.readiness.ignore-severe-issues: "true"

  spark:
    platform: linux/x86_64
    image: alexmerced/spark35nb:latest
    ports: 
      - "8080:8080"  # Master Web UI
      - "7077:7077"  # Master Port
      - "8888:8888"  # Jupyter Notebook
    volumes:
      - ./icebergdata:/data
    environment:
      - AWS_REGION=us-east-1
      - AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
      - AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
    container_name: spark
    networks:
      polaris-quickstart:

networks:
  polaris-quickstart:
```

Docker Compose Commands

```bash
# 🚀 Start services defined in docker-compose.yml

docker-compose up               # Start all services in the foreground
docker-compose up -d           # Start all services in detached (background) mode
docker-compose up --build      # Build images before starting containers
docker-compose up --force-recreate     # Recreate containers even if config hasn't changed
docker-compose up --no-deps    # Don’t start linked services (dependencies)
docker-compose up --remove-orphans     # Remove containers for services not defined in the current file
docker-compose up --scale service=NUM  # Scale a specific service (e.g., web=3 for 3 instances)

# 🧹 Tear down services

docker-compose down                     # Stop and remove containers, networks, and volumes
docker-compose down --volumes           # Also remove named volumes declared in the `volumes` section
docker-compose down --rmi all           # Remove all images used by any service
docker-compose down --rmi local         # Remove only images that don't have a custom tag
docker-compose down --remove-orphans    # Remove containers not defined in the current compose file
docker-compose down --timeout 30        # Specify shutdown timeout in seconds (default is 10)

# 🔁 Typical workflow

docker-compose up -d --build --remove-orphans
# This spins up everything fresh and clean, building images and removing any leftover containers

docker-compose down --volumes --remove-orphans
# This ensures a complete teardown, including volumes and stray containers
```