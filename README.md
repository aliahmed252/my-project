# Java Application with GitLab CI/CD Pipeline

This project demonstrates a Java application with a complete GitLab CI/CD pipeline that builds, tests, packages, and deploys the application using Docker.

## Pipeline Overview

The GitLab CI/CD pipeline consists of the following stages:

### 1. Build Stage
- Uses Maven 3.8.4 with OpenJDK 11
- Compiles the Java application
- Creates artifacts for the compiled classes
- Runs automatically on main branch and merge requests

### 2. Test Stage
- Runs unit tests using Maven
- Generates JUnit test reports
- Depends on successful build stage
- Uses the same Maven and JDK configuration

### 3. Package Stage
- Creates the final JAR file
- Skips tests (already run in test stage)
- Stores the JAR file as an artifact
- Depends on successful test stage

### 4. Docker Stage
- Builds a Docker image using the packaged JAR
- Tags the image with both commit SHA and 'latest'
- Pushes the image to Docker Hub
- Uses Docker-in-Docker (DinD) service
- Depends on successful package stage

### 5. Deploy Stage (Commented out)
- Pulls the latest Docker image
- Runs the container on port 8080
- Currently disabled but can be enabled for deployment

## Required GitLab CI/CD Variables

Set these variables in GitLab (Settings > CI/CD > Variables):

1. `DOCKER_USERNAME`
   - Your Docker Hub username
   - Example: `johndoe`

2. `DOCKER_PASSWORD`
   - Your Docker Hub password or access token
   - Should be marked as Masked and Protected
   - Recommended to use Docker Hub access token instead of password

## Pipeline Configuration Details

### Workflow Rules
```yaml
workflow:
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      when: always
    - if: $CI_COMMIT_BRANCH == "main"
      when: always
    - when: manual
```
- Pipeline runs automatically on merge requests
- Pipeline runs automatically on main branch
- Other branches require manual trigger

### Cache Configuration
```yaml
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - .m2/repository/
  policy: pull-push
```
- Caches Maven dependencies between pipeline runs
- Uses branch name as cache key
- Pulls and pushes cache in each job

### Docker Configuration
- Uses Docker-in-Docker (DinD) service for building images
- Tags images with both commit SHA and 'latest'
- Requires Docker Hub credentials

## Local Development

### Prerequisites
- Java 11
- Maven 3.8.4
- Docker (for local container testing)

### Building Locally
```bash
mvn clean package
```

### Running with Docker
```bash
# Build the image
docker build -t your-dockerhub-username/java-app:latest .

# Run the container
docker run -d -p 8080:8080 your-dockerhub-username/java-app:latest
```

## Pipeline Stages Explanation

### Build Stage
- Cleans previous builds
- Compiles the application
- Stores compiled classes as artifacts

### Test Stage
- Runs all unit tests
- Generates test reports
- Uses artifacts from build stage

### Package Stage
- Creates executable JAR
- Stores JAR as artifact
- Used by Docker stage

### Docker Stage
- Builds Docker image
- Pushes to Docker Hub
- Uses packaged JAR from previous stage

### Deploy Stage (Optional)
- Pulls latest image
- Runs container
- Exposes port 8080

## Best Practices
1. Always use access tokens instead of passwords for Docker Hub
2. Mark sensitive variables as Masked and Protected
3. Use specific version tags for Docker images
4. Keep the pipeline stages focused and minimal
5. Use caching to speed up builds
6. Implement proper error handling in pipeline stages

## Troubleshooting Docker Hub Authentication

If you encounter "denied: requested access to the resource is denied" error, follow these steps:

1. **Create Docker Hub Access Token**:
   - Go to Docker Hub → Account Settings → Security
   - Click "New Access Token"
   - Give it a name (e.g., "GitLab CI")
   - Copy the token immediately (you won't see it again)

2. **Update GitLab CI/CD Variables**:
   - Go to GitLab project → Settings → CI/CD → Variables
   - Update `DOCKER_PASSWORD` with the new access token
   - Make sure `DOCKER_USERNAME` is your Docker Hub username
   - Both variables should be marked as:
     - Masked: Yes
     - Protected: Yes

3. **Verify Repository Name**:
   - Update the `DOCKER_IMAGE` variable in `.gitlab-ci.yml`:
   ```yaml
   variables:
     DOCKER_IMAGE: "your-actual-dockerhub-username/java-app"
   ```
   - Replace `your-actual-dockerhub-username` with your real Docker Hub username

4. **Test Docker Login Locally**:
   ```bash
   docker login -u your-username
   # Enter your password/access token when prompted
   ```

5. **Check Repository Permissions**:
   - Ensure you have write access to the repository
   - If using an organization, verify your permissions

6. **Common Issues**:
   - Using password instead of access token
   - Incorrect username
   - Repository name mismatch
   - Expired access token
   - Insufficient permissions

## Multi-Architecture Support

The pipeline now supports building Docker images for multiple architectures:
- `linux/amd64`: For traditional x86_64 systems
- `linux/arm64/v8`: For Apple Silicon (M1/M2/M3) and ARM-based systems

### Running on Apple Silicon (M1/M2/M3) Mac

1. **Pull the image**:
   ```bash
   docker pull matoor/java-app:latest
   ```

2. **Run the container**:
   ```bash
   docker run -d -p 8080:8080 matoor/java-app:latest
   ```

The image will automatically use the correct architecture for your system.

### Verifying Architecture

To check which architecture your container is running on:
```bash
docker run --rm matoor/java-app:latest uname -m
```

For Apple Silicon Macs, this should show `aarch64` or `arm64`.

### Available Architectures

You can see all available architectures for the image:
```bash
docker manifest inspect matoor/java-app:latest
```

This will show both `linux/amd64` and `linux/arm64/v8` architectures. 