# Product Brief: Professional Portfolio Landing Page

## Purpose of this document

This brief is the starting context for a new chat session focused exclusively on
building a professional portfolio landing page. Feed this document into a fresh
Claude Code session to begin implementation.

---

## About the developer

- **Name:** Mark Riggs
- **GitHub:** github.com/markriggs-labs
- **Role target:** Senior software engineer / architect
- **Background:** Experienced .NET / C# engineer building a portfolio of projects
  to demonstrate distributed systems, microservices, AI integration, and modern
  engineering practices
- **Current situation:** Actively job searching; the landing page and portfolio
  apps are the primary showcase for landing interviews

---

## What this page is

A professional, high-impact personal portfolio landing page hosted on GitHub Pages
at `https://markriggs-dev.github.io`. It is the first thing a hiring manager,
recruiter, or technical lead sees when they follow a link from a resume or LinkedIn
profile. It must immediately convey:

- Technical depth and breadth
- Real engineering decisions (architecture, tradeoffs)
- Working, deployable software (not toy examples)
- Professional presentation on par with a well-designed product site

---

## Hosting

- **Repo name:** `markriggs-dev.github.io` (GitHub user page repo)
- **URL:** `https://markriggs-dev.github.io`
- **Deployment:** Static files pushed to the `main` branch — GitHub Pages serves
  them automatically, no build pipeline needed initially
- **No framework required:** Pure HTML, CSS, and vanilla JS only

---

## Technology decision

Pure HTML/CSS with minimal vanilla JavaScript. No React, no build step, no
bundler. Reasons:

- Landing pages are marketing surfaces, not applications — no state management needed
- Loads faster, scores better on Core Web Vitals
- Demonstrates knowing the right tool for the job
- GitHub Pages serves static files directly — zero deployment friction
- Modern CSS (Grid, custom properties, `@keyframes`, scroll-driven animations)
  is capable of producing visually impressive results without a framework

---

## Design direction

- **Tone:** Polished, technical, confident — like a well-designed SaaS product
  site, not a personal blog
- **Feel:** Dark theme preferred (common in developer tooling / enterprise SaaS);
  strong typography; subtle animations that add professionalism without being distracting
- **Audience:** Technical hiring managers, senior engineers, and recruiters at
  software companies — they will recognise good engineering and good design
- **Must avoid:** Template-looking layouts, clip art icons, walls of text,
  anything that reads as "beginner portfolio"

---

## Page structure

Single page with smooth scroll navigation. Each nav item scrolls to its section.
No full page reloads. A sticky header with nav links stays visible while scrolling.

### Sections (in order)

#### 1. Hero
- Full-viewport opening section
- Name, professional title (e.g. "Senior Software Engineer")
- One sharp tagline that captures the engineering story
  (e.g. "Distributed systems. Real problems. Production-grade code.")
- Two CTA buttons: "View My Work" (scrolls to Portfolio) and "GitHub" (opens
  github.com/markriggs-labs in new tab)

#### 2. Tech Stack
- Visual grid or icon row of technologies
- .NET 8 / C#, React, TypeScript, Apache Kafka, PostgreSQL, Docker,
  Kubernetes, Auth0, SignalR, YARP, Entity Framework Core, MinIO,
  REST APIs, GitHub Actions (CI/CD — planned)
- Brief framing sentence: not a list of buzzwords but a set of tools used
  together in a real distributed system

#### 3. Architecture
- High-level diagram or visual representation of the microservices architecture
- Could be an SVG diagram or a clean HTML/CSS layout showing:
  - React frontend → API Gateway (YARP) → microservices
  - Kafka async messaging layer
  - PostgreSQL, MinIO storage
  - Auth0 identity
- Brief narrative: why these choices were made (Kafka for async decoupling,
  gateway for single entry point, etc.)
- Link to full ADR documentation (GitHub Pages docs site or GitHub repo)

#### 4. Portfolio Applications
- Card-based grid layout, one card per application
- Each card contains:
  - App name and icon/logo
  - 2–3 sentence summary of what it does and what it demonstrates technically
  - Key tech badges (e.g. .NET 8, Kafka, SignalR, React)
  - "Launch" button — opens the app's Auth0 login page in a new browser tab
  - "Docs" link — opens the app's documentation (GitHub or GitHub Pages)
- **Current app: Job Tracker**
  - Summary: Full-stack job search tracker built as a distributed microservices
    system. Demonstrates Kafka async messaging, SignalR real-time push, YARP
    reverse proxy, Auth0 authentication, and a React frontend — all running as
    independently deployable services behind an API gateway.
  - Tech: .NET 8, React, TypeScript, Apache Kafka, SignalR, PostgreSQL, Docker,
    Auth0, YARP, MinIO, EF Core
  - Launch URL: the Auth0 login page for the deployed job tracker app
  - (Additional portfolio apps will be added as cards over time)

#### 5. Docs
- Links to key documentation artifacts that demonstrate engineering rigour
- Architecture Decision Records (ADRs): links to individual ADR markdown files
  in the GitHub repo
- Local development guide
- Repository overview
- These open in new tabs pointing at the GitHub-hosted markdown files

#### 6. Contact / Connect
- GitHub profile link
- LinkedIn profile link (placeholder — update with actual URL)
- Email (optional — developer's preference)
- Simple, clean — not a full contact form

---

## Navigation

- Sticky header, visible on scroll
- Nav links: Work / Tech / Architecture / Portfolio / Docs / Contact
- Smooth scroll to section on click
- Active section highlighted in nav as user scrolls (Intersection Observer)
- Mobile hamburger menu for small screens

---

## Portfolio app connection

The Job Tracker React app is deployed separately at:
`https://markriggs-labs.github.io/job-tracker-app-web-react`
(or the production VPS URL once deployed to Akamai)

The landing page "Launch" button for Job Tracker opens this URL in a new tab.
The user lands on the Auth0 login page for the job tracker application.

---

## Repo structure

```
markriggs-dev.github.io/
  index.html
  css/
    styles.css
  js/
    main.js
  assets/
    icons/
    diagrams/
  README.md
```

---

## What is already built (context for the new session)

The Job Tracker Application referenced in the Portfolio section is a real,
complete distributed system with the following repos on GitHub under
`github.com/markriggs-labs`:

| Repo | Purpose |
|------|---------|
| job-tracker-app-gateway | API Gateway (YARP reverse proxy, Auth0 JWT) |
| job-tracker-app-job-service | Job tracking service (Kafka publisher, 202 pattern) |
| job-tracker-app-job-service-consumer | Kafka consumer, EF Core DB writer, SignalR hub |
| job-tracker-app-contact-service | Contact management per job application |
| job-tracker-app-journal-service | Journal/interaction log per job application |
| job-tracker-app-resume-service | Resume file storage (MinIO) |
| job-tracker-app-notification-service | Kafka consumer, logging (email future) |
| job-tracker-app-web-react | React + TypeScript frontend (Vite) |
| job-tracker-app-infrastructure | Docker Compose, Nginx Proxy Manager config |
| job-tracker-app-docs | ADRs, user stories, guides |

Key architectural patterns demonstrated:
- Publisher-only Job Service returns 202 immediately; consumer handles DB write
- Kafka topics: `job.application.created`, `job.application.updated`
- SignalR hub at `/hubs/jobs` pushes `jobCreated`/`jobUpdated` to React via
  TanStack Query invalidation
- All traffic routed through YARP gateway on port 5000
- Auth0 JWT validated at the gateway; downstream services also validate
- EF Core migrations owned by Job Service; Consumer shares the schema

---

## Success criteria

A hiring manager who lands on this page should:
1. Immediately understand who Mark is and what level of engineer he is
2. Be able to navigate to the architecture section and understand the system design
3. Click Launch on the Job Tracker and be taken to a working application
4. Leave with the impression that this is production-quality engineering work,
   not a tutorial project
