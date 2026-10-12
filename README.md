# Server (Backend)

The REST API for this module. **Owner:** the Backend member.

> ⚠️ **Midterm requirement:** every endpoint in [`openapi.yaml`](openapi.yaml) must be **implemented** and testable in **Swagger UI** at `/docs`. It uses **mock data**, so no database is needed yet. See the [Midterm Checklist](#8-midterm-checklist).

## Contents

1. [What you're building](#1-what-youre-building)
2. [Write the API contract first](#2-write-the-api-contract-first)
3. [Choose your framework](#3-choose-your-framework)
4. [Set up the project](#4-set-up-the-project)
5. [Folder structure](#5-folder-structure)
6. [Mock data (no database yet)](#6-mock-data-no-database-yet)
7. [Best practices](#7-best-practices)
8. [Midterm checklist](#8-midterm-checklist)

---

## 1. What you're building

```
  Client (your frontend) ──HTTP + JSON──►  Server (this folder)  ──►  Mock data now
  Other modules' servers ──HTTP + JSON──►                              Database at finals
```

Your server is the **only** way to reach your module's data. The client and the other modules never touch your data directly. They call your API. That's why the API has to be predictable and documented.

## 2. Write the API contract first

Before you write any code, describe every endpoint in [`openapi.yaml`](openapi.yaml).
👉 **Full guide: [API-CONTRACT.md](API-CONTRACT.md)**

**Why first?** The frontend member and the other groups can start working from the contract right away instead of waiting for your code. It's also your to-do list: when every endpoint in the contract works, you're done.

## 3. Choose your framework

Use any backend framework. Pick the one your group knows best, and record the choice in an ADR: `docs/decisions/0001-backend-framework.md`.

| Framework | Language | How you get Swagger UI | Good to know |
|-----------|----------|------------------------|--------------|
| **Express** | JavaScript | `swagger-ui-express` shows your `openapi.yaml` | Very minimal. You decide the structure |
| **FastAPI** | Python | Built in. It creates the docs from your code | Type hints also do input validation |
| **Flask** | Python | `flask-swagger-ui` shows your `openapi.yaml` | Minimal, similar to Express |
| **ASP.NET Core** | C# | `Swashbuckle.AspNetCore.SwaggerUI`. It creates the docs from your code | Strong conventions, lots of built-ins |
| NestJS | TypeScript | `@nestjs/swagger` → `SwaggerModule.setup('docs', app, document)` | |
| Spring Boot | Java | `springdoc-openapi-starter-webmvc-ui`, then set `springdoc.swagger-ui.path=/docs` | |
| Laravel | PHP | `dedoc/scramble` or `darkaonline/l5-swagger` | |
| Django REST Framework | Python | `drf-spectacular` (`SpectacularSwaggerView`) | |

**Every group:** put Swagger UI at **`/docs`** and all endpoints under **`/api/v1`**. This makes grading and integration easier.

## 4. Set up the project

Run these commands inside `server/`. Pick the section for your framework. Nothing is installed yet. Setting it up is part of your job.

<details>
<summary><b>Node.js + Express</b></summary>

Requires Node.js 22 or newer.

```bash
npm init -y
npm pkg set type=module
npm pkg set scripts.dev="node --watch --env-file=.env src/index.js"
npm pkg set scripts.start="node --env-file=.env src/index.js"
npm install express cors swagger-ui-express yaml
cp .env.example .env        # Windows: copy .env.example .env
```

Create `src/index.js`:

```js
import fs from 'node:fs';
import express from 'express';
import cors from 'cors';
import swaggerUi from 'swagger-ui-express';
import YAML from 'yaml';

const app = express();
app.use(cors({ origin: process.env.CORS_ORIGINS?.split(',') }));
app.use(express.json());

// Swagger UI shows your openapi.yaml at /docs
const spec = YAML.parse(fs.readFileSync(new URL('../openapi.yaml', import.meta.url), 'utf8'));
app.use('/docs', swaggerUi.serve, swaggerUi.setup(spec));

app.get('/api/v1/health', (req, res) => res.json({ status: 'ok' }));
// app.use('/api/v1/announcements', announcementsRouter);   // from src/routes/

const port = process.env.PORT || 8000;
app.listen(port, () => console.log(`API docs: http://localhost:${port}/docs`));
```

Run: `npm run dev`
</details>

<details>
<summary><b>Python + FastAPI</b></summary>

Requires Python 3.10 or newer.

```bash
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install "fastapi[standard]" python-dotenv
pip freeze > requirements.txt
cp .env.example .env               # Windows: copy .env.example .env
```

Add an empty `__init__.py` file to `src/` and to every folder inside it. Then create `src/main.py`:

```python
import os

from dotenv import load_dotenv
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

load_dotenv()

app = FastAPI(title="[Module Name] API", version="0.1.0")  # Swagger UI is built in at /docs

app.add_middleware(
    CORSMiddleware,
    allow_origins=os.getenv("CORS_ORIGINS", "").split(","),
    allow_methods=["*"],
    allow_headers=["*"],
)


@app.get("/api/v1/health", tags=["Health"])
def health():
    return {"status": "ok"}


# from src.routes import announcements
# app.include_router(announcements.router, prefix="/api/v1")
```

Run: `fastapi dev src/main.py --port 8000`

⚠️ FastAPI creates Swagger from your **code**, not from `openapi.yaml`. Make sure the two match. See [API-CONTRACT.md → Code-first frameworks](API-CONTRACT.md#code-first-frameworks).
</details>

<details>
<summary><b>Python + Flask</b></summary>

Requires Python 3.10 or newer.

```bash
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install flask flask-cors flask-swagger-ui python-dotenv
pip freeze > requirements.txt
cp .env.example .env               # Windows: copy .env.example .env
```

Add an empty `__init__.py` file to `src/` and to every folder inside it. Then create `src/main.py`:

```python
import os
from pathlib import Path

from flask import Flask, send_from_directory
from flask_cors import CORS
from flask_swagger_ui import get_swaggerui_blueprint

SERVER_DIR = Path(__file__).resolve().parent.parent

app = Flask(__name__)
CORS(app, origins=os.getenv("CORS_ORIGINS", "").split(","))


# Serve the contract file, then show it in Swagger UI at /docs
@app.get("/openapi.yaml")
def openapi_spec():
    return send_from_directory(SERVER_DIR, "openapi.yaml")


app.register_blueprint(get_swaggerui_blueprint("/docs", "/openapi.yaml"))


@app.get("/api/v1/health")
def health():
    return {"status": "ok"}


# from src.routes.announcements import bp as announcements_bp
# app.register_blueprint(announcements_bp, url_prefix="/api/v1/announcements")
```

Run: `flask --app src.main run --debug --port 8000` (Flask reads `.env` automatically when `python-dotenv` is installed)
</details>

<details>
<summary><b>C# + ASP.NET Core</b></summary>

Requires the .NET 9 SDK or newer.

```bash
dotnet new webapi --use-controllers -n [ModuleName]Api -o .
dotnet add package Swashbuckle.AspNetCore.SwaggerUI
```

ASP.NET Core has its own folder names, so **delete `src/`** and use `Controllers/`, `Services/`, `Data/`, `Models/`, and `Middleware/` at the project root (see [Folder structure](#5-folder-structure)). Settings go in `appsettings.json` instead of `.env`, so you can delete `.env.example`.

Update `Program.cs`:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddOpenApi();                                // creates /openapi/v1.json from your code
builder.Services.AddProblemDetails();                         // errors use the Problem Details format
builder.Services.AddRouting(o => o.LowercaseUrls = true);     // /api/v1/announcements, not /Announcements
builder.Services.AddCors(o => o.AddDefaultPolicy(p => p
    .WithOrigins(builder.Configuration.GetSection("CorsOrigins").Get<string[]>() ?? [])
    .AllowAnyHeader()
    .AllowAnyMethod()));

var app = builder.Build();

app.MapOpenApi();
app.UseSwaggerUI(o =>
{
    o.SwaggerEndpoint("/openapi/v1.json", "v1");
    o.RoutePrefix = "docs";                                   // Swagger UI at /docs
});

app.UseExceptionHandler();
app.UseCors();
app.MapControllers();
app.MapGet("/api/v1/health", () => new { status = "ok" });

app.Run();
```

In `appsettings.json`, add `"CorsOrigins": ["http://localhost:5173"]`. In `Properties/launchSettings.json`, set `applicationUrl` to `http://localhost:8000`. On each controller, use `[Route("api/v1/[controller]")]`.

Run: `dotnet watch`

⚠️ ASP.NET Core creates Swagger from your **code**, not from `openapi.yaml`. Make sure the two match. See [API-CONTRACT.md → Code-first frameworks](API-CONTRACT.md#code-first-frameworks).
</details>

<details>
<summary><b>Any other framework: show openapi.yaml with plain HTML</b></summary>

If your framework has no Swagger package, set it up to serve two static files: `openapi.yaml` and this page at `/docs`:

```html
<!doctype html>
<html>
  <head>
    <title>API Docs</title>
    <link rel="stylesheet" href="https://unpkg.com/swagger-ui-dist@5/swagger-ui.css">
  </head>
  <body>
    <div id="swagger-ui"></div>
    <script src="https://unpkg.com/swagger-ui-dist@5/swagger-ui-bundle.js"></script>
    <script>SwaggerUIBundle({ url: '/openapi.yaml', dom_id: '#swagger-ui' });</script>
  </body>
</html>
```
</details>

When the server runs, open **`http://localhost:<port>/docs`**. You should see Swagger UI.

## 5. Folder structure

```
server/
├── openapi.yaml       # API contract. Write it first
├── API-CONTRACT.md    # How to write the contract
├── .env.example       # List of environment variables (copy it to .env)
└── src/
    ├── routes/        # HTTP layer: URL + method → handler. Reads the request, sends the response
    ├── services/      # Business rules. No HTTP code here
    ├── data/          # Mock data + functions that read/write it. Becomes the database layer later
    ├── models/        # Data shapes: request and response schemas
    ├── middleware/    # Runs on every request: error handler, logging, later authentication
    ├── config/        # Reads environment variables in one place
    └── (entry file)   # index.js / main.py
```

Each folder has a short README explaining what goes in it.

**The one rule:** requests flow **routes → services → data**, and no layer is skipped (routes never read mock data directly). Why? At finals, when you add a database, you only rewrite `data/`. Everything else stays the same.

### Same layers, different names

Frameworks use different names for the same layers. Use your framework's names, but keep the layers.

| Layer | Express | FastAPI | Flask | ASP.NET Core |
|-------|---------|---------|-------|--------------|
| routes | `src/routes/announcements.routes.js` (can split handlers into `src/controllers/`) | `src/routes/announcements.py` (an `APIRouter`) | `src/routes/announcements.py` (a `Blueprint`) | `Controllers/AnnouncementsController.cs` |
| services | `src/services/announcements.service.js` | `src/services/announcements.py` | `src/services/announcements.py` | `Services/AnnouncementService.cs` |
| data | `src/data/announcements.repository.js` | `src/data/announcements.py` | `src/data/announcements.py` | `Data/AnnouncementRepository.cs` |
| models | `src/models/` (JSDoc or a validation library) | `src/models/announcement.py` (Pydantic) | `src/models/` (dataclasses or Pydantic) | `Models/AnnouncementDtos.cs` |
| middleware | `src/middleware/error-handler.js` | exception handlers in `src/middleware/` | `@app.errorhandler` in `src/middleware/` | `Middleware/` or built-in `AddProblemDetails()` |
| entry | `src/index.js` | `src/main.py` | `src/main.py` | `Program.cs` |

## 6. Mock data (no database yet)

For the midterm, your endpoints must **really work**. They just store data in memory instead of a database.

- Put the mock data in `src/data/`, either as an in-memory list or a JSON file loaded when the server starts.
- Make **5–10 realistic records** per resource with real-looking names, dates, and IDs, and follow the schemas in `openapi.yaml` exactly.
- `GET` reads from the list. `POST` adds to it (generate the `id` and dates) and returns `201`. `PATCH` updates an item. `DELETE` removes one and returns `204`.
- Filters and pagination (`?status=...&page=2`) must actually work on the list.
- Validation and errors must be real too: missing fields → `400`, unknown ID → `404`.
- For another module's data (e.g., a `studentId` from the Registrar), use fake IDs that follow the format agreed on in `docs/integration.md`.
- The data resets every time the server restarts. That's fine for the midterm.

## 7. Best practices

1. **Follow the contract exactly:** same paths, field names, and status codes. Need to change something? Update `openapi.yaml` first and tell the frontend member.
2. **Keep route handlers thin.** They read the request, call a service, and send the response. The rules live in `services/`.
3. **Validate every input** (body, path parameters, query parameters). Never trust the client.
4. **Handle errors in one place** (`middleware/`). Always return Problem Details JSON (see [API-CONTRACT.md](API-CONTRACT.md#error-format-problem-details)). Never send stack traces to the client.
5. **Use the right status codes.** See the table in [API-CONTRACT.md](API-CONTRACT.md#http-methods-and-status-codes).
6. **Use environment variables for configuration.** No hardcoded ports, URLs, or passwords. Every variable goes in `.env.example` with a safe example value.
7. **Use camelCase for JSON fields** (`studentId`, `createdAt`). This is the same for all modules.
   - FastAPI/Pydantic: `model_config = ConfigDict(alias_generator=to_camel, populate_by_name=True)` (`to_camel` comes from `pydantic.alias_generators`)
   - ASP.NET Core: camelCase by default
   - Express/Flask: write the keys in camelCase
8. **Only allow your own frontend** (and modules that call you from a browser) in `CORS_ORIGINS`.
9. **Log each request and each error** to the console. It saves hours of debugging.
10. **Commit your dependency list** (`package-lock.json`, `requirements.txt`, `.csproj`) so teammates install the same versions.
11. **Test every endpoint** with Swagger UI's "Try it out" button, including the error cases.

## 8. Midterm checklist

- [ ] `openapi.yaml` lists **every** endpoint the module needs, with schemas, examples, and error responses
- [ ] `openapi.yaml` has **no errors** in the Swagger Editor or in `npx @redocly/cli lint openapi.yaml`
- [ ] The frontend member reviewed the contract (and so did any group that calls your API)
- [ ] **Every** endpoint in the contract is implemented and returns mock data in the correct shape
- [ ] Swagger UI runs at `http://localhost:<port>/docs`, and "Try it out" works for every endpoint
- [ ] Bad input returns `400`, and unknown IDs return `404`, both as Problem Details
- [ ] `GET /api/v1/health` returns `{ "status": "ok" }`
- [ ] A teammate can run the server using only the root README's Getting Started steps
- [ ] `.env.example` is committed, and `.env` is **not**
