# Tyro CLI Quick Reference

## Installation Commands
```bash
composer require hasinhayder/tyro
php artisan tyro:sys-install
```

## Most Used Commands

### User Management
```bash
php artisan tyro:user-create                              # Create new user
php artisan tyro:user-list                                # List all users
php artisan tyro:user-roles --user=1                      # Show user's roles
php artisan tyro:user-suspend --user=email@example.com    # Suspend user
php artisan tyro:user-unsuspend --user=email@example.com  # Unsuspend
```

### Role Management
```bash
php artisan tyro:role-list                                # List roles
php artisan tyro:role-create --name="Name" --slug="slug"  # Create role
php artisan tyro:role-assign --user=1 --role="editor"     # Assign to user
```

### Privilege Management
```bash
php artisan tyro:privilege-list                           # List privileges
php artisan tyro:privilege-create --name="Name" --slug="slug"
php artisan tyro:privilege-attach --role="editor" --privilege="articles.edit"
```

### Authentication
```bash
php artisan tyro:auth-login --email=admin@tyro.project    # Get token
php artisan tyro:auth-logout-all --user=1                 # Revoke all tokens
```

## Full Command List by Category

### System (7 commands)
- `tyro:sys-install` - Full installation
- `tyro:user-prepare` - Add trait to User model
- `tyro:publish-config` - Publish config
- `tyro:publish-migrations` - Publish migrations
- `tyro:sys-version` - Show version
- `tyro:sys-about` - Show info
- `tyro:sys-postman` - Get Postman collection

### User Management (8 commands)
- `tyro:user-create` - Create user
- `tyro:user-update` - Update user
- `tyro:user-delete` - Delete user
- `tyro:user-list` - List users
- `tyro:user-list-with-roles` - List with roles
- `tyro:user-suspend` - Suspend user
- `tyro:user-unsuspend` - Unsuspend
- `tyro:user-suspended` - List suspended

### Role Management (8 commands)
- `tyro:role-list` - List roles
- `tyro:role-list-with-privileges` - List with privileges
- `tyro:role-create` - Create role
- `tyro:role-update` - Update role
- `tyro:role-delete` - Delete role
- `tyro:role-assign` - Assign to user
- `tyro:role-remove` - Remove from user
- `tyro:role-users` - List users with role

### Privilege Management (7 commands)
- `tyro:privilege-list` - List privileges
- `tyro:privilege-create` - Create privilege
- `tyro:privilege-update` - Update privilege
- `tyro:privilege-delete` - Delete privilege
- `tyro:privilege-attach` - Attach to role
- `tyro:privilege-detach` - Detach from role
- `tyro:user-privileges` - Show user's privileges

### Authentication (6 commands)
- `tyro:auth-login` - Mint token
- `tyro:user-token` - Create token
- `tyro:auth-logout` - Revoke token
- `tyro:auth-logout-all` - Revoke all user tokens
- `tyro:auth-logout-all-users` - Emergency revoke ALL
- `tyro:auth-me` - Inspect token

### Audit (2 commands)
- `tyro:audit-list` - List logs
- `tyro:audit-purge` - Purge old logs

### Seeding (5 commands)
- `tyro:seed-all` - Seed everything
- `tyro:seed-roles` - Seed roles
- `tyro:seed-privileges` - Seed privileges
- `tyro:role-purge` - Delete all roles
- `tyro:privilege-purge` - Delete all privileges

## Common Flags

All commands support:
- `--force` - Skip confirmation prompts
- `--user=ID|email` - Specify user by ID or email
- `--role=slug|ID` - Specify role by slug or ID
- `--privilege=slug|ID` - Specify privilege by slug or ID
