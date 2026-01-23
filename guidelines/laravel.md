# Laravel Code Preferences

## Import Classes Instead of Using Long FQCNs Inline

Prefer importing classes at the top of the file with use rather than referencing long fully-qualified class names inline.

Why: Improves readability, keeps code concise, and still preserves refactor safety and LSP support.

```
// ✅ Good
use Illuminate\Bus\Batch;
use Illuminate\Support\Facades\App;

$this->assertInstanceOf(Batch::class, $batch);
App::make();

// ❌ Avoid
$this->assertInstanceOf(\Illuminate\Bus\Batch::class, $batch);
\App::make();
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
Auth::check()
Auth::user()
Gate::allows('premium')
Cache::get('key')
App::make()

// ❌ Avoid
auth()->check()
auth()->user()
app()

{{-- ✅ Blade --}}
@if(Auth::check())
{{ Auth::user()->name }}
@endif
```

Why: Class names are refactor-safe. `Auth::` provides better LSP autocomplete and jump-to-definition in Neovim.

### Practical Examples

- Prefer `$request->user()` when `Request $request` is injected
- Use `Auth::user()` as fallback when Request unavailable
- Never use `auth()->user()` global helper

If you are unable to use injected dependencies, please follow the rules regarding PHPDocs.

Use `@var` PHPDoc annotations when IDE or static analysis (Larastan) can't infer types, particularly for user variables or when working with mixed/uncertain types.


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


### Dependencies vs Data

  When creating Action classes, clearly separate dependencies from data:

  **Constructor (Dependencies):**
  - Services, helpers, clients, repositories
  - Things resolved from the container
  - Stateless collaborators the action needs to do its work

  **Execute Method (Data):**
  - Models, DTOs, arrays, primitives
  - The specific data being operated on
  - Things that change per invocation

  ```php
  // ✅ Correct
  class SendMassTextAction {
      public function __construct(
          protected SMSHelper $smsHelper  // Dependency - injected via DI
      ) {}

      public function execute(TextMessage $textMessage): Batch {  // Data - passed per call
          // ...
      }
  }

  // Usage
  $action = App::make(SendMassTextAction::class);
  $batch = $action->execute($textMessage);

  // ❌ Wrong - mixing data into constructor
  class SendMassTextAction {
      public function __construct(
          public TextMessage $textMessage,  // Data should NOT be here
          protected SMSHelper $smsHelper
      ) {}

      public function execute(): Batch { ... }
  }

  // Awkward usage
  $action = App::make(SendMassTextAction::class, ['textMessage' => $txt]);
  $batch = $action->execute();

  Rule of thumb: If you need to pass parameters to App::make(), you're probably putting data in the constructor that belongs in the method.
  ```

### Benefits

- Models stay thin (data access and relationships only)
- Logic is testable in isolation
- Single responsibility principle per class
- Clear dependency injection
- Easy to find and maintain business logic
