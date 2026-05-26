# Ep 03: Create a Layout File Using Blade Components

## Overview

Episode 3 tackles one of the first real-world problems every web developer encounters: **code duplication across pages**. When each page has its own copy of the navigation, head tags, and page structure, updating anything means editing every single file. The solution is a **layout file** — a single shared template that wraps all your pages. Laravel implements this elegantly through **Blade components**. By the end of this episode you have a clean three-page app where all shared markup lives in one place.

---

## 1. Where We Left Off

At the end of Episode 2 (and after completing the homework), the project has three routes and three views:

```php
// routes/web.php
Route::get('/', function () {
    return view('home');
});

Route::get('/about', function () {
    return view('about');
});

Route::get('/contact', function () {
    return view('contact');
});
```

> **Note:** The default `welcome` view that ships with Laravel has been renamed to `home` to better reflect its purpose. Remember to update both the file name in `resources/views/` and the `view()` call in the route.

Each view currently contains its own full HTML document:

```html
<!-- resources/views/home.php -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>My Site</title>
</head>
<body>
    <h1>Hello from the homepage</h1>
</body>
</html>
```

The about and contact views are identical in structure, differing only in the `<h1>` text.

---

## 2. The Problem: Adding Navigation Reveals the Duplication

Adding a simple navigation bar exposes the core problem immediately. If you add nav links to the homepage:

```html
<body>
    <nav>
        <a href="/">Home</a>
        <a href="/about">About</a>
        <a href="/contact">Contact</a>
    </nav>
    <h1>Hello from the homepage</h1>
</body>
```

It works on the homepage — but the moment you click to `/about`, the navigation disappears. The about page has no nav. The naive fix is to paste the nav block into every view:

```html
<!-- about.php — WRONG approach -->
<body>
    <nav>
        <a href="/">Home</a>
        <a href="/about">About</a>
        <a href="/contact">Contact</a>
    </nav>
    <h1>Hello from the about page</h1>
</body>
```

This works, but it **does not scale**. With three pages it's annoying. With thirty pages it's a maintenance nightmare — changing one link means editing every single file. This is a classic violation of the **DRY principle** (Don't Repeat Yourself).

---

## 3. Introducing Blade Templates

Before building the layout, rename all view files from `.php` to `.blade.php`:

```
resources/views/
├── home.blade.php       (was home.php)
├── about.blade.php      (was about.php)
└── contact.blade.php    (was contact.php)
```

From the browser's perspective, nothing changes — the pages still render identically. But now you have access to the full power of **Blade**, Laravel's templating engine.

### What is Blade?

Blade is a **thin layer on top of PHP** that compiles down to plain PHP before being executed. It provides:

| Feature | Description |
|---|---|
| `{{ }}` syntax | Shorthand for PHP `echo` with automatic HTML escaping |
| `{!! !!}` syntax | Unescaped output (use carefully) |
| Directives (`@if`, `@foreach`, etc.) | Clean shorthand for common PHP control structures |
| Components | Reusable template files referenced as custom HTML tags |
| Slots | Placeholders inside components for injecting dynamic content |
| Layout inheritance | Share structure across multiple pages |

Blade files are cached as compiled PHP, so there is **no performance cost** to using Blade over plain PHP.

---

## 4. What is a Component?

A **component** is any reusable block of markup that can be referenced in multiple places across your application. Components live in a special directory:

```
resources/views/components/
```

> The directory name `components` is important — Laravel watches this specific folder and automatically treats any `.blade.php` file inside it as a component.

Examples of things that make good components:

- Layout / master template
- Navigation bar
- Navigation link
- Card
- Alert / flash message
- Avatar
- Dropdown menu
- Modal dialog

---

## 5. Creating the Layout Component

Create a new directory and file:

```
resources/views/components/layout.blade.php
```

Move all the shared boilerplate HTML into this file — the `<!DOCTYPE>`, `<html>`, `<head>`, navigation, and the outer `<body>` structure. Leave a **slot** placeholder where the unique page content will be injected:

```html
<!-- resources/views/components/layout.blade.php -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Site</title>
</head>
<body>
    <nav>
        <a href="/">Home</a>
        <a href="/about">About</a>
        <a href="/contact">Contact</a>
    </nav>

    <?php echo $slot; ?>
</body>
</html>
```

The `$slot` variable is **automatically available** inside every Blade component. It holds whatever content is placed between the component's opening and closing tags when the component is used.

---

## 6. Using the Layout Component in Views

Components are referenced using a special `<x->` prefix tag. The name after `x-` matches the filename of the component (without the `.blade.php` extension):

| Component File | Tag to Use |
|---|---|
| `components/layout.blade.php` | `<x-layout>` |
| `components/nav-link.blade.php` | `<x-nav-link>` |
| `components/alert.blade.php` | `<x-alert>` |
| `components/card.blade.php` | `<x-card>` |

The `x-` prefix ensures component tags never clash with native HTML tags.

### Updated Page Views

```html
<!-- resources/views/home.blade.php -->
<x-layout>
    <h1>Hello from the homepage</h1>
</x-layout>
```

```html
<!-- resources/views/about.blade.php -->
<x-layout>
    <h1>Hello from the about page</h1>
</x-layout>
```

```html
<!-- resources/views/contact.blade.php -->
<x-layout>
    <h1>Hello from the contact page</h1>
</x-layout>
```

Everything between `<x-layout>` and `</x-layout>` is captured as `$slot` and injected into the layout where `<?php echo $slot; ?>` appears.

---

## 7. The `{{ $slot }}` Blade Syntax

The long-form PHP echo can be replaced with Blade's double curly brace syntax:

```html
<!-- Long form (plain PHP) -->
<?php echo $slot; ?>

<!-- Blade shorthand (equivalent) -->
{{ $slot }}
```

Both produce identical output. The `{{ }}` syntax is:
- Shorter and more readable
- The standard convention in all Laravel projects
- **Automatically HTML-escaped** to protect against XSS attacks

> **Mental model:** Whenever you see `{{ $variable }}` in a Blade file, read it as "PHP echo this variable, safely."

The Blade file is compiled to a cached PHP file behind the scenes — you never have to worry about performance.

---

## 8. How the Slot System Works

The flow when Laravel renders `home.blade.php`:

```
home.blade.php                    layout.blade.php
──────────────                    ────────────────
<x-layout>               ──►      <!DOCTYPE html>
    <h1>Hello from                <html>
        the homepage</h1>         <head>...</head>
</x-layout>                       <body>
                                    <nav>...</nav>
                          ◄──       {{ $slot }}
                                    ↑
                                    <h1>Hello from the homepage</h1>
                                  </body>
                                  </html>
```

1. Laravel sees `<x-layout>` and loads `components/layout.blade.php`
2. Everything between `<x-layout>` and `</x-layout>` becomes `$slot`
3. The layout renders with `$slot` injected in place of `{{ $slot }}`
4. The final merged HTML is sent to the browser

---

## 9. The Benefit: Single Point of Change

Now that navigation lives in one file, adding or changing a link requires editing only `layout.blade.php`. All pages immediately reflect the change.

Before (duplication):
```
home.php     → contains nav
about.php    → contains nav (copy)
contact.php  → contains nav (copy)
             → adding a 4th link means editing 3 files
```

After (layout component):
```
layout.blade.php  → contains nav (single source of truth)
home.blade.php    → <x-layout> + unique content only
about.blade.php   → <x-layout> + unique content only
contact.blade.php → <x-layout> + unique content only
                  → adding a 4th link means editing 1 file
```

---

## 10. Final File Structure

```
resources/views/
├── components/
│   └── layout.blade.php    ← shared layout (nav, head, body wrapper)
├── home.blade.php           ← homepage content only
├── about.blade.php          ← about content only
└── contact.blade.php        ← contact content only
```

### Complete `layout.blade.php`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Site</title>
</head>
<body>
    <nav>
        <a href="/">Home</a>
        <a href="/about">About</a>
        <a href="/contact">Contact</a>
    </nav>

    {{ $slot }}
</body>
</html>
```

### Complete `home.blade.php`

```html
<x-layout>
    <h1>Hello from the homepage</h1>
</x-layout>
```

---

## 11. Homework — Extract Nav Links into Their Own Component

The homework for Episode 3 reinforces the component and slot concepts by going one level deeper.

**Task:** Create a new component called `nav-link` and extract the `<a>` tags from the layout into it. The label text (e.g. "Home", "About") must be dynamic using a slot.

### Step 1 — Create the component file

```
resources/views/components/nav-link.blade.php
```

```html
<!-- resources/views/components/nav-link.blade.php -->
<a href="#">{{ $slot }}</a>
```

> The `href` is hardcoded for now — making it dynamic with props is covered in Episode 4.

### Step 2 — Use it in the layout

```html
<!-- resources/views/components/layout.blade.php -->
<nav>
    <x-nav-link>Home</x-nav-link>
    <x-nav-link>About</x-nav-link>
    <x-nav-link>Contact</x-nav-link>
</nav>
```

The text between the tags ("Home", "About", "Contact") is passed in as `$slot` and rendered inside the `<a>` tag. The links won't navigate anywhere useful yet — that's addressed in the next episode when you learn about **component attributes and props**.

---

## Key Takeaways

| Concept | Summary |
|---|---|
| Layout file | A single shared template containing all common page structure (head, nav, footer) |
| `.blade.php` extension | Required to use Blade features — rename views from `.php` to `.blade.php` |
| `resources/views/components/` | Special directory — any `.blade.php` file here is automatically a component |
| `<x-componentname>` | How you reference a component in a Blade template |
| `$slot` | Auto-available variable inside components; holds the content placed between the component tags |
| `{{ $slot }}` | Blade shorthand for `<?php echo $slot; ?>` — also HTML-escapes the output |
| DRY principle | Don't Repeat Yourself — shared markup belongs in one file, not copied across many |
| Blade compilation | Blade templates are compiled to plain PHP and cached — no performance overhead |

---
