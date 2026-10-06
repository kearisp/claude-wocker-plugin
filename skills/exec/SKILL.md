---
name: exec
description: Run PHP, Composer, artisan and npm commands in a project that uses `ws` (wocker), and start/set up the project. Use whenever you need to run `php`, `composer`, `php artisan`, `npm`, tests or migrations — the host has no PHP/Composer, so they must go through `ws exec`.
---

## Running the project (wocker)

There is no PHP/Composer on the host. All PHP (and npm) commands **must** be run through `ws exec` from the project root, never directly:

```bash
ws exec php artisan migrate
ws exec composer install
ws exec php artisan test
```

All PHP and NPM commands **must** be run through the `ws` wrapper from the project root:

### Setup (one-time)

```bash
npm install -g @wocker/ws
ws plugin:install mariadb
ws mariadb:create default
ws init
```

### Database

```bash
ws mariadb:start
```

### Start the project

```bash
ws start
```

The app runs on the host configured via `VIRTUAL_HOST` in `wocker.config.json`.
