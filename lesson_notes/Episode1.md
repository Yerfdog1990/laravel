# Ep 01: Hello, Laravel

## Overview

Episode 1 is the starting point of the entire series. It covers who this course is for, a brief introduction to the instructor's background with Laravel, and — most importantly — how to get a working Laravel development environment up and running on your machine with zero friction. By the end of this episode you have a fully functional Laravel application running in your browser without having manually installed PHP, MySQL, or a web server.

---

## 1. About the Instructor

Jeffrey Way is the creator of Laracasts and one of the longest-standing Laravel educators in the community. Some highlights he shares:

- Has been working with Laravel for **over a decade**, from its earliest days before it was widely known.
- Was a **presenter at the very first Laracon** — the Laravel-specific conference — when it had only around 60 attendees. Today Laracon events are held worldwide with thousands of attendees.
- Notes a common mispronunciation: many people say *"LEL"* instead of the correct *"Lara-vel"*.

The point of sharing this background is not self-promotion, but to establish that he is a reliable guide for someone just beginning their Laravel journey.

---

## 2. What is Laravel?

Laravel is a **PHP web application framework** designed to make common web development tasks — routing, authentication, database access, templating, and more — faster and more enjoyable. It follows the MVC (Model–View–Controller) architectural pattern and has become one of the most popular PHP frameworks in the world.

### Why Laravel?

| Feature | Description |
|---|---|
| Expressive syntax | Clean, readable PHP that feels natural to write |
| Batteries included | Authentication, queues, mail, storage, and more out of the box |
| Blade templating | A lightweight yet powerful PHP templating engine |
| Eloquent ORM | An elegant ActiveRecord implementation for database access |
| Ecosystem | Tools like Herd, Valet, Sail, Forge, Vapor, and Envoyer all built around Laravel |
| Community | One of the largest and most active PHP communities worldwide |

---

## 3. Setting Up a Development Environment

### The Old Way (Still Valid)

Historically, setting up a PHP development environment involved choosing between several different tools, which could cause decision paralysis for newcomers:

| Tool | Platform | Notes |
|---|---|---|
| **XAMPP** | Cross-platform | Apache + MySQL + PHP bundled together |
| **MAMP** | Mac / Windows | Similar to XAMPP, popular on Mac |
| **WAMP** | Windows | Windows-specific Apache/MySQL/PHP stack |
| **Laragon** | Windows | Modern, lightweight alternative |
| **Homebrew** | Mac | Manual installation: `brew install php`, `brew install mysql`, etc. |

All of these are still valid and actively maintained. However, they require more setup steps and configuration knowledge than is ideal for a beginner.

### The Recommended Way — Laravel Herd

For this series, the recommended tool is **Laravel Herd** — a one-click desktop application that installs and manages everything you need to build PHP and Laravel applications locally.

**Download:** [herd.laravel.com](https://herd.laravel.com)

- Available for **macOS** and **Windows**
- Linux users typically already have the tools to set up manually
- There is a **free version** that covers everything needed for this course and beyond
- A paid **Pro version** exists for extra features but is entirely optional

### What Herd Installs

When you first open Herd, it automatically downloads and configures:

| Binary | Purpose |
|---|---|
| **PHP** (8.3 at time of recording) | The language runtime |
| **laravel** | The Laravel installer CLI tool |
| **composer** | PHP's dependency manager (covered in more detail later) |

### Herd's Magic: Automatic `.test` Domains

One of Herd's most impressive features is automatic local domain registration. Any PHP project placed inside the Herd-monitored folder is **instantly accessible in your browser** at `<folder-name>.test` — no virtual host configuration, no editing of `/etc/hosts`, no web server setup required.

```
~/Herd/example/   →   http://example.test
~/Herd/myapp/     →   http://myapp.test
```

You can also register additional directories (e.g. `~/code`) from Herd's General settings tab, and all projects in those folders will be treated the same way.

### Managing PHP Versions

Herd lets you install and switch between **multiple PHP versions** simultaneously. This is particularly useful when you have older projects that depend on PHP 7.x or 8.1 while newer projects use 8.3. You can switch versions with a single click from the menu bar icon — no terminal commands required.

---

## 4. Creating Your First Laravel Project

With Herd installed, creating a new Laravel project is a single command. Open your terminal, navigate to your Herd directory, and run:

```bash
cd ~/Herd
laravel new example
```

The `laravel new` command will walk you through a short interactive setup:

### Installer Prompts

| Prompt | Recommended Choice for Beginners | Notes |
|---|---|---|
| **Install a starter kit?** | No / None | Starter kits add auth scaffolding; we start from scratch here |
| **Test framework** | Default (Pest or PHPUnit) | Accept the default |
| **Initialize a Git repo?** | No | Just playing around for now |
| **Which database?** | SQLite | File-based, zero config — perfect for learning |

### What Happens During Installation

The installer uses **Composer** (PHP's package manager) under the hood to pull in all of Laravel's dependencies. Even though you didn't need to know anything about Composer for this step, it is working behind the scenes — and you'll learn more about it as the series progresses.

Once complete, a new folder called `example` is created inside `~/Herd/` with the full Laravel framework structure inside.

---

## 5. Viewing Your Application

Once the project is created, open your browser and visit:

```
http://example.test
```

You will see the **Laravel 11 default landing page** — a clean, styled welcome screen confirming that your application is configured correctly and ready for development.

No server restart, no port numbers, no configuration. It just works.

---

## 6. The Laravel Project Structure (First Look)

When you `cd` into the new project and list its files, you'll see the full Laravel framework directory layout:

```
example/
├── app/            # Core application code (Models, Controllers, etc.)
├── bootstrap/      # Framework bootstrapping files
├── config/         # All configuration files
├── database/       # Migrations, seeders, and factories
├── public/         # Web root — index.php lives here
├── resources/      # Views (Blade templates), CSS, JS
├── routes/         # Route definitions (web.php, api.php, etc.)
├── storage/        # Logs, compiled views, file uploads
├── tests/          # Automated tests
├── vendor/         # Composer dependencies (don't edit these)
├── .env            # Environment variables (DB, mail, etc.)
├── artisan         # Laravel's CLI tool
├── composer.json   # PHP dependency manifest
└── package.json    # Node/JS dependency manifest
```

Don't feel overwhelmed — you won't need to know all of these immediately. The series introduces each part in context as it becomes relevant.

---

## 7. Alternative Setup Options

If you choose not to use Herd, these are the main alternatives mentioned, all of which are valid:

| Tool | Best For | Link |
|---|---|---|
| **Laragon** | Windows users wanting a lightweight local stack | laragon.net |
| **XAMPP** | Cross-platform, very established | apachefriends.org |
| **Docker + Laravel Sail** | Developers comfortable with containers | Built into Laravel |
| **Homebrew (Mac)** | Developers who prefer manual control | brew.sh |

Each has different installation steps. If you get stuck, asking in the comments or Laravel community forums is encouraged.

---

## 8. What's Coming Next

Episode 1 ends with a teaser for Episode 2:

> *"In the next video you are going to create your first route."*

Routes are how Laravel maps a URL (like `/about`) to a specific response or page. This is one of the most fundamental concepts in any web framework and will be the foundation for everything that follows.

---

## Key Takeaways

| Concept | Summary |
|---|---|
| Laravel | A PHP web framework for building modern web applications |
| Laravel Herd | One-click desktop tool to run Laravel locally — highly recommended for beginners |
| `laravel new <name>` | Creates a new Laravel project with an interactive setup wizard |
| `.test` domains | Herd automatically serves any project at `<folder-name>.test` in your browser |
| SQLite | The simplest database choice for local development — file-based, zero config |
| Composer | PHP's package manager, used behind the scenes to install Laravel's dependencies |
| `artisan` | Laravel's built-in CLI tool — you'll use this constantly throughout the series |

---
