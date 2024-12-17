# GoldenEye Surveillance System

## Table of Contents

1. [Introduction](#introduction)
2. [Project Structure](#project-structure)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Usage](#usage)
6. [API Endpoints](#api-endpoints)
7. [Security Features](#security-features)
8. [Development](#development)
9. [Deployment](#deployment)
10. [Troubleshooting](#troubleshooting)
11. [Contributing](#contributing)
12. [License](#license)

## Introduction

The GoldenEye Surveillance System is a comprehensive, secure, and scalable solution for managing and monitoring multiple camera streams. It provides real-time video feeds, recording capabilities, and PTZ (Pan-Tilt-Zoom) control for ONVIF-compatible cameras. The system is designed to be deployed in a Docker environment, ensuring ease of deployment and scalability.

## Project Structure

The project is organized into several directories and files, each serving a specific purpose:

- **webcam_server/**: Contains the main application code.
  - **web_camera_stream.py**: The main Flask application file.
  - **auth.py**: Handles user authentication and session management.
  - **config.py**: Manages configuration settings, including encryption.
  - **ptz_controller.py**: Provides PTZ control for ONVIF cameras.
  - **validators.py**: Contains validation schemas for input data.
  - **rate_limiter.py**: Implements rate limiting for API endpoints.
  - **Dockerfile**: Docker configuration for building the application image.
  - **docker-compose.yml**: Docker Compose configuration for multi-container setup.
  - **requirements.txt**: Python dependencies for the project.
  - **templates/**: HTML templates for the web interface.
  - **static/**: Static files including CSS, JavaScript, and images.
  - **storage/**: Contains scripts and Dockerfile for video storage management.
  - **scripts/**: Utility scripts for setup and maintenance.

## Installation

### Prerequisites

- Docker: Ensure Docker is installed on your system. Follow the official [Docker installation guide](https://docs.docker.com/get-docker/) for your operating system.
- Docker Compose: Install Docker Compose by following the [official guide](https://docs.docker.com/compose/install/).

### Steps

1. **Clone the Repository**

   ```bash
   git clone https://github.com/acseguin21/webcam_server.git
   cd webcam_server
   ```

2. **Environment Configuration**

   Create a `.env` file in the `webcam_server` directory with the following environment variables:

   ```plaintext
   FLASK_ENV=development
   LOG_LEVEL=DEBUG
   CAMERA_USERNAME=your_camera_username
   CAMERA_PASSWORD=your_camera_password
   SECRET_KEY=your_secret_key
   ADMIN_PASSWORD=your_admin_password
   SSL_CERT_PATH=/app/certs/cert.pem
   SSL_KEY_PATH=/app/certs/key.pem
   ```

3. **Generate SSL Certificates**

   Use the provided script to generate self-signed SSL certificates:

   ```bash
   bash scripts/generate_cert.sh
   ```

4. **Build and Run Docker Containers**

   Use Docker Compose to build and run the application:

   ```bash
   docker-compose up --build
   ```

   This command will build the Docker images and start the containers defined in `docker-compose.yml`.

## Configuration

### Camera Configuration

The camera settings are stored in `camera_config.yml`. This file should be placed in the `webcam_server` directory. The configuration file should follow the YAML format and include details such as camera URLs and ONVIF credentials.

Example `camera_config.yml`:

```yaml
- name: "Front Door Camera"
  url: "rtsp://192.168.1.100:554/stream"
  onvif:
    host: "192.168.1.100"
    username: "admin"
    password: "password"
```

### Application Configuration

The application configuration is managed through environment variables and the `config.py` file. Sensitive information such as passwords can be encrypted using the `SecureConfig` class in `config.py`.

## Usage

### Accessing the Web Interface

Once the application is running, access the web interface by navigating to `https://localhost:443` in your web browser. You will be prompted to log in using the credentials specified in the `.env` file.

### Managing Camera Streams

- **Live View**: View live streams from connected cameras.
- **PTZ Control**: Use the PTZ controls to adjust the camera's position and zoom.
- **Recording**: Start and stop recordings for each camera. Access recorded videos from the "Recordings" section.

### Settings

Adjust system settings such as recording length, file size, and video quality through the "Settings" menu. Changes are applied immediately and stored for future sessions.

## API Endpoints

The application provides several API endpoints for interacting with the system programmatically. Below is a list of available endpoints:

- **GET /video_feed/<camera_id>**: Retrieve the video feed for a specific camera.
- **POST /camera/<camera_id>/record/start**: Start recording for a specific camera.
- **POST /camera/<camera_id>/record/stop**: Stop recording for a specific camera.
- **GET /settings/current**: Retrieve current system settings.
- **POST /settings/update**: Update system settings.
- **GET /recordings**: List all recorded videos.
- **GET /recordings/<filename>**: Download a specific recording.

## Security Features

- **Authentication**: User authentication is managed using Flask-Login. Passwords are hashed using Werkzeug's security utilities.
- **Rate Limiting**: API requests are rate-limited using Redis to prevent abuse.
- **SSL/TLS**: The application uses SSL/TLS to encrypt data in transit. Self-signed certificates are generated for development purposes.
- **Content Security Policy**: Flask-Talisman is used to enforce a strict Content Security Policy (CSP) to mitigate XSS attacks.

## Development

### Setting Up a Development Environment

1. **Install Python and Virtualenv**

   Ensure Python 3.9 and virtualenv are installed on your system.

   ```bash
   python3 -m venv venv
   source venv/bin/activate
   ```

2. **Install Dependencies**

   Install the required Python packages:

   ```bash
   pip install -r requirements.txt
   ```

3. **Run the Application Locally**

   Start the Flask development server:

   ```bash
   flask run
   ```

   The application will be accessible at `http://localhost:5000`.

### Code Structure

- **Flask Application**: The main application logic is contained in `web_camera_stream.py`.
- **Templates**: HTML templates are located in the `templates` directory.
- **Static Files**: CSS, JavaScript, and images are stored in the `static` directory.
- **Scripts**: Utility scripts for setup and maintenance are located in the `scripts` directory.

## Deployment

### Production Deployment

For production deployment, it is recommended to use a reverse proxy such as Nginx to handle SSL termination and serve static files. The application should be run using a WSGI server like Gunicorn.

1. **Build Docker Image**

   Build the Docker image for production:

   ```bash
   docker build -t goldeneye-surveillance:latest .
   ```

2. **Run Docker Container**

   Run the container with the appropriate environment variables and volume mounts:

   ```bash
   docker run -d --name goldeneye-surveillance \
     -p 443:443 \
     -v /path/to/certs:/app/certs:ro \
     -v /path/to/recordings:/app/static/recordings \
     -e FLASK_ENV=production \
     -e LOG_LEVEL=WARNING \
     goldeneye-surveillance:latest
   ```

3. **Configure Nginx**

   Set up Nginx as a reverse proxy to forward requests to the Flask application. Ensure SSL certificates are properly configured.

## Troubleshooting

### Common Issues

- **Camera Stream Not Loading**: Ensure the camera URL is correct and accessible from the server. Check network connectivity and firewall settings.
- **SSL Certificate Errors**: Verify that the SSL certificates are correctly generated and placed in the specified directory.
- **Authentication Failures**: Ensure the correct credentials are used and that the environment variables are set properly.

### Logs

Application logs are output to the console and can be viewed using Docker logs:

```bash
docker logs -f goldeneye-surveillance
```

## Contributing

Contributions are welcome! Please follow the guidelines below:

1. Fork the repository and create a new branch for your feature or bugfix.
2. Write clear, concise commit messages and include tests for new features.
3. Submit a pull request with a detailed description of your changes.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for more information.