# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ImageFlow is a modern image service system that provides efficient image management and distribution. It's a full-stack application with a Go backend and Next.js frontend that automatically optimizes images based on device type and browser compatibility.

## Architecture

### Backend (Go 1.22+)
- **Main Service**: HTTP server with CORS middleware and graceful shutdown
- **Image Processing**: Uses libvips (via bimg) for high-performance image conversion to WebP/AVIF
- **Storage**: Supports both local filesystem and S3-compatible storage
- **Metadata**: Redis-based metadata storage with file-based fallback
- **Worker Pool**: Asynchronous image processing with configurable concurrency
- **Authentication**: API key-based authentication for upload/management endpoints

### Frontend (Next.js 14)
- **App Router**: Modern Next.js with TypeScript and Tailwind CSS
- **Static Export**: Built as static site and embedded in Go binary
- **Components**: React components with drag-and-drop upload, masonry layout, and dark mode
- **Device Detection**: Automatic landscape/portrait image selection

### Key Dependencies
- **Backend**: bimg (libvips), AWS SDK v2, Redis client, Zap logger
- **Frontend**: Next.js 14, React 18, Tailwind CSS, Framer Motion, Lucide icons

## Development Commands

### Backend Development
```bash
# Start Go backend in development
go run main.go

# Build production binary  
go build -o imageflow

# Install Go dependencies
go mod tidy
```

### Frontend Development  
```bash
# Navigate to frontend directory
cd frontend

# Install dependencies
npm install

# Development server
npm run dev

# Build for production (static export)
npm run build

# Lint frontend code
npm run lint

# Build frontend (using build script)
bash build.sh    # Linux/Mac
build.bat        # Windows
```

### Docker Deployment
```bash
# Backend-only deployment (embedded frontend)
docker-compose up -d

# Frontend-backend separated (pre-built images)
docker-compose -f docker-compose-separate.yaml up -d

# Frontend-backend separated (local build)
docker-compose -f docker-compose-separate-build.yaml up --build -d
```

### Database Migration
```bash
# Migrate metadata from files to Redis
bash migrate.sh

# Force migration
bash migrate.sh --force

# Specify custom .env file
bash migrate.sh --env /path/to/.env
```

## Configuration

The service is configured via environment variables in `.env` file:

### Required Settings
- `API_KEY`: Authentication key for upload/management endpoints
- `STORAGE_TYPE`: `local` or `s3` for storage backend
- `LOCAL_STORAGE_PATH`: Directory for local image storage (default: `static/images`)

### Redis Configuration  
- `REDIS_ENABLED`: Enable Redis for metadata storage
- `REDIS_HOST`, `REDIS_PORT`, `REDIS_PASSWORD`: Redis connection settings
- `METADATA_STORE_TYPE`: Currently only `redis` is supported

### S3 Configuration (when STORAGE_TYPE=s3)
- `S3_ENDPOINT`, `S3_REGION`, `S3_BUCKET`: S3 connection details
- `S3_ACCESS_KEY`, `S3_SECRET_KEY`: S3 credentials  
- `CUSTOM_DOMAIN`: Optional custom domain for S3 assets

### Image Processing
- `MAX_UPLOAD_COUNT`: Max images per upload request (default: 20)
- `IMAGE_QUALITY`: Conversion quality 1-100 (default: 80)
- `WORKER_THREADS`: Parallel processing threads (default: 4)
- `SPEED`: Encoding speed 0-8 (default: 5)

## API Endpoints

### Public Endpoints
- `GET /api/random` - Get random image with advanced filtering options:
  - `tag=tag1` or `tags=tag1,tag2,tag3` - Filter by tags (AND logic for multiple tags)
  - `exclude=nsfw,private` - Exclude images with specified tags 
  - `orientation=portrait|landscape` - Force specific orientation (overrides device detection)
  - `format=avif|webp|original` - Prefer specific image format
  - Device-based orientation: Mobile devices get portrait by default, desktop gets landscape
  - Example: `/api/random?tags=nature,sunset&exclude=nsfw&orientation=landscape&format=webp`
- `POST /api/validate-api-key` - Validate API key

### Authenticated Endpoints (require API key header)
- `POST /api/upload` - Upload images with optional expiry and tags
- `GET /api/images` - List uploaded images (optional `?tag=` filter) 
- `POST /api/delete-image` - Delete specific image
- `GET /api/config` - Get system configuration
- `GET /api/tags` - List all available tags
- `POST /api/trigger-cleanup` - Manually trigger expired image cleanup

## Project Structure

```
├── main.go              # Main application entry point
├── config/             
│   └── config.go       # Configuration loading and validation
├── handlers/           # HTTP request handlers
│   ├── auth.go         # API key authentication middleware
│   ├── upload.go       # Image upload handling
│   ├── random.go       # Random image serving
│   └── *.go            # Other API handlers
├── utils/              # Backend utilities
│   ├── converter_bimg.go    # Image conversion using libvips
│   ├── storage.go      # Storage interface (local/S3)
│   ├── redis.go        # Redis metadata operations
│   ├── worker_pool.go  # Async processing pool
│   └── cleaner.go      # Expired image cleanup
├── frontend/           # Next.js application
│   ├── app/           # Next.js app directory
│   │   ├── components/ # React components
│   │   ├── hooks/     # Custom React hooks
│   │   └── utils/     # Frontend utilities
│   ├── package.json   # Frontend dependencies
│   └── build.sh       # Frontend build script
├── static/            # Generated static files and image storage
├── logs/              # Application logs
└── docker-compose*.yaml  # Docker deployment configurations
```

## Development Workflow

1. **Local Development**: Use `go run main.go` for backend and `npm run dev` in frontend/ for frontend development
2. **Image Processing**: Images are automatically converted to WebP/AVIF in background worker pool
3. **Testing**: Upload images via web interface or API, verify format conversion and device-appropriate serving
4. **Deployment**: Use Docker Compose for production deployment with Redis and persistent storage

## Prerequisites

- Go 1.22+ for backend development
- Node.js 18+ for frontend development  
- libvips development libraries for image processing
- Redis (optional but recommended for metadata storage)
- Docker and Docker Compose for containerized deployment