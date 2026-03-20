# Laravel Tyro Skill

Complete knowledge and operation guide for **Tyro** - the Laravel authentication, authorization, and role-based access control package.

## About This Skill

This skill provides AI agents with comprehensive knowledge of Tyro, enabling them to:
- Install and configure Tyro in Laravel projects
- Use all 40+ CLI commands
- Implement route protection with middleware
- Use Blade directives for permission checks
- Work with the REST API
- Troubleshoot common issues

## Skill Contents

| File | Description |
|------|-------------|
| `SKILL.md` | Main skill documentation (500+ lines) |
| `references/quick-commands.md` | CLI command quick reference |
| `references/code-examples.md` | Real-world code examples |
| `evals/evals.json` | Test cases for skill validation |

## What is Tyro?

Tyro is a comprehensive authentication and authorization package for Laravel 12 and 13 that provides:

- **User Authentication** - Integration with Laravel Sanctum
- **Role-Based Access Control** - Unlimited roles with slug-based system
- **Privilege Management** - Fine-grained permissions
- **User Suspension** - Freeze accounts with token revocation
- **40+ CLI Commands** - Complete management toolkit
- **7 Blade Directives** - Template permission checks
- **REST API** - Optional HTTP endpoints
- **Audit Trail** - Track all administrative actions

## Quick Start

```bash
composer require hasinhayder/tyro
php artisan tyro:sys-install
```

Default credentials after seeding:
- Email: `admin@tyro.project`
- Password: `tyro`

## Default Roles

| Slug | Name | Purpose |
|------|------|---------|
| `super-admin` | Super Admin | Full system access |
| `administrator` | Administrator | Administrative access |
| `editor` | Editor | Content management |
| `user` | User | Standard registered user |
| `customer` | Customer | Customer account |
| `all` | All | Universal access (wildcard) |

## Installation

This skill should be placed in your Claude skills directory:
```
~/.claude/skills/laravel-tyro/
```

## Version

- **Skill Version:** 1.0.0
- **Tyro Version Supported:** 1.5.0+
- **Laravel Version:** 12 and 13

## Links

- Tyro GitHub: https://github.com/hasinhayder/tyro
- Tyro Labs: https://tyrolabs.dev/
- Related Projects: tyro-login, tyro-dashboard
