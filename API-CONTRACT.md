# Writing the API Contract (OpenAPI)

[`openapi.yaml`](openapi.yaml) is your **API contract**. It's a promise that says: *"These are our endpoints. This is what you send. This is what you get back."* It's written in the **OpenAPI** format, the industry standard for describing REST APIs.

**Who uses it:**

- **Frontend member:** builds screens against it before the backend is finished.
- **Other groups:** learn how to get data from your module.
- **You (backend):** it's your to-do list of endpoints to implement.
- **Swagger UI:** turns it into interactive docs where anyone can try the endpoints.

## Contents

- [The workflow](#the-workflow)
- [From features to endpoints](#from-features-to-endpoints)
- [Class-wide API conventions](#class-wide-api-conventions)
- [HTTP methods and status codes](#http-methods-and-status-codes)
- [Error format (Problem Details)](#error-format-problem-details)
- [How openapi.yaml is organized](#how-openapiyaml-is-organized)
- [Adding your own resource, step by step](#adding-your-own-resource-step-by-step)
- [Check your contract for errors](#check-your-contract-for-errors)
- [Mock server: try the API before the code exists](#mock-server-try-the-api-before-the-code-exists)
- [Changing the contract later](#changing-the-contract-later)
- [Code-first frameworks](#code-first-frameworks)
- [Adding login later (finals)](#adding-login-later-finals)

---

## The workflow

1. **List the features** your module needs (from the Features section of the root README).
2. **Find the resources.** These are the *nouns* in your features (books, loans, payments, visits).
3. **Plan the endpoints** in a table (see the next section).
4. **Define the data shapes (schemas)** for each resource.
5. **Write them into `openapi.yaml`.** Copy the Announcements example.
6. **Check it for errors** (validate).
7. **Review it** with your frontend member and with any group that will call your API.
8. **Implement** each endpoint in code.
9. **Verify in Swagger UI** that every endpoint works exactly as the contract says.

## From features to endpoints

Each feature (what a user wants to do) turns into one or more endpoints. Example with announcements:

| Feature (user story) | Method | Path | Success code |
|----------------------|--------|------|:------------:|
| Staff posts an announcement | `POST` | `/announcements` | 201 |
| Anyone sees the latest announcements | `GET` | `/announcements?audience=students&page=1` | 200 |
| Anyone opens one announcement | `GET` | `/announcements/{announcementId}` | 200 |
| Staff edits an announcement | `PATCH` | `/announcements/{announcementId}` | 200 |
| Staff deletes an announcement | `DELETE` | `/announcements/{announcementId}` | 204 |
| Staff archives an announcement | `POST` | `/announcements/{announcementId}/archive` | 200 |

**Tip:** most resources need the 5 basic endpoints (list, get one, create, update, delete), known as **CRUD**. Then add **action endpoints** for things that aren't simple CRUD: `POST /loans/{loanId}/return`, `POST /payments/{paymentId}/void`, `POST /requests/{requestId}/approve`.

**How many endpoints is "all of them"?** Every feature in your README must be possible through at least one endpoint, and every piece of data a screen shows must come from an endpoint. Nothing on a screen should be hardcoded.

Copy this table into your notes and plan your endpoints:

| Feature | Method | Path | Request body | Success response | Possible errors |
|---------|--------|------|--------------|------------------|-----------------|
| | | | | | |

## Class-wide API conventions

**All 8 modules follow these rules.** When every API looks the same, integrating modules is much easier.

| Rule | Convention | Example |
|------|------------|---------|
| Base path | `/api/v1` | `http://localhost:8002/api/v1/books` |
| Resource names | Plural nouns, lowercase, words separated by hyphens. **No verbs** | `/course-offerings`, not `/getCourseOfferings` |
| IDs in the path | `{resourceNameId}` | `/books/{bookId}` |
| JSON field names | camelCase | `studentId`, `createdAt` |
| IDs | Strings | `"BK-0001"`, `"2026-00123"` |
| Dates and times | ISO 8601 strings, in UTC | `"2026-09-25T08:00:00Z"`, or `"2026-09-25"` for a date with no time |
| Money | Number in pesos, at most 2 decimals | `"amount": 1500.50` |
| Lists | Wrapped with pagination info | `{ "data": [...], "meta": { "page": 1, "limit": 20, "total": 42 } }` |
| Pagination | `?page=` and `?limit=` | `/books?page=2&limit=20` |
| Filtering | Query parameters named after the field | `/loans?status=overdue&studentId=2026-00123` |
| Sorting | `?sort=field`, with `-field` for descending | `/books?sort=-createdAt` |
| Errors | Problem Details (see below) | |
| Health check | `GET /api/v1/health` | `{ "status": "ok" }` |
| Swagger UI | `/docs` | `http://localhost:8002/docs` |

## HTTP methods and status codes

| Method | Use it to | Example |
|--------|-----------|---------|
| `GET` | Read data. Never changes anything | `GET /books/BK-0001` |
| `POST` | Create something, or trigger an action | `POST /books`, `POST /loans/L-01/return` |
| `PATCH` | Update **some** fields | `PATCH /books/BK-0001` with `{ "title": "New" }` |
| `PUT` | Replace the **whole** item (less common, `PATCH` is usually enough) | |
| `DELETE` | Remove something | `DELETE /books/BK-0001` |

| Code | Meaning | When to use it |
|------|---------|----------------|
| `200 OK` | Success | `GET`, `PATCH`, and action endpoints |
| `201 Created` | Something new was created | `POST` that creates. Return the new item |
| `204 No Content` | Success, nothing to return | `DELETE` |
| `400 Bad Request` | The client sent invalid data | Missing field, wrong type, bad query parameter |
| `401 Unauthorized` | Not logged in | Later, when you add login |
| `403 Forbidden` | Logged in, but not allowed | Later, e.g., a student calling a staff-only endpoint |
| `404 Not Found` | That ID doesn't exist | `GET /books/BK-9999` |
| `409 Conflict` | Can't do it in the item's current state | Borrowing a book that's already on loan |
| `500 Internal Server Error` | A bug in the server | Never send this on purpose. The error handler sends it |

> **FastAPI note:** FastAPI returns `422` for validation errors by default. Either add an exception handler that returns `400`, or write `422` in your contract. **The contract and the real behavior must match.**

## Error format (Problem Details)

Every error, from every module, uses this JSON shape. It follows the standard [RFC 9457](https://www.rfc-editor.org/rfc/rfc9457), and the response `Content-Type` is `application/problem+json`:

```json
{
  "type": "about:blank",
  "title": "Bad Request",
  "status": 400,
  "detail": "The request body has invalid fields.",
  "errors": {
    "title": ["title is required"],
    "audience": ["must be one of: everyone, students, faculty, staff"]
  }
}
```

- `title` and `status` are always present. `detail` explains this specific error.
- `errors` is only for validation errors (400). It maps each field name to a list of messages.
- ASP.NET Core does this automatically. In other frameworks, write one error handler in `src/middleware/` (see its README).

## How openapi.yaml is organized

```yaml
openapi: 3.0.3            # Version of the OpenAPI format (not your API's version)

info:                     # Title, version, and description of YOUR API
servers:                  # Base URL(s), e.g. http://localhost:8000/api/v1
tags:                     # Groups of endpoints (one per resource). Swagger UI groups by tag

paths:                    # THE ENDPOINTS
  /announcements:         #   a path
    get:                  #     a method on that path = one "operation"
      summary:            #       short description, shown in Swagger UI
      operationId:        #       unique name, e.g. listAnnouncements
      parameters:         #       path, query, or header parameters
      requestBody:        #       what the client sends (POST/PATCH)
      responses:          #       every possible response, by status code

components:               # REUSABLE PIECES, referenced with $ref
  schemas:                #   data shapes (Announcement, AnnouncementCreate, ...)
  parameters:             #   reusable parameters (page, limit, IDs)
  responses:              #   reusable responses (BadRequest, NotFound, ...)
```

`$ref: "#/components/schemas/Announcement"` means "use the Announcement shape defined below". Define a shape **once** and reuse it everywhere.

YAML tips: **indentation matters** (use spaces, never tabs), and put text that contains `:` or `#` in quotes.

## Adding your own resource, step by step

Use the Announcements example in `openapi.yaml` as your pattern. For a new resource (say, `books`):

1. **Add a tag:** `- name: Books`
2. **Add the schemas** under `components/schemas`:
   - `Book`: what the server **returns** (includes `id`, `createdAt`, and other fields the server sets)
   - `BookCreate`: what the client **sends** to create one (no `id` or dates)
   - `BookUpdate`: what the client sends to update one (all fields optional)
   - `BookList`: `{ data: Book[], meta: PageMeta }`
3. **Add the ID parameter** under `components/parameters`: `BookId`
4. **Add the paths:** `/books` (get, post) and `/books/{bookId}` (get, patch, delete), plus any action endpoints.
5. **Reuse the error responses:** `$ref: "#/components/responses/BadRequest"` (and `NotFound`, `Conflict`).
6. **Delete the Announcements example** once you have your own.

**For every operation, include:**

- [ ] `tags`, `summary`, and a unique `operationId` (a verb plus a noun: `listBooks`, `getBook`, `createBook`)
- [ ] At least one success response (`2xx`) and every error it can return (`4xx`)
- [ ] `example` values, which Swagger UI and the mock server show

**For every schema, include:**

- [ ] `required`: which fields must be present
- [ ] The correct `type`, plus `format` where it helps (`date-time`, `date`, `email`)
- [ ] `enum` for fixed lists of values (`[pending, approved, rejected]`)
- [ ] Limits where they make sense (`minLength`, `maxLength`, `minimum`, `maximum`)

## Check your contract for errors

Use any of these:

- **Online:** paste the file into <https://editor.swagger.io>. Errors appear at the top, and the right side shows a Swagger UI preview.
- **VS Code:** install the "OpenAPI (Swagger) Editor" extension (by 42Crunch). It underlines errors and has a preview.
- **Command line** (needs Node.js), inside `server/`:
  ```bash
  npx @redocly/cli lint openapi.yaml
  ```
  It must report **no errors** before the midterm. These 3 warnings are fine for a class project and can be ignored: missing `license`, `localhost` in the server URL, and `/health` having no `4XX` response. Fix any other warnings.

## Mock server: try the API before the code exists

Prism reads `openapi.yaml` and starts a fake API that returns your `example` values:

```bash
npx @stoplight/prism-cli mock openapi.yaml
```

It prints the URL it's running on. The **frontend member** can build against this URL while the real backend is still being written. Just switch the URL when the real server is ready. This is the main reason to write the contract first.

## Changing the contract later

Once the frontend or other groups depend on your contract, changes affect them:

| Change | Safe? | What to do |
|--------|:-----:|------------|
| Add a new endpoint | ✅ | Update the contract, then mention it to the group |
| Add an **optional** field | ✅ | Same as above |
| Rename or remove a field, change a type, make a field required | ⚠️ Breaks other people's code | Tell the frontend member and every group that uses it **before** merging. Increase `info.version` and write it in the Agreements Log in `docs/integration.md` |

Contract changes go in their own commit, e.g. `feat(server): add POST /loans/{loanId}/return to contract`.

## Code-first frameworks

**FastAPI, ASP.NET Core, NestJS, Spring Boot, and Django REST** generate the Swagger docs **from your code**, so their Swagger UI does not read `openapi.yaml`.

You still write `openapi.yaml` first. It's the plan the frontend and the other groups rely on. Then write your code so that the **generated** docs match the contract:

- Same paths, methods, field names, required fields, and status codes.
- Before each presentation, open your Swagger UI and `openapi.yaml` side by side and compare them endpoint by endpoint.

Tips that make the generated docs match:

- **FastAPI:** `APIRouter(prefix="/announcements", tags=["Announcements"])`, `response_model=Announcement`, `status_code=201`, `summary="..."`, and `responses={404: {"model": ProblemDetails}}` to document errors.
- **ASP.NET Core:** `[Route("api/v1/[controller]")]`, `[ProducesResponseType<Announcement>(StatusCodes.Status200OK)]`, `[ProducesResponseType(StatusCodes.Status404NotFound)]`, and XML comments or `[EndpointSummary("...")]` for summaries.

## Adding login later (finals)

Login isn't needed for the midterm, which is why `openapi.yaml` has `security: []` (no login required). When you add login, replace that line with this:

```yaml
components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

security:
  - bearerAuth: []     # every endpoint needs a token, unless an operation sets `security: []`
```

Swagger UI then shows an **Authorize** button where you can paste a token.
