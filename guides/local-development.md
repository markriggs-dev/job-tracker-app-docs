# Local development setup

## Prerequisites

- [.NET 8 SDK](https://dotnet.microsoft.com/download)
- [Node.js 20+](https://nodejs.org)
- [Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Visual Studio Code](https://code.visualstudio.com) with Claude Code extension
- An [Auth0](https://auth0.com) account (free tier)

## Step 1: Clone all repositories

```bash
git clone https://github.com/your-username/job-tracker-app-infrastructure
git clone https://github.com/your-username/job-tracker-app-gateway
git clone https://github.com/your-username/job-tracker-app-job-service
git clone https://github.com/your-username/job-tracker-app-contact-service
git clone https://github.com/your-username/job-tracker-app-journal-service
git clone https://github.com/your-username/job-tracker-app-resume-service
git clone https://github.com/your-username/job-tracker-app-notification-service
git clone https://github.com/your-username/job-tracker-app-ai-service
git clone https://github.com/your-username/job-tracker-app-user-service
git clone https://github.com/your-username/job-tracker-app-web-react
git clone https://github.com/your-username/job-tracker-app-shared
```

All repos should be cloned into the same parent directory.

## Step 2: Configure environment variables

```bash
cd job-tracker-app-infrastructure
cp .env.example .env
```

Edit `.env` and fill in your Auth0 credentials and API keys.

## Step 3: Start infrastructure dependencies

```bash
cd job-tracker-app-infrastructure
docker-compose up -d postgres kafka minio
```

## Step 4: Run services locally

Each service can be run independently:

```bash
cd job-tracker-app-job-service
dotnet run --project src/JobService.Api
```

Or bring up all services via Docker Compose:

```bash
docker-compose up -d
```

## Step 5: Start the React frontend

```bash
cd job-tracker-app-web-react
cp .env.example .env.local
npm install
npm run dev
```

The application will be available at `http://localhost:5173`.
