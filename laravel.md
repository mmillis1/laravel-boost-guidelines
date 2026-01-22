# Laravel Code Preferences

## Import Classes Instead of Using Long FQCNs Inline

Prefer importing classes at the top of the file with use rather than referencing long fully-qualified class names inline.

Why: Improves readability, keeps code concise, and still preserves refactor safety and LSP support.

```
// ✅ Good
use Illuminate\Bus\Batch;

$this->assertInstanceOf(Batch::class, $batch);

// ❌ Avoid
$this->assertInstanceOf(\Illuminate\Bus\Batch::class, $batch);
```

## Class Names Over Strings (Everywhere)

**Always prefer class references (::class) over string literals anywhere they are supported.**

Why: Class references are refactor-safe, discoverable by IDE/LSP tooling, and reduce runtime errors caused by typos.

```
// ✅ Good
use App\Console\Commands\SyncUsers;
use App\Models\User;

Schedule::command(SyncUsers::class)->daily();
artisan(SyncUsers::class);

$this->assertInstanceOf(User::class, $user);

// ❌ Avoid
Schedule::command('users:sync')->daily();
artisan('users:sync');

$this->assertInstanceOf('App\Models\User', $user);
```

This applies to:

- Console commands
- Jobs
- Events / listeners
- Policies
- Model references
- Tests
- Container bindings
- Anywhere Laravel accepts a class reference

## Facades Over Helpers

Use static facades with leading backslash for better IDE/LSP support (Neovim + Intelephense).

```php
// ✅ Good
\Auth::check()
\Auth::user()
\Gate::allows('premium')
\Cache::get('key')

// ❌ Avoid
auth()->check()
auth()->user()

{{-- ✅ Blade --}}
@if(\Auth::check())
{{ \Auth::user()->name }}
@endif
```

Why: Class names are refactor-safe. `\Auth::` provides better LSP autocomplete and jump-to-definition in Neovim.
