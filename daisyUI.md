### Tailwind/daisyUI vs Flux

This project does NOT use ANY flux components. If there are any present, they are legacy and will be
removed in due course. ONLY use Tailwind CSS and daisyUI components for styling and UI elements.

### Tailwind / daisyUI class generation (NO dynamic string concat)

Do **not** build Tailwind or daisyUI classes via raw string concatenation
(e.g. `class="btn btn-{{ $status }}"` or `"bg-{$color}"`). This breaks
Tailwind's content scanning and forces us to maintain ugly manual safelists
or fake CSS rules like:

```css
@layer components {
    .btn-primary,
    .btn-secondary,
    .bg-primary,
    .bg-secondary {
        /* force Tailwind to keep these */
    }
}
```

This is an **anti-pattern**.

**Instead:**

1. Prefer **enumerated maps** in PHP / Blade where the possible classes are
   explicit literals in the file:

    ```php
    @php
        $statusClass = match ($status) {
            'success' => 'btn-success',
            'error' => 'btn-error',
            'warning' => 'btn-warning',
            default => 'btn-neutral',
        };
    @endphp

    <button class="btn {{ $statusClass }}">
    ```

2. If you truly need dynamic variants (e.g. `bg-{color}`, `btn-{color}`,
   etc.), make sure the set of possible values is:

- finite and known, and
- covered by Tailwind’s `safelist` in `tailwind.config.js` using
  explicit entries or `pattern` regexes.

```js
// tailwind.config.js
module.exports = {
    safelist: [
        {
            pattern:
                /^(bg|text|border|btn|alert)-(primary|secondary|accent|info|success|warning|error|neutral)$/,
        },
        {
            pattern:
                /^hover:(bg|text|border)-(primary|secondary|accent|info|success|warning|error|neutral)$/,
        },
    ],
};
```

Rule of thumb: **classes should be discoverable as plain strings in the
source or in Tailwind’s config, not assembled via arbitrary string
concatenation at render time.**

Don't ever use `btn-ghost`. I hate how it looks. If you want to use this button style, consider just using a gray or secondary button color.

