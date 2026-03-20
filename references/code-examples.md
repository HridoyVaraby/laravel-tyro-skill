# Tyro Code Examples

## Checking Permissions in Controllers

```php
use Illuminate\Http\Request;

class ArticleController extends Controller
{
    public function publish(Request $request, Article $article)
    {
        // Check privilege
        if (!$request->user()->can('articles.publish')) {
            abort(403, 'You do not have permission to publish articles.');
        }

        $article->published_at = now();
        $article->save();

        return response()->json($article);
    }

    public function destroy(Request $request, Article $article)
    {
        // Check role
        if (!$request->user()->hasRole('admin')) {
            abort(403, 'Admins only.');
        }

        $article->delete();
        return response()->json(['message' => 'Deleted']);
    }

    public function bulkAction(Request $request)
    {
        // Check multiple roles (must have ALL)
        if (!$request->user()->hasRoles(['editor', 'publisher'])) {
            abort(403, 'Requires both editor and publisher roles.');
        }

        // Check multiple privileges (must have ALL)
        if (!$request->user()->hasPrivileges(['articles.edit', 'articles.publish'])) {
            abort(403, 'Missing required privileges.');
        }
    }
}
```

## Managing Roles Programmatically

```php
use HasinHayder\Tyro\Models\Role;
use App\Models\User;

class UserRoleService
{
    public function assignEditorRole(int $userId)
    {
        $user = User::find($userId);
        $editorRole = Role::where('slug', 'editor')->first();

        $user->assignRole($editorRole);
        // Cache is automatically cleared
    }

    public function removeRole(int $userId, string $roleSlug)
    {
        $user = User::find($userId);
        $role = Role::where('slug', $roleSlug)->first();

        $user->removeRole($role);
    }

    public function syncRoles(int $userId, array $roleSlugs)
    {
        $user = User::find($userId);
        $roleIds = Role::whereIn('slug', $roleSlugs)->pluck('id');

        $user->roles()->sync($roleIds);
        // Sync replaces all existing roles
    }

    public function getUserRoles(int $userId): array
    {
        $user = User::find($userId);

        // Get just slugs (cached, fast)
        return $user->tyroRoleSlugs();
        // ['admin', 'editor']

        // Or get full models
        return $user->roles()->select(['id', 'name', 'slug'])->get();
        // Collection of Role models
    }
}
```

## Managing Privileges on Roles

```php
use HasinHayder\Tyro\Models\Role;
use HasinHayder\Tyro\Models\Privilege;

class RolePrivilegeService
{
    public function grantArticlePermissions(string $roleSlug)
    {
        $role = Role::where('slug', $roleSlug)->first();

        $privileges = Privilege::whereIn('slug', [
            'articles.create',
            'articles.edit',
            'articles.publish',
        ])->get();

        $role->privileges()->attach($privileges);
    }

    public function revokePermission(string $roleSlug, string $privilegeSlug)
    {
        $role = Role::where('slug', $roleSlug)->first();
        $privilege = Privilege::where('slug', $privilegeSlug)->first();

        $role->privileges()->detach($privilege->id);
    }

    public function setRolePermissions(string $roleSlug, array $privilegeSlugs)
    {
        $role = Role::where('slug', $roleSlug)->first();
        $privilegeIds = Privilege::whereIn('slug', $privilegeSlugs)->pluck('id');

        $role->privileges()->sync($privilegeIds);
    }

    public function checkRolePermission(string $roleSlug, string $privilegeSlug): bool
    {
        $role = Role::where('slug', $roleSlug)->first();
        return $role->hasPrivilege($privilegeSlug);
    }
}
```

## User Suspension

```php
use App\Models\User;

class ModerationService
{
    public function suspendUser(int $userId, string $reason)
    {
        $user = User::find($userId);

        $user->suspend($reason);
        // This:
        // 1. Sets suspended_at = now()
        // 2. Sets suspension_reason = $reason
        // 3. Deletes all Sanctum tokens
        // 4. Clears cache
    }

    public function unsuspendUser(int $userId)
    {
        $user = User::find($userId);
        $user->unsuspend();
    }

    public function checkSuspension(User $user)
    {
        if ($user->isSuspended()) {
            return [
                'suspended' => true,
                'reason' => $user->getSuspensionReason(),
            ];
        }
        return ['suspended' => false];
    }
}
```

## Policies with Tyro

```php
use App\Models\Article;
use App\Models\User;

class ArticlePolicy
{
    public function create(User $user): bool
    {
        return $user->can('articles.create');
    }

    public function update(User $user, Article $article): bool
    {
        // Can edit any article
        if ($user->can('articles.edit.any')) {
            return true;
        }

        // Can edit own articles
        if ($user->can('articles.edit.own') && $article->user_id === $user->id) {
            return true;
        }

        return false;
    }

    public function publish(User $user): bool
    {
        return $user->can('articles.publish');
    }

    public function delete(User $user, Article $article): bool
    {
        return $user->hasRole('admin') || $user->can('articles.delete');
    }
}

// In controller
public function update(Article $article)
{
    $this->authorize('update', $article);
    // ...
}
```

## API Authentication

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;

class AuthController extends Controller
{
    public function login(Request $request)
    {
        $request->validate([
            'email' => 'required|email',
            'password' => 'required',
        ]);

        $user = User::where('email', $request->email)->first();

        if (!$user || !Hash::check($request->password, $user->password)) {
            return response()->json(['message' => 'Invalid credentials'], 401);
        }

        if ($user->isSuspended()) {
            return response()->json([
                'message' => 'Account suspended.',
                'reason' => $user->getSuspensionReason(),
            ], 403);
        }

        // Delete previous tokens if single-session enabled
        if (config('tyro.delete_previous_access_tokens_on_login')) {
            $user->tokens()->delete();
        }

        // Token includes roles and privileges as abilities
        $abilities = array_merge(
            $user->tyroRoleSlugs(),
            $user->tyroPrivilegeSlugs()
        );

        $token = $user->createToken('auth-token', $abilities)->plainTextToken;

        return response()->json([
            'token' => $token,
            'user' => $user->load('roles'),
        ]);
    }

    public function me(Request $request)
    {
        return response()->json([
            'user' => $request->user(),
            'roles' => $request->user()->tyroRoleSlugs(),
            'privileges' => $request->user()->tyroPrivilegeSlugs(),
        ]);
    }

    public function logout(Request $request)
    {
        $request->user()->currentAccessToken()->delete();
        return response()->json(['message' => 'Logged out']);
    }
}
```

## Middleware in Controllers

```php
use Illuminate\Support\Facades\Route;

// routes/api.php

// Public routes
Route::post('login', [AuthController::class, 'login']);
Route::post('register', [AuthController::class, 'register']);

// Authenticated routes
Route::middleware('auth:sanctum')->group(function () {
    Route::get('me', [AuthController::class, 'me']);
    Route::post('logout', [AuthController::class, 'logout']);
});

// Role-based routes
Route::middleware(['auth:sanctum', 'role:admin'])->group(function () {
    Route::apiResource('users', UserController::class);
    Route::apiResource('roles', RoleController::class);
});

// Privilege-based routes
Route::middleware(['auth:sanctum', 'privilege:articles.publish'])->group(function () {
    Route::post('articles/{article}/publish', [ArticleController::class, 'publish']);
});

// Combined
Route::middleware(['auth:sanctum', 'roles:editor,admin'])->group(function () {
    Route::apiResource('articles', ArticleController::class);
});
```

## Custom Queries

```php
// Get users with specific role
$editors = User::whereHas('roles', function ($query) {
    $query->where('slug', 'editor');
})->get();

// Get users with any of multiple roles
$managers = User::whereHas('roles', function ($query) {
    $query->whereIn('slug', ['editor', 'admin']);
})->get();

// Get role with all privileges
$role = Role::with('privileges')->where('slug', 'admin')->first();

// Count users per role
$roleCounts = Role::withCount('users')->get()->pluck('name', 'users_count');
```
