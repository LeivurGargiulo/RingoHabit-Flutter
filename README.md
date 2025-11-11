# RingoHabit

A cross-platform personal organization application built with Flutter and FastAPI. Track habits, manage notes, set reminders, plan your week, and organize your calendar - all with local-first architecture and seamless multi-device sync.

## Features

- **Habits Management**: Create, track, and manage daily habits with streaks and completion tracking
- **Notes**: Organize notes with hierarchical folder structure
- **Reminders**: Set up reminders with recurrence support and system notifications
- **Calendar**: Manage calendar events with full calendar view
- **Weekly Planner**: Plan your week with time blocks
- **Categories**: Organize habits with categories
- **Wiki**: Document management with linking and search
- **Tasks**: Task management with priorities, status, and multiple views (Kanban, Eisenhower)
- **Feelings**: Daily mood and feelings tracking
- **Analytics**: View habit streaks, completion rates, and statistics
- **Local-First**: All data stored locally in Isar database, works without network connection
- **Multi-Device Sync**: Automatic sync every 15 minutes between devices via home server
- **Device Tracking**: Automatic device identification and registration
- **Cross-Platform**: Works on Linux, Windows, macOS, Android, and iOS

## Project Structure

This repository contains two main projects:

- **Flutter App** (`flutter_app/`) - Cross-platform mobile and desktop application
- **FastAPI Backend** (`fastapi_backend/`) - RESTful API server for multi-device sync

## Quick Start

### Prerequisites

- **Flutter**: Version 3.9.2 or higher
- **Dart**: Version 3.9.2 or higher
- **Python**: Version 3.8 or higher (for backend)
- **Docker & Docker Compose**: For backend deployment (optional but recommended)

### FastAPI Backend

#### Option 1: Docker Deployment (Recommended)

See [Backend Docker Deployment](#backend-docker-deployment) section below for complete instructions.

#### Option 2: Local Development

1. Navigate to the backend directory:
```bash
cd fastapi_backend
```

2. Create a virtual environment (recommended):
```bash
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. Install dependencies:
```bash
pip install -r requirements.txt
```

4. Run the server:
```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

The API will be available at `http://localhost:8000` with interactive docs at `http://localhost:8000/docs`

### Flutter App

1. Navigate to the Flutter app directory:
```bash
cd flutter_app
```

2. Install dependencies:
```bash
flutter pub get
```

3. Generate database code:
```bash
flutter pub run build_runner build --delete-conflicting-outputs
```

4. Run the app (see platform-specific instructions below)

5. Configure API connection (for multi-device sync):
   - Open the app and go to Settings
   - Enter the API base URL (default: `http://localhost:8000`)
   - For devices on your network, use your server's IP address (e.g., `http://192.168.1.100:8000`)
   - No authentication token required for local network use

## Production Docker Deployment (Root Level)

The project includes a complete Docker setup at the root level for production deployment, including both the backend API and Flutter build environment.

### Prerequisites

- Docker and Docker Compose installed
- At least 4GB of available disk space
- Linux, macOS, or Windows with WSL2

### Quick Start

1. **Clone the repository** (if not already done):
```bash
git clone <repository-url>
cd RIngoHabit-Flutter
```

2. **Configure environment variables** (optional):
```bash
# Copy the example environment file
cp .env.example .env

# Edit .env with your settings (optional, defaults work for most cases)
nano .env
```

3. **Start the production environment**:
```bash
./docker-start.sh
```

This will:
- Build the backend Docker image
- Start the backend API service
- Set up persistent volumes for data and logs
- Configure networking between services

4. **Verify the backend is running**:
```bash
curl http://localhost:8000/api/health
# Should return: {"status":"healthy"}
```

### Building Flutter Releases

The Docker setup includes a Flutter build environment for creating production releases:

**Build for Linux:**
```bash
./docker-build.sh --platform linux --mode release
```

**Build for Windows:**
```bash
./docker-build.sh --platform windows --mode release
```

**Build for Android:**
```bash
./docker-build.sh --platform android --mode release
```

**Build for Web:**
```bash
./docker-build.sh --platform web --mode release
```

**Note:** iOS builds require a macOS host and cannot be done in Docker.

### Available Scripts

- `./docker-start.sh` - Start all services
- `./docker-stop.sh` - Stop all services
- `./docker-build.sh` - Build Flutter releases (see options with `--help`)
- `./docker-logs.sh` - View logs (use `-f` to follow, `-s SERVICE` for specific service)

### Environment Variables

Create a `.env` file in the project root to customize the deployment:

```bash
# Backend Configuration
BACKEND_PORT=8000
DATABASE_URL=sqlite:///./data/database.db
CORS_ORIGINS=*
LOG_LEVEL=INFO
WORKERS=4

# Flutter Build Configuration
FLUTTER_BUILD_MODE=release
FLUTTER_TARGET_PLATFORM=linux
```

### Accessing Services

- **Backend API**: http://localhost:8000
- **API Documentation**: http://localhost:8000/docs
- **Health Check**: http://localhost:8000/api/health

### Managing Data

Data is persisted in Docker volumes:
- `ringohabit-backend_data` - Database files
- `ringohabit-backend_logs` - Application logs
- `ringohabit-flutter_builds` - Flutter build outputs

**View volumes:**
```bash
docker volume ls | grep ringohabit
```

**Backup database:**
```bash
docker run --rm -v ringohabit-backend_data:/data -v $(pwd):/backup ubuntu:22.04 tar czf /backup/backup-$(date +%Y%m%d).tar.gz /data
```

**Restore database:**
```bash
docker run --rm -v ringohabit-backend_data:/data -v $(pwd):/backup ubuntu:22.04 tar xzf /backup/backup-YYYYMMDD.tar.gz -C /
```

### Stopping Services

```bash
./docker-stop.sh
```

To also remove volumes (⚠️ **WARNING**: This deletes all data):
```bash
docker-compose down -v
```

### Troubleshooting

**Backend won't start:**
```bash
# Check logs
./docker-logs.sh -s backend

# Check if port is in use
sudo netstat -tulpn | grep 8000
```

**Flutter build fails:**
```bash
# Check build logs
./docker-logs.sh -s flutter-builder

# Rebuild the Flutter builder image
docker-compose build flutter-builder
```

**View all service status:**
```bash
docker-compose ps
```

**View resource usage:**
```bash
docker stats
```

### Production Considerations

1. **Security**: For production deployments exposed to the internet:
   - Set `CORS_ORIGINS` to specific domains instead of `*`
   - Consider adding authentication/API tokens
   - Use HTTPS with a reverse proxy (nginx, Traefik, Caddy)
   - Use PostgreSQL instead of SQLite for better performance

2. **Performance**: 
   - Adjust `WORKERS` based on your server's CPU cores
   - Monitor resource usage with `docker stats`
   - Consider using a production database (PostgreSQL)

3. **Backups**: 
   - Set up automated backups for the database volume
   - Store backups in a separate location
   - Test restore procedures regularly

4. **Monitoring**:
   - Use Docker logging drivers for centralized logging
   - Set up health check monitoring
   - Monitor disk space for volumes

## Backend Docker Deployment

### Prerequisites

- Docker and Docker Compose installed
- Server or computer with Linux
- All devices on the same local network (for local deployment)

### Step 1: Install Docker (if not installed)

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Install Docker Compose
sudo apt install docker-compose -y

# Add your user to docker group (to run docker without sudo)
sudo usermod -aG docker $USER
# Log out and log back in for this to take effect

# Verify installation
docker --version
docker-compose --version
```

### Step 2: Prepare Backend Files

#### Option A: Transfer from Local Machine

**On your local machine:**
```bash
cd /path/to/habit-tracker-flutter
tar --exclude='venv' --exclude='__pycache__' --exclude='*.pyc' --exclude='database.db' \
    -czf fastapi_backend.tar.gz fastapi_backend/

# Transfer to server (replace with your server details)
scp fastapi_backend.tar.gz user@your-server-ip:/opt/habit-tracker/
```

**On your server:**
```bash
# Create directory
sudo mkdir -p /opt/habit-tracker
sudo chown $USER:$USER /opt/habit-tracker

# Extract files
cd /opt/habit-tracker
tar -xzf fastapi_backend.tar.gz
cd fastapi_backend
```

#### Option B: Clone or Copy Directly

If you have the files directly on the server:
```bash
sudo mkdir -p /opt/habit-tracker
sudo chown $USER:$USER /opt/habit-tracker
cd /opt/habit-tracker
# Copy or clone the fastapi_backend directory here
```

### Step 3: Create Required Directories

```bash
cd /opt/habit-tracker/fastapi_backend
mkdir -p data logs
```

### Step 4: Configure Environment (Optional)

Create a `.env` file if you need custom configuration:

```bash
cd /opt/habit-tracker/fastapi_backend
nano .env
```

**Example `.env` file:**
```bash
DATABASE_URL=sqlite:///./data/database.db
CORS_ORIGINS=*
LOG_LEVEL=INFO
```

**Note:** For local network deployment, no authentication token is required. The defaults in `docker-compose.yml` work fine.

### Step 5: Build and Start Docker Container

```bash
cd /opt/habit-tracker/fastapi_backend

# Build and start the container
docker-compose up -d --build

# Check status
docker-compose ps

# View logs
docker-compose logs -f
```

The container will:
- Build the Docker image
- Start the API service on port 8000
- Create persistent volumes for data and logs
- Automatically restart if it crashes

### Step 6: Configure Firewall

```bash
# Allow port 8000 for local network access
sudo ufw allow 8000/tcp

# Allow SSH (important!)
sudo ufw allow 22/tcp

# Enable firewall
sudo ufw enable

# Check status
sudo ufw status
```

### Step 7: Find Your Server IP Address

```bash
hostname -I
```

**Note the IP address** (e.g., `192.168.1.100`) - you'll need this for the Flutter apps.

### Step 8: Verify Backend is Running

```bash
# Test health endpoint locally
curl http://localhost:8000/api/health
# Should return: {"status":"healthy"}

# Test from another device on your network
curl http://your-server-ip:8000/api/health
```

You can also access the API documentation:
- Swagger UI: `http://your-server-ip:8000/docs`
- ReDoc: `http://your-server-ip:8000/redoc`

### Step 9: Configure Flutter Apps

1. Open the Flutter app on your device
2. Go to **Settings**
3. Enter **API Base URL**: `http://your-server-ip:8000` (e.g., `http://192.168.1.100:8000`)
   - **Important**: Use your server's IP address, not `localhost`
4. No API token required for local network deployment
5. The app will automatically register your device on first sync

### Common Operations

#### View Logs

```bash
cd /opt/habit-tracker/fastapi_backend

# Real-time logs
docker-compose logs -f

# Last 100 lines
docker-compose logs --tail=100

# Logs for specific service
docker-compose logs -f api
```

#### Restart Backend

```bash
cd /opt/habit-tracker/fastapi_backend
docker-compose restart
```

#### Stop Backend

```bash
cd /opt/habit-tracker/fastapi_backend
docker-compose down
```

#### Start Backend

```bash
cd /opt/habit-tracker/fastapi_backend
docker-compose up -d
```

#### Update Backend

```bash
cd /opt/habit-tracker/fastapi_backend

# Pull latest code (if using git)
git pull

# Rebuild and restart
docker-compose down
docker-compose build
docker-compose up -d
```

#### Check Container Status

```bash
cd /opt/habit-tracker/fastapi_backend
docker-compose ps

# Check resource usage
docker stats habit-tracker-api
```

### Database Backup

#### Manual Backup

```bash
# Create backup directory
sudo mkdir -p /var/backups/habit-tracker

# Backup and compress database
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
gzip -c /opt/habit-tracker/fastapi_backend/data/database.db > \
       /var/backups/habit-tracker/database_$TIMESTAMP.db.gz

# Or use the automated backup script
sudo /usr/local/bin/backup-habit-tracker.sh
```

#### Restore Database

```bash
# Stop the service
cd /opt/habit-tracker/fastapi_backend
docker-compose down

# Restore from backup (backups are compressed with gzip)
gunzip -c /var/backups/habit-tracker/database_YYYYMMDD_HHMMSS.db.gz > \
   /opt/habit-tracker/fastapi_backend/data/database.db

# Restart the service
docker-compose up -d
```

#### Automatic Daily Backups

A backup script can be created for automated database backups. It should automatically:
- Creates compressed backups of the database
- Maintains exactly 7 backups (one week retention)
- Logs all backup operations

**Setup:**

1. **Copy the script to a system location:**
   ```bash
   # Create your backup script at /usr/local/bin/backup-habit-tracker.sh
   # See the backup section below for script contents
   sudo chmod +x /usr/local/bin/backup-habit-tracker.sh
   ```

2. **Create log directory:**
   ```bash
   sudo mkdir -p /var/log
   sudo touch /var/log/habit-tracker-backup.log
   sudo chmod 644 /var/log/habit-tracker-backup.log
   ```

3. **Set up daily cron job:**
   ```bash
   sudo crontab -e
   # Add the following line (default: runs at 2:00 AM):
   0 2 * * * /usr/local/bin/backup-habit-tracker.sh
   ```

   **Configuring Backup Time:**
   
   The cron format is: `MINUTE HOUR DAY MONTH DAY-OF-WEEK COMMAND`
   
   Examples:
   - `0 2 * * *` - Daily at 2:00 AM (default)
   - `0 3 * * *` - Daily at 3:00 AM
   - `30 1 * * *` - Daily at 1:30 AM
   - `0 0 * * *` - Daily at midnight (00:00)
   - `0 14 * * *` - Daily at 2:00 PM (14:00)
   - `30 23 * * *` - Daily at 11:30 PM
   
   To change the backup time, edit the cron job:
   ```bash
   sudo crontab -e
   # Modify the time in the cron expression
   ```

4. **Verify the cron job:**
   ```bash
   sudo crontab -l
   ```

**Backup Details:**
- **Location**: `/var/backups/habit-tracker/`
- **Format**: Compressed files named `database_YYYYMMDD_HHMMSS.db.gz`
- **Retention**: 7 most recent backups (one week)
- **Logs**: `/var/log/habit-tracker-backup.log`

**Manual Backup:**
You can also run the backup script manually:
```bash
sudo /usr/local/bin/backup-habit-tracker.sh
```

### Docker Deployment Troubleshooting

#### Container Won't Start

```bash
# Check logs
cd /opt/habit-tracker/fastapi_backend
docker-compose logs

# Check if port is in use
sudo netstat -tulpn | grep 8000

# Check Docker status
sudo systemctl status docker
```

#### Database Issues

```bash
# Check data directory permissions
ls -la /opt/habit-tracker/fastapi_backend/data

# Fix permissions if needed
chmod 755 /opt/habit-tracker/fastapi_backend/data
chown -R $USER:$USER /opt/habit-tracker/fastapi_backend/data
```

#### Can't Access from Other Devices

- Verify firewall allows port 8000: `sudo ufw status`
- Check server IP address: `hostname -I`
- Ensure all devices are on the same network
- Test connectivity: `curl http://your-server-ip:8000/api/health` from another device

#### Sync Fails in Flutter App

- Verify API Base URL is correct (use server IP, not localhost)
- Ensure backend is running: `docker-compose ps`
- Check network connectivity
- Verify devices are on the same local network
- Check backend logs: `docker-compose logs`

### Docker Deployment Security Notes

#### Local Network Deployment

This setup is designed for local network/home server use:
- **No Authentication**: Authentication is disabled for simplicity
- **CORS**: Set to `*` to allow all origins (acceptable for local network)
- **Network Isolation**: Backend should only be accessible on local network

#### For Public/Cloud Deployment

If deploying to a public server, **additional security is REQUIRED**:
- Enable authentication with a strong API token
- Use HTTPS/SSL with a reverse proxy (nginx, Traefik, Caddy)
- Restrict CORS to specific origins
- Use PostgreSQL instead of SQLite
- Configure firewall rules appropriately
- Enable rate limiting
- Use secrets management

### Docker Deployment Quick Reference

#### Server IP Address
```bash
hostname -I
```

#### Health Check
```bash
curl http://localhost:8000/api/health
# or from another device:
curl http://your-server-ip:8000/api/health
```

#### Common Commands
```bash
cd /opt/habit-tracker/fastapi_backend
docker-compose up -d          # Start
docker-compose down           # Stop
docker-compose restart        # Restart
docker-compose logs -f        # View logs
docker-compose ps             # Check status
docker-compose build          # Rebuild image
```

## Platform-Specific Build Instructions

### Linux Desktop

#### Prerequisites
- Flutter SDK installed
- Linux desktop development tools
- GTK development libraries

#### Build and Run

```bash
cd flutter_app

# Install dependencies
flutter pub get

# Generate database code
flutter pub run build_runner build --delete-conflicting-outputs

# Run in debug mode
flutter run -d linux

# Build release
flutter build linux --release
```

The built application will be in `build/linux/x64/release/bundle/`

#### Installation

After building, you can install the app:
```bash
cd build/linux/x64/release/bundle
# Copy the bundle to your desired location
sudo cp -r . /opt/ringohabit
# Create a desktop entry (optional)
```

### Windows Desktop

#### Prerequisites
- Flutter SDK installed
- Visual Studio 2022 with "Desktop development with C++" workload
- Windows 10 or later

#### Build and Run

```bash
cd flutter_app

# Install dependencies
flutter pub get

# Generate database code
flutter pub run build_runner build --delete-conflicting-outputs

# Run in debug mode
flutter run -d windows

# Build release
flutter build windows --release
```

The built application will be in `build/windows/x64/runner/Release/`

#### Installation

The Windows build creates a standalone executable. You can:
- Run `ringo_habit.exe` directly from the Release folder
- Create a shortcut to the executable
- Package it for distribution using tools like Inno Setup or NSIS

### macOS Desktop

#### Prerequisites
- Flutter SDK installed
- Xcode installed (latest version recommended)
- macOS 10.14 or later
- CocoaPods (for iOS dependencies)

#### Build and Run

```bash
cd flutter_app

# Install dependencies
flutter pub get

# Generate database code
flutter pub run build_runner build --delete-conflicting-outputs

# Run in debug mode
flutter run -d macos

# Build release
flutter build macos --release
```

The built application will be in `build/macos/Build/Products/Release/ringo_habit.app`

#### Installation

1. Build the app as shown above
2. Open Finder and navigate to `build/macos/Build/Products/Release/`
3. Drag `ringo_habit.app` to your Applications folder
4. Right-click the app and select "Open" (first time only, to bypass Gatekeeper)

#### Code Signing (for Distribution)

For distribution outside the App Store, you'll need to:
1. Get an Apple Developer certificate
2. Configure code signing in Xcode
3. Build with proper signing:
```bash
flutter build macos --release
```

### Android

#### Prerequisites
- Flutter SDK installed
- Android Studio installed
- Android SDK (API level 21 or higher)
- Java Development Kit (JDK) 17 or higher

#### Build and Run

```bash
cd flutter_app

# Install dependencies
flutter pub get

# Generate database code
flutter pub run build_runner build --delete-conflicting-outputs

# Run in debug mode (with device connected or emulator running)
flutter run -d android

# Build debug APK
flutter build apk --debug

# Build release APK
flutter build apk --release

# Build App Bundle (for Google Play Store)
flutter build appbundle --release
```

#### Installation

**Debug APK:**
- Located at: `build/app/outputs/flutter-apk/app-debug.apk`
- Install via: `adb install build/app/outputs/flutter-apk/app-debug.apk`
- Or transfer to device and install manually

**Release APK:**
- Located at: `build/app/outputs/flutter-apk/app-release.apk`
- Install via: `adb install build/app/outputs/flutter-apk/app-release.apk`
- Or transfer to device and install manually

**App Bundle:**
- Located at: `build/app/outputs/bundle/release/app-release.aab`
- Upload to Google Play Console for distribution

#### Signing

For release builds, see [ANDROID_SIGNING.md](ANDROID_SIGNING.md) for detailed signing instructions.

### iOS

#### Prerequisites
- Flutter SDK installed
- Xcode installed (latest version recommended)
- macOS (required for iOS development)
- Apple Developer account (for device testing and distribution)
- CocoaPods installed

#### Setup

1. Install CocoaPods (if not already installed):
```bash
sudo gem install cocoapods
```

2. Install iOS dependencies:
```bash
cd flutter_app/ios
pod install
cd ..
```

#### Build and Run

```bash
cd flutter_app

# Install dependencies
flutter pub get

# Generate database code
flutter pub run build_runner build --delete-conflicting-outputs

# Run in debug mode (with device connected or simulator running)
flutter run -d ios

# Build release (for device)
flutter build ios --release

# Build for simulator
flutter build ios --simulator --release
```

#### Installation

**Development Build:**
- Connect your iOS device via USB
- Open Xcode: `open ios/Runner.xcworkspace`
- Select your device in Xcode
- Click Run (or press Cmd+R)
- Trust the developer certificate on your device (Settings > General > Device Management)

**Release Build:**
- Build the release version as shown above
- Open Xcode: `open ios/Runner.xcworkspace`
- Select "Any iOS Device" as target
- Product > Archive
- Distribute via App Store or Ad Hoc distribution

#### Code Signing

1. Open Xcode: `open ios/Runner.xcworkspace`
2. Select the Runner project in the navigator
3. Select the Runner target
4. Go to "Signing & Capabilities"
5. Select your development team
6. Xcode will automatically manage provisioning profiles

**Note:** For App Store distribution, you'll need:
- An Apple Developer account ($99/year)
- Proper App Store provisioning profiles
- App Store Connect setup

## Configuration

### Backend Configuration

**Environment Variables** (optional):
- `DATABASE_URL`: SQLite database path (default: `sqlite:///./database.db`)
- `CORS_ORIGINS`: Comma-separated list of allowed origins (default: `*`)
- `LOG_LEVEL`: Logging level (default: `INFO`)

Create a `.env` file or set environment variables:
```bash
export DATABASE_URL="sqlite:///./database.db"
export CORS_ORIGINS="*"
export LOG_LEVEL="INFO"
```

**Note:** Authentication has been removed for simplified home server setup. The backend is designed for local network use.

### Flutter App Configuration

1. **API Configuration** (for multi-device sync): 
   - Open the app and go to Settings
   - Enter the API base URL (default: `http://localhost:8000`)
   - For devices on your network, use your server's IP address (e.g., `http://192.168.1.100:8000`)
   - No authentication token required
   - The app will automatically generate and register a unique device ID on first sync

2. **Notifications**:
   - **Android**: Permissions are requested automatically on first launch
   - **iOS/macOS**: Permissions are requested automatically on first launch
   - **Windows/Linux**: Notifications work automatically if notification server is available

3. **System Tray** (Desktop platforms):
   - The app supports system tray on Linux, Windows, and macOS
   - Minimize to tray is enabled by default
   - Right-click tray icon for menu options

## Development

### Development Script

Use the provided development script to start both backend and Flutter app:

```bash
./start_dev.sh
```

This script will:
- Start the FastAPI backend on port 8000
- Start the Flutter Linux app
- Handle cleanup on exit

### Flutter App Development

**Regenerating Database Code:**
After modifying Isar collections, regenerate the code:
```bash
cd flutter_app
flutter pub run build_runner build --delete-conflicting-outputs
```

**Database schema changes require regenerating Isar code**

**Hot Reload:**
- Press `r` in the terminal to hot reload
- Press `R` to hot restart
- Press `q` to quit

### FastAPI Backend Development

**Adding New Endpoints:**
1. Create a new router in `app/api/`
2. Add the router to `app/main.py`
3. Update schemas in `app/schemas.py` if needed
4. Add models in `app/models.py` if needed

**Database Migrations:**
Currently, the database schema is created automatically. For production, consider using Alembic for migrations.

## API Documentation

Once the server is running, access the interactive API documentation:

- Swagger UI: `http://localhost:8000/docs`
- ReDoc: `http://localhost:8000/redoc`

## API Endpoints

### Authentication

**No authentication required** - The backend is designed for local network/home server use without authentication.

### Main Endpoints

- **Habits**: `/api/habits` - CRUD operations, toggle completion, history tracking
- **Notes**: `/api/notes` - CRUD operations, search, folder association
- **Folders**: `/api/folders` - CRUD operations, nested folders, cascade deletion
- **Reminders**: `/api/reminders` - CRUD operations, recurrence, toggle
- **Calendar**: `/api/calendar/events` - CRUD operations, all-day events
- **Planner**: `/api/planner/blocks` - CRUD operations, day validation
- **Categories**: `/api/categories` - CRUD operations
- **Wiki**: `/api/wiki` - Documents, folders, links, search, backlinks
- **Tasks**: `/api/tasks` - CRUD operations, status transitions, priority
- **Feelings**: `/api/feelings` - CRUD operations, date validation, star rating
- **Analytics**: `/api/analytics` - History, streaks, frequency, overview
- **Sync**: `/api/sync` - Bidirectional sync endpoint with last-write-wins conflict resolution

## Tech Stack

### Frontend (Flutter)
- **Framework**: Flutter (Dart)
- **Database**: Isar (local-first NoSQL database)
- **State Management**: Provider
- **HTTP Client**: http package
- **Device Info**: device_info_plus for device identification
- **Notifications**: flutter_local_notifications
- **System Tray**: tray_manager, window_manager (desktop platforms)
- **Platforms**: Linux, Windows, macOS, Android, iOS

### Backend (FastAPI)
- **Framework**: FastAPI (Python)
- **Database**: SQLite with SQLAlchemy ORM (PostgreSQL supported)
- **Authentication**: None (designed for local network use)
- **Device Tracking**: Automatic device registration and tracking
- **API**: RESTful endpoints with automatic OpenAPI documentation
- **Deployment**: Docker support included

## Architecture

### Local-First Design
- All data is stored locally in Isar database
- App works fully offline - no network required for core functionality
- Changes are marked for sync automatically
- Automatic sync every 15 minutes when connected to server
- Manual sync available in Settings
- Sync on app open and when app comes to foreground

### Sync Strategy
- Bidirectional sync between Flutter app and FastAPI backend
- Last-write-wins conflict resolution
- Device tracking: Each device is automatically registered and tracked
- Efficient incremental sync based on last sync timestamp
- Multi-device support: Sync data across all your devices

## Project Structure

### Backend Structure
```
fastapi_backend/
├── app/
│   ├── __init__.py
│   ├── main.py              # FastAPI app initialization
│   ├── database.py          # SQLAlchemy setup
│   ├── models.py            # SQLAlchemy models
│   ├── schemas.py           # Pydantic schemas
│   ├── auth.py              # Authentication utilities
│   └── api/                 # API route handlers
│       ├── habits.py
│       ├── notes.py
│       ├── folders.py
│       ├── reminders.py
│       ├── calendar.py
│       ├── planner.py
│       ├── categories.py
│       ├── sync.py
│       ├── analytics.py
│       ├── wiki.py
│       ├── tasks.py
│       └── feelings.py
├── requirements.txt
├── docker-compose.yml       # Docker deployment configuration
└── Dockerfile               # Docker image definition
```

### Flutter App Structure
```
lib/
├── main.dart                 # App entry point
├── database/                 # Isar database setup
│   ├── isar_database.dart   # Database initialization
│   └── collections/         # Isar collection schemas
├── models/                   # Data models
├── repositories/             # Repository pattern for data access
├── services/                 # API client, sync service, device service
│   ├── sync_service.dart    # Bidirectional sync logic
│   ├── device_service.dart  # Device ID and name management
│   ├── notification_service.dart  # Platform notifications
│   ├── tray_service.dart    # System tray (desktop)
│   └── periodic_sync_service.dart  # 15-minute sync scheduler
├── screens/                  # UI screens
│   ├── habits/
│   ├── notes/
│   ├── calendar/
│   ├── reminders/
│   ├── planner/
│   └── settings/
└── widgets/                  # Reusable widgets
```

## Troubleshooting

### Backend Issues

1. **Port already in use**: Change the port with `--port 8001` or kill the existing process
2. **Database errors**: Delete `database.db` to recreate the database
3. **Import errors**: Ensure you're in the virtual environment and dependencies are installed
4. **Docker issues**: Check logs with `docker-compose logs -f`

### Flutter App Issues

1. **Sync fails**: 
   - Check API base URL in Settings (no token needed)
   - Ensure backend is running and accessible on your network
   - Check network connectivity
   - Verify your device can reach the server IP address
   - Check firewall settings on server

2. **Database errors**: 
   - Delete the app and reinstall (database will be recreated)
   - Or clear app data on Android/iOS
   - Regenerate database code: `flutter pub run build_runner build --delete-conflicting-outputs`

3. **Build errors**: 
   - Run `flutter clean` and `flutter pub get`
   - Regenerate database code with `build_runner`
   - Check platform-specific prerequisites

4. **Device sync issues**:
   - Each device automatically gets a unique ID on first launch
   - Device name is auto-detected from device info
   - Check Settings to see your device information

5. **Notifications not working**:
   - **Android**: Check notification permissions in app settings
   - **iOS/macOS**: Check notification permissions in system settings
   - **Windows/Linux**: Ensure notification server is running

6. **Platform-specific build issues**:
   - **Windows**: Ensure Visual Studio with C++ tools is installed
   - **macOS/iOS**: Ensure Xcode and CocoaPods are properly installed
   - **Android**: Ensure Android SDK and JDK are properly configured

## Security Notes

- **No authentication required** - Designed for local network/home server use
- CORS is enabled for all origins by default (suitable for local network)
- For internet-facing deployments, consider adding authentication
- Use HTTPS if exposing the server over the internet
- Use a production-grade database (PostgreSQL recommended) for production
- Keep your home server and network secure

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
