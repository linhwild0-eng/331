# NTTCLASS deployment

This project includes the web interface and its Express API. Deploy the whole
project as a Node.js service; deploying only `public/index.html` will not provide
shared login or database storage.

## Existing Vercel URL

The NTTCLASS Vercel deployment is currently serving only static HTML:
`/api/health`, `/api/login`, and `/api/data` return Vercel `404 NOT_FOUND`.
Therefore the login form cannot authenticate on that URL, regardless of the
username/password. This app uses Express, SQLite, and in-memory sessions, so it
must run as one persistent Node.js web service with a persistent SQLite disk.
Do not deploy this SQLite/session backend as a Vercel static site or serverless
function.

The included `render.yaml` is a ready-to-configure Render Blueprint for the
whole app. Deploy it as a Node web service with its persistent disk, set the
administrator password when Render prompts for `NTTCLASS_ADMIN_PASSWORD`, and
use the resulting Render URL as the website link. The Vercel URL does not
automatically forward API requests to that service.

## Requirements

- Node.js 20 or newer
- A hosting service that runs `npm start`
- Persistent disk/storage for SQLite
- One running service instance (sessions and write ordering are held in memory)

## Deploy

1. Upload/extract this project, then install dependencies with `npm ci`.
2. Configure the service start command as `npm start`. The service must use its
   assigned `PORT`; the app reads that environment variable automatically.
3. Configure `NTTCLASS_DB_PATH` to a file on the host's persistent disk, such as
   `/var/data/nttclass.db`. Ensure the service can write to that disk.
4. Before the first start on an empty database, set `NTTCLASS_ADMIN_USERNAME`,
   `NTTCLASS_ADMIN_PASSWORD`, and optionally `NTTCLASS_ADMIN_DISPLAY_NAME`.
   The password must be at least 8 characters. Do not publish `.env` or put
   credentials in source control.
5. Open the public HTTPS URL provided by the host and sign in with the configured
   administrator account.

If the administrator environment variables are omitted on a brand-new database,
the compatibility defaults are `linhAdmin` and `Linh2000`; set a strong password
before exposing the service to the internet. These variables seed an account
only when the database has no app data. They do not change passwords in an
existing database.

## Updating an existing deployment

- Keep the persistent SQLite database and its storage path. Do not overwrite or
  delete it while replacing application files.
- Back up the database before deploying an update. If moving hosts, transfer the
  database using the hosting provider's private storage/backup tools, not a
  public website or source archive.
- `nttclass.db`, `.env`, and `node_modules` are intentionally not part of the
  source deployment archive. The database contains private school records;
  configure its persistent path separately on the hosting service.
- After deploy, verify `/api/health`, sign in, and confirm existing classes and
  students are still present.

## Local run

```powershell
npm ci
$env:PORT = "3000"
npm start
```

The database defaults to `nttclass.db` next to `server.js`. Set
`NTTCLASS_DB_PATH` to another writable location to use a different database.
