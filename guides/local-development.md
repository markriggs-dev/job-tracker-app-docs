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
git clone https://github.com/your-username/job-tracker-app-job-service-consumer
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
docker compose up -d postgres kafka minio
```

The `kafka-init` service runs automatically as part of `docker compose up` and pre-creates the required Kafka topics (`job.application.created`, `job.application.updated`) before any consuming service starts. No manual `kafka-topics` commands are needed.

## Step 4: Run services locally

Each service runs in its own terminal. Start them individually with `dotnet run`:

| Service | Command | Port |
|---------|---------|------|
| API Gateway | `dotnet run --project job-tracker-app-gateway/src/Gateway.Api` | 5000 |
| Job Service | `dotnet run --project job-tracker-app-job-service/src/JobService.Api` | 5153 |
| Job Service Consumer | `dotnet run --project job-tracker-app-job-service-consumer/src/JobServiceConsumer.Api` | 5290 |
| Contact Service | `dotnet run --project job-tracker-app-contact-service/src/ContactService.Api` | 5297 |
| Journal Service | `dotnet run --project job-tracker-app-journal-service/src/JournalService.Api` | 5241 |
| Resume Service | `dotnet run --project job-tracker-app-resume-service/src/ResumeService.Api` | 5271 |
| Notification Service | `dotnet run --project job-tracker-app-notification-service/src/NotificationService.Api` | 5261 |

All frontend traffic flows through the API Gateway on port 5000. Individual service ports are not accessed by the frontend directly.

## Step 5: Start the React frontend

```bash
cd job-tracker-app-web-react
cp .env.example .env.local
npm install
npm run dev
```

The application will be available at `http://localhost:5173`.
