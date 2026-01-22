## DDEV Local Development Environment

### Overview

- This project uses DDEV for local development.
- The application is accessible at `https://<project-name>.ddev.site` when running (use `ddev describe` to see the exact URL).
- Use `ddev describe` to see project URLs, database credentials, and container status.

### Command Execution Methods

**Use command shortcuts when possible:**

- `ddev artisan <command>` - For all Laravel Artisan commands
- `ddev composer <command>` - For all Composer commands
- `ddev npm <command>` - For all NPM commands
- `ddev mysql` - For interactive MySQL access

**Use `ddev exec <command>` when:**

- Running a single command not covered by shortcuts
- Examples: `ddev exec vendor/bin/pint`, `ddev exec php -v`, `ddev exec ls -la storage/logs`
- Commands execute in the container's docroot (`/var/www/html`) using Bash

### MCP Tools

- Laravel Boost MCP tools (tinker, database-query, list-artisan-commands, etc.) automatically run inside the DDEV
  container via the configured MCP bridge.
- No `ddev` prefix is needed when using MCP tools - the connection is already configured.

### File Paths: Container vs Host

- Files in your project directory are automatically synced to `/var/www/html` in the container.
- When inside the container (via `ddev ssh`), you're already at the project root.
- **Do not use absolute host paths in DDEV commands** - they won't exist in the container.
- Read Laravel logs using the Read tool at `storage/logs/laravel.log` (relative to project root).

### Laravel-Specific Commands

**Running tests:**

- All tests: `ddev artisan test`
- Specific file: `ddev artisan test tests/Feature/ExampleTest.php`
- Filter: `ddev artisan test --filter=testName`

### Queue Workers & Scheduled Tasks

### Frontend Development

- Developer is running ddev npm run dev. No need to rebuild or manually run vite.

**Running Vite:**

- Development: `ddev npm run dev` (starts Vite dev server with HMR)
- Build: `ddev npm run build`
- If assets don't update, check if Vite dev server is running

**Vite configuration:**

- Configured to work seamlessly with DDEV
- Hot module replacement (HMR) works across host/container boundary

### Debugging & Troubleshooting

**Check container info:**

- `ddev describe` - Project info, URLs, credentials
- `ddev exec env` - Environment variables

### Common DDEV Gotchas

**Always prefix PHP/Laravel commands with `ddev`:**

- ❌ `php artisan migrate` (fails - runs on host, no PHP)
- ✅ `ddev artisan migrate` (works - runs in container)
- ❌ `vendor/bin/pint` (fails - no vendor directory on host)
- ✅ `ddev exec vendor/bin/pint` (works - runs in container)
- ❌ `composer install` (fails - runs on host)
- ✅ `ddev composer install` (works - runs in container)

**Port conflicts:**

- DDEV uses ports 80 and 443 by default
- If conflicts occur, check for other Docker environments or local servers

## Laravel Pint Code Formatter

- You must run `ddev exec vendor/bin/pint --dirty` before finalizing changes to ensure your code matches the project's
  expected style.
- Do not run `vendor/bin/pint --test`, simply run `ddev exec vendor/bin/pint` to fix any formatting issues.

## User Login

If you ever need to login to the application to view a page in chrome-dev-tools or something. Please use the credentials in Database\Seeders\DevSeeder.php (email: mmillis1@gmail.com, password: password).
