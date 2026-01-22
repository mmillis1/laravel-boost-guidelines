# Pest Testing Conventions

  ## Prefer Global Functions Over $this Methods
  When writing Pest tests, prefer using Pest's global helper functions with explicit imports instead of `$this->` methods. This aligns with Pest's functional style and improves code readability.

  **Good:**
  ```php
  use function Pest\Laravel\assertDatabaseHas;

  test('creates user', function () {
      assertDatabaseHas('users', ['email' => 'test@example.com']);
  });

  Avoid:
  test('creates user', function () {
      $this->assertDatabaseHas('users', ['email' => 'test@example.com']);
  });

  Single-Line Imports Only

  Always use single-line imports for functions. Never use grouped imports.

  Good:
  use function Pest\Laravel\assertDatabaseHas;
  use function Pest\Laravel\assertDatabaseMissing;
  use function Pest\Laravel\actingAs;

  Never:
  use function Pest\Laravel\{assertDatabaseHas, assertDatabaseMissing, actingAs};
