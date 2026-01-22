# Laravel Code Preferences

## Command References

**Always use class names instead of strings.**

```php
// ✅ Good
use App\Console\Commands\SyncUsers;
Schedule::command(SyncUsers::class)->daily();
artisan(SyncUsers::class);

// ❌ Avoid
Schedule::command('users:sync')->daily();
artisan('users:sync');
```

## Prefer Injected Dependencies Over Facades

Use injected dependencies when available instead of pulling data through facades or global helpers. This provides better type inference and follows dependency injection best practices.

```php
// ✅ Good - Using injected Request
public function __invoke(Request $request) {
    $user = $request->user(); // Returns App\Models\User|null
    $view = $request->query('view', 'card');
}

// ✅ Good - Fallback to global helpers when not injected
public function __invoke() {
    $view = request()->query('view', 'card'); // No Request injected
}

// ❌ Avoid - Using global helper when Request is available
public function __invoke(Request $request) {
    $view = request()->query('view', 'card'); // Should use $request
}
```

## Facades Over Helpers

When using facades/global methods (when injected dependencies aren't available), use static facades with leading backslash for better IDE/LSP support.

```php
// ✅ Good - Auth
\Auth::check()
\Auth::user()
\Gate::allows('premium')

// ✅ Good - Other facades
\View::make('welcome')
\Config::get('app.name')
\Cache::get('key')

// ❌ Avoid
auth()->check()
auth()->user()
view()->make('welcome')
config()->get('app.name')

{{-- ✅ Blade --}}
@if(\Auth::check())
{{ \Auth::user()->name }}
@endif
```

Why: Class names are refactor-safe. `\Auth::` provides better LSP autocomplete and jump-to-definition in Neovim.

## Practical Examples

- Prefer `$request->user()` when `Request $request` is injected
- Use `\Auth::user()` as fallback when Request unavailable
- Never use `auth()->user()` global helper

If you are unable to use injected dependencies, please follow the rules regarding PHPDocs.

Use `@var` PHPDoc annotations when IDE or static analysis (Larastan) can't infer types, particularly for user variables or when working with mixed/uncertain types.

```php
// ✅ Good - When user is guaranteed to exist
/** @var \App\Models\User $user */
$user = \Auth::user();

// ✅ Good - When user might be null
/** @var \App\Models\User|null $user */
$user = \Auth::user();

// ✅ Good - When working with ambiguous relationships
/** @var \App\Models\Vehicle|null $vehicle */
$vehicle = $timeslip->vehicle;
```

When to use `@var` annotations:

- After retrieving user via `\Auth::user()` or similar methods that return unions
- When accessing relationships that might return `Model` instead of specific model types
- When working with mixed types from dynamic properties/methods
- Anywhere Larastan complains about undefined properties or type mismatches

## Single Action Classes

Prefer single action classes over business logic methods on models. Encapsulate business logic in dedicated action classes.

### Structure Requirements

- Exactly ONE public `execute()` method - this is the only external interface
- Can have multiple private/protected methods for internal implementation
- Private methods must be **internal to the class** (never called externally)
- All action classes must be **instances**, never static methods
- Names must **ALWAYS start with a verb** (e.g., `Get`, `Calculate`, `Process`)

### Placement and Organization

- Store in `app/Actions/` directory
- Create subdirectories inside `Actions/` for organizational purposes (e.g., `app/Actions/Dashboard/`, `app/Actions/Timeslip/`)
- Organize by domain or feature area

### Usage Patterns

```php
// ✅ Preferred - Dependency Injection
public function __invoke(
    GetPrimaryVehicle $getPrimaryVehicle,
    GetVehicleStats $getVehicleStats
) {
    $primaryVehicle = $getPrimaryVehicle->execute($user);
    $stats = $getVehicleStats->execute($vehicle);
}

// ✅ Acceptable - Manual Instantiation (when DI not available)
$action = new GetPrimaryVehicle();
$result = $action->execute($user);
```

### Naming Conventions

Use descriptive names following Verb-Noun or Verb-Adjective patterns:

- `GetPrimaryVehicle` - Retrieve specific data
- `GetPerformanceTrends` - Compute trends from data
- `GetVehicleStats` - Calculate statistics
- `GetDefaultTimeslipName` - Generate formatted output

### Benefits

- Models stay thin (data access and relationships only)
- Logic is testable in isolation
- Single responsibility principle per class
- Clear dependency injection
- Easy to find and maintain business logic
