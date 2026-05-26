# Ep 05 - Style the Currently Active Navigation Link

## Overview

In a typical multi-page Laravel application, the navigation bar needs to visually indicate which page the user is currently on. Without dynamic styling, the active link style ends up hardcoded — always highlighting the same item regardless of the current URL.

This guide covers how to detect the current route/URL, apply conditional Tailwind CSS classes, and ultimately extract the logic into a clean, reusable Blade component. The demo project uses the **Tailwind CSS Stacked Layout** as its shell, explained in full below.

---

## 0. The Tailwind CSS Stacked Layout

### What Is a Stacked Layout?

A **Stacked Layout** (also called an Application Shell) is a full-width page structure where the navigation bar sits horizontally at the top of the screen, and the main content area fills the space below it. This is one of the most common patterns in web applications — used by tools like GitHub, Vercel, and Laravel Forge.

The Tailwind CSS team provides production-ready stacked layout templates as part of [Tailwind Plus UI Blocks](https://tailwindcss.com/plus/ui-blocks/application-ui/application-shells/stacked). The demo in this series uses the **"With lighter page header"** dark variant.

### Visual Structure

```
┌─────────────────────────────────────────────────────┐
│  Logo   Home   About   Contact          🔔  Avatar  │  ← <nav> (bg-gray-800)
├─────────────────────────────────────────────────────┤
│  Page Title                                         │  ← <header> (bg-white / shadow)
├─────────────────────────────────────────────────────┤
│                                                     │
│  Main content area ({{ $slot }})                    │  ← <main>
│                                                     │
└─────────────────────────────────────────────────────┘
```

### Key Structural Elements

| Element | Tailwind Classes | Purpose |
|---------|-----------------|---------|
| `<html>` | `h-full bg-gray-100` | Full viewport height, light page background |
| `<body>` | `h-full` | Inherits full height from html |
| Outer wrapper `<div>` | `min-h-full` | Ensures the layout always fills the screen |
| `<nav>` | `bg-gray-800` | Dark navigation bar |
| `<header>` | `bg-white shadow-sm` | Lighter page header with the current page title |
| `<main>` | — | Content injected via `{{ $slot }}` |

### Installing `@tailwindplus/elements`

The stacked layout uses Tailwind Plus's `<el-dropdown>` and `<el-disclosure>` custom elements for the profile dropdown and mobile menu. Include the script either via CDN or npm:

```html
<!-- CDN (development) -->
<script src="https://cdn.jsdelivr.net/npm/@tailwindplus/elements@1" type="module"></script>
```

```bash
# npm (production)
npm install @tailwindplus/elements
```

### Raw Tailwind Stacked Layout HTML

This is the unstyled starter template provided by Tailwind CSS (before any Laravel/Blade integration):

```html
<div class="min-h-full">
  <nav class="bg-gray-800 dark:bg-gray-800/50">
    <div class="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8">
      <div class="flex h-16 items-center justify-between">
        <div class="flex items-center">
          <div class="shrink-0">
            <img src="..." alt="Your Company" class="size-8" />
          </div>
          <div class="hidden md:block">
            <div class="ml-10 flex items-baseline space-x-4">
              <!-- Current: "bg-gray-900 dark:bg-gray-950/50 text-white" -->
              <!-- Default: "text-gray-300 hover:bg-white/5 hover:text-white" -->
              <a href="#" aria-current="page"
                 class="rounded-md bg-gray-900 px-3 py-2 text-sm font-medium text-white dark:bg-gray-950/50">
                Dashboard
              </a>
              <a href="#"
                 class="rounded-md px-3 py-2 text-sm font-medium text-gray-300 hover:bg-white/5 hover:text-white">
                Team
              </a>
            </div>
          </div>
        </div>

        <!-- Right side: notifications + profile dropdown -->
        <div class="hidden md:block">
          <div class="ml-4 flex items-center md:ml-6">
            <!-- Bell icon button -->
            <button type="button" class="relative rounded-full p-1 text-gray-400 hover:text-white ...">
              <!-- SVG bell icon -->
            </button>

            <!-- Profile dropdown using el-dropdown -->
            <el-dropdown class="relative ml-3">
              <button class="relative flex max-w-xs items-center rounded-full ...">
                <img src="..." alt="" class="size-8 rounded-full" />
              </button>
              <el-menu anchor="bottom end" popover class="w-48 ...">
                <a href="#">Your profile</a>
                <a href="#">Settings</a>
                <a href="#">Sign out</a>
              </el-menu>
            </el-dropdown>
          </div>
        </div>

        <!-- Mobile hamburger button -->
        <div class="-mr-2 flex md:hidden">
          <button type="button" command="--toggle" commandfor="mobile-menu" class="...">
            <!-- Hamburger / X icons toggle via aria-expanded -->
          </button>
        </div>
      </div>
    </div>

    <!-- Mobile menu panel (hidden by default) -->
    <el-disclosure id="mobile-menu" hidden class="block md:hidden">
      <div class="space-y-1 px-2 pt-2 pb-3 sm:px-3">
        <a href="#" aria-current="page" class="block rounded-md bg-gray-900 px-3 py-2 ...">Dashboard</a>
        <a href="#" class="block rounded-md px-3 py-2 ...">Team</a>
      </div>
    </el-disclosure>
  </nav>

  <header class="bg-white shadow-sm ...">
    <div class="mx-auto max-w-7xl px-4 py-6 sm:px-6 lg:px-8">
      <h1 class="text-3xl font-bold tracking-tight text-gray-900">Dashboard</h1>
    </div>
  </header>

  <main>
    <div class="mx-auto max-w-7xl px-4 py-6 sm:px-6 lg:px-8">
      <!-- Your content -->
    </div>
  </main>
</div>
```

### Key Responsive Patterns

The layout uses Tailwind's `md:` breakpoint to switch between desktop and mobile views:

| Class | Meaning |
|---|---|
| `hidden md:block` | Hidden on mobile, visible on desktop (md = 768px+) |
| `block md:hidden` | Visible on mobile only |
| `-mr-2 flex md:hidden` | Mobile hamburger button, hidden on desktop |
| `ml-10 flex items-baseline space-x-4` | Desktop nav link row with horizontal spacing |

### The Mobile Menu — `<el-disclosure>`

On small screens, the nav links collapse into a hidden panel toggled by a hamburger button. The `@tailwindplus/elements` library handles this with two attributes:

- `command="--toggle" commandfor="mobile-menu"` on the button — triggers the toggle
- `<el-disclosure id="mobile-menu" hidden>` — the collapsible panel

The hamburger icon and the X icon are both present in the DOM but toggled via `in-aria-expanded:hidden` and `not-in-aria-expanded:hidden` CSS classes that respond to the element's ARIA state automatically.

### The Profile Dropdown — `<el-dropdown>` / `<el-menu>`

The profile dropdown uses:

- `<el-dropdown>` — wrapper that manages open/close state
- `<el-menu anchor="bottom end" popover>` — the dropdown panel, anchored to the bottom-right of the trigger button
- CSS transition classes (`data-closed:opacity-0`, `data-enter:duration-100`, etc.) handle the open/close animation

---

---

## 1. The Problem: Hardcoded Active Styles

Out of the box, a hand-written navigation might look like this in a Blade layout file:

```html
<!-- resources/views/layouts/app.blade.php -->
<nav>
    <a href="/" class="bg-gray-900 text-white px-3 py-2 rounded-md text-sm font-medium"
       aria-current="page">
        Home
    </a>
    <a href="/about" class="text-gray-300 hover:bg-gray-700 hover:text-white px-3 py-2 rounded-md text-sm font-medium">
        About
    </a>
    <a href="/contact" class="text-gray-300 hover:bg-gray-700 hover:text-white px-3 py-2 rounded-md text-sm font-medium">
        Contact
    </a>
</nav>
```

Here the "Home" link always gets the `bg-gray-900 text-white` active styling, even when the user is on `/about` or `/contact`. We need to make this dynamic.

---

## 2. Fixing the Body/HTML Height (Tailwind Housekeeping)

Before diving into navigation logic, Tailwind requires the `html` and `body` tags to have `h-full` (height: 100%) and a background set for layouts to fill the screen properly:

```html
<!DOCTYPE html>
<html lang="en" class="h-full bg-gray-100">
<body class="h-full">
    ...
</body>
</html>
```

Without `h-full` on both elements, full-height containers will collapse to fit only their content.

---

## 3. Understanding the Tailwind Active vs Inactive Classes

The two states for navigation links are:

| State    | Tailwind Classes                                             | Description                        |
|----------|--------------------------------------------------------------|------------------------------------|
| Active   | `bg-gray-900 text-white`                                     | Dark background, white text        |
| Inactive | `text-gray-300 hover:bg-gray-700 hover:text-white`           | Light text, darker on hover        |

In Tailwind's scale, `100` is the lightest shade and `900` is the darkest. So `gray-900` is near-black, giving the active link a strong visual contrast.

---

## 4. Using Laravel's `request()->is()` Helper

Laravel ships with a global `request()` helper that gives you access to the current HTTP request. One of its most useful methods for navigation is `is()`, which accepts a URL pattern (including wildcards) and returns `true` if the current URL matches.

### Basic Usage

```php
request()->is('/')         // true only on the homepage
request()->is('about')     // true on /about
request()->is('contact')   // true on /contact
request()->is('blog/*')    // true on /blog/anything
```

### Applying it Inline with a Ternary Operator

```html
<a href="/"
   class="{{ request()->is('/') 
       ? 'bg-gray-900 text-white' 
       : 'text-gray-300 hover:bg-gray-700 hover:text-white' }} 
       px-3 py-2 rounded-md text-sm font-medium"
   aria-current="{{ request()->is('/') ? 'page' : 'false' }}">
    Home
</a>

<a href="/about"
   class="{{ request()->is('about') 
       ? 'bg-gray-900 text-white' 
       : 'text-gray-300 hover:bg-gray-700 hover:text-white' }} 
       px-3 py-2 rounded-md text-sm font-medium"
   aria-current="{{ request()->is('about') ? 'page' : 'false' }}">
    About
</a>

<a href="/contact"
   class="{{ request()->is('contact') 
       ? 'bg-gray-900 text-white' 
       : 'text-gray-300 hover:bg-gray-700 hover:text-white' }} 
       px-3 py-2 rounded-md text-sm font-medium"
   aria-current="{{ request()->is('contact') ? 'page' : 'false' }}">
    Contact
</a>
```

This works, but the repeated inline logic makes the layout file messy and hard to maintain — especially once you add responsive classes.

---

## 5. Extracting to a Blade Component

The cleaner solution is to create a dedicated `NavLink` Blade component.

### Creating the Component File

```
resources/views/components/nav-link.blade.php
```

You can generate this via Artisan (if using class-based components) or create the file manually for anonymous components.

---

## 6. Attributes vs. Props in Blade Components

Before writing the component, it's important to understand the difference between **attributes** and **props** in Blade.

| Concept    | What it is                                       | Example              |
|------------|--------------------------------------------------|----------------------|
| Attribute  | Standard HTML attribute passed through to the tag | `href`, `id`, `class` |
| Prop       | Custom data for component logic, NOT rendered as HTML | `active`, `type`   |

If you pass a custom value like `active="true"` to a component without declaring it as a prop, Laravel will treat it as an HTML attribute and render `<a active="true" ...>` — which is invalid HTML.

To distinguish props from attributes, declare them using the `@props` directive at the top of your component.

### Declaring a Prop

```blade
{{-- resources/views/components/nav-link.blade.php --}}
@props(['active' => false])

<a {{ $attributes }}
   class="{{ $active 
       ? 'bg-gray-900 text-white' 
       : 'text-gray-300 hover:bg-gray-700 hover:text-white' }} 
       px-3 py-2 rounded-md text-sm font-medium"
   aria-current="{{ $active ? 'page' : 'false' }}">
    {{ $slot }}
</a>
```

Breaking this down:

- `@props(['active' => false])` — declares `active` as a prop with a default value of `false`.
- `$active` is now available as a PHP variable in the component template.
- `{{ $attributes }}` — echoes all remaining HTML attributes (like `href`, `id`, etc.) onto the tag.
- `{{ $slot }}` — renders whatever content is placed between the component tags (e.g. "Home", "About").

---

## 7. Using the Component in the Layout

```html
<nav>
    <x-nav-link href="/" :active="request()->is('/')">
        Home
    </x-nav-link>

    <x-nav-link href="/about" :active="request()->is('about')">
        About
    </x-nav-link>

    <x-nav-link href="/contact" :active="request()->is('contact')">
        Contact
    </x-nav-link>
</nav>
```

---

## 8. The Colon Prefix — Passing Expressions vs. Strings

A critical detail: without the colon prefix, all prop values are treated as **strings**.

### Without Colon (String)

```html
<x-nav-link active="false">Home</x-nav-link>
```

Here, `"false"` is a 5-character string. PHP considers any non-empty string truthy — so `active` evaluates to `true` even though you wrote `false`. This is a common gotcha.

### With Colon (PHP Expression)

```html
<x-nav-link :active="false">Home</x-nav-link>
```

The colon prefix tells Blade: *evaluate this as a PHP expression*, not a string literal. Now `false` is the actual boolean `false`.

### Practical Comparison

```html
{{-- Passing a hardcoded boolean --}}
<x-nav-link :active="true">Home</x-nav-link>    {{-- active = true  ✓ --}}
<x-nav-link :active="false">Home</x-nav-link>   {{-- active = false ✓ --}}
<x-nav-link active="false">Home</x-nav-link>    {{-- active = true  ✗ (string!) --}}

{{-- Passing a PHP expression --}}
<x-nav-link :active="request()->is('/')">Home</x-nav-link>   {{-- evaluates at runtime ✓ --}}
```

---

## 9. The `aria-current` Accessibility Attribute

For screen readers and accessibility tools, the `aria-current="page"` attribute communicates that an anchor represents the user's current page in a set of links. When a link is not the current page, set it to `"false"` or omit it.

```blade
aria-current="{{ $active ? 'page' : 'false' }}"
```

This is especially important for production applications and contributes to WCAG compliance.

---

## 10. Adding PHP Logic Inside the Component

For more complex cases, you can add a `@php` block at the top of the component to pre-process prop values:

```blade
@props(['active' => false])

@php
    // Example: normalize active to a strict boolean
    $isActive = (bool) $active;

    // You could also inspect the slot or other props here
    $classes = $isActive
        ? 'bg-gray-900 text-white px-3 py-2 rounded-md text-sm font-medium'
        : 'text-gray-300 hover:bg-gray-700 hover:text-white px-3 py-2 rounded-md text-sm font-medium';
@endphp

<a {{ $attributes }} class="{{ $classes }}" aria-current="{{ $isActive ? 'page' : 'false' }}">
    {{ $slot }}
</a>
```

This keeps your conditional logic isolated in one file, making the template itself easy to read.

---

## 11. Homework — Adding a `type` Prop

A great exercise to reinforce this is adding a `type` prop that controls whether the component renders as an `<a>` tag or a `<button>` tag. This is useful when a navigation action triggers a form submission (like logout) rather than a page link.

### Updated Component

```blade
@props([
    'active' => false,
    'type'   => 'a',
])

@php
    $classes = ($active
        ? 'bg-gray-900 text-white'
        : 'text-gray-300 hover:bg-gray-700 hover:text-white')
        . ' px-3 py-2 rounded-md text-sm font-medium';
@endphp

@if ($type === 'a')
    <a {{ $attributes }} class="{{ $classes }}" aria-current="{{ $active ? 'page' : 'false' }}">
        {{ $slot }}
    </a>
@elseif ($type === 'button')
    <button {{ $attributes }} class="{{ $classes }}">
        {{ $slot }}
    </button>
@endif
```

### Usage

```html
{{-- Renders as an anchor tag (default) --}}
<x-nav-link href="/" :active="request()->is('/')">
    Home
</x-nav-link>

{{-- Renders as a button (e.g. for logout form) --}}
<x-nav-link type="button" onclick="document.getElementById('logout-form').submit()">
    Logout
</x-nav-link>
```

---

## 12. Complete Final Project Files

Below are all five files from the finished demo project.

### `resources/views/layouts/layout.blade.php`

The main layout file integrates the Tailwind Stacked Layout shell with the `<x-nav-link>` component and a named `$heading` slot for the page header title.

```html
<!doctype html>
<html lang="en" class="h-full bg-gray-100">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/@tailwindplus/elements@1" type="module"></script>
</head>
<body class="h-full">

<div class="min-h-full">
    <nav class="bg-gray-800 dark:bg-gray-800/50">
        <div class="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8">
            <div class="flex h-16 items-center justify-between">
                <div class="flex items-center">
                    <div class="shrink-0">
                        <img src="https://cdn.jsdelivr.net/gh/homarr-labs/dashboard-icons/svg/laracasts-dark.svg"
                             alt="Your Company" class="size-8" />
                    </div>
                    <div class="hidden md:block">
                        <div class="ml-10 flex items-baseline space-x-4">
                            {{-- x-nav-link components with :active bound to request()->is() --}}
                            <x-nav-link href="/" :active="request()->is('/')">Home</x-nav-link>
                            <x-nav-link href="/about" :active="request()->is('about')">About</x-nav-link>
                            <x-nav-link href="/contact" :active="request()->is('contact')">Contact</x-nav-link>
                        </div>
                    </div>
                </div>

                {{-- Desktop: notifications + profile dropdown --}}
                <div class="hidden md:block">
                    <div class="ml-4 flex items-center md:ml-6">
                        <button type="button" class="relative rounded-full p-1 text-gray-400 hover:text-white ...">
                            {{-- Bell icon SVG --}}
                        </button>
                        <el-dropdown class="relative ml-3">
                            <button class="relative flex max-w-xs items-center rounded-full ...">
                                <img src="https://assets.laracasts.com/images/mascot/larysmart.svg"
                                     alt="" class="size-8 rounded-full" />
                            </button>
                            <el-menu anchor="bottom end" popover class="w-48 ...">
                                <a href="#">Your profile</a>
                                <a href="#">Settings</a>
                                <a href="#">Sign out</a>
                            </el-menu>
                        </el-dropdown>
                    </div>
                </div>

                {{-- Mobile: hamburger toggle --}}
                <div class="-mr-2 flex md:hidden">
                    <button type="button" command="--toggle" commandfor="mobile-menu" class="...">
                        {{-- Hamburger / X SVG icons --}}
                    </button>
                </div>
            </div>
        </div>

        {{-- Mobile menu panel --}}
        <el-disclosure id="mobile-menu" hidden class="block md:hidden">
            <div class="space-y-1 px-2 pt-2 pb-3 sm:px-3">
                <a href="#" aria-current="page"
                   class="block rounded-md bg-gray-900 px-3 py-2 text-base font-medium text-white">Home</a>
                <a href="#"
                   class="block rounded-md px-3 py-2 text-base font-medium text-gray-300 hover:bg-white/5 hover:text-white">About</a>
                <a href="#"
                   class="block rounded-md px-3 py-2 text-base font-medium text-gray-300 hover:bg-white/5 hover:text-white">Contact</a>
            </div>
        </el-disclosure>
    </nav>

    {{-- Page header — title injected via named slot $heading --}}
    <header class="relative bg-white shadow-sm dark:bg-gray-800">
        <div class="mx-auto max-w-7xl px-4 py-6 sm:px-6 lg:px-8">
            <h1 class="text-3xl font-bold tracking-tight text-gray-900 dark:text-white">
                {{ $heading }}
            </h1>
        </div>
    </header>

    <main>
        <div class="mx-auto max-w-7xl px-4 py-6 sm:px-6 lg:px-8">
            {{ $slot }}
        </div>
    </main>
</div>

</body>
</html>
```

> **Note on `$heading`:** This is a named slot. Each page view passes its own title using `<x-slot:heading>`. This is different from `$slot`, which is the unnamed default slot for the main content.

---

### `resources/views/components/nav-link.blade.php`

```blade
{{-- Active:  "bg-gray-900 dark:bg-gray-950/50 text-white"         --}}
{{-- Default: "text-gray-300 hover:bg-white/5 hover:text-white"    --}}

@props(['active' => false])

<a
   class="{{ $active
         ? 'bg-gray-900 dark:bg-gray-950/50 text-white'
         : 'text-gray-300 hover:bg-white/5 hover:text-white'
         }} rounded-md px-3 py-2 text-sm font-medium"
   aria-current="{{ $active ? 'page' : 'false' }}"
   {{ $attributes }}
>{{ $slot }}</a>
```

Notice two important things from the real file compared to the basic version shown earlier:

1. **Dark mode classes are included** — `dark:bg-gray-950/50` on the active state ensures the active pill looks correct in dark mode too. `hover:bg-white/5` uses a translucent white overlay rather than a specific gray shade, which works cleanly on both light and dark backgrounds.
2. **`aria-current` is set based on `$active`** — not on a separate `request()->is()` call. Since `$active` already holds the result of that check, it's reused here to avoid duplication.

---

### `resources/views/home.blade.php`

```blade
<x-layout>
    <x-slot:heading>
        Home Page
    </x-slot:heading>

    <h1>Hello from home page</h1>
</x-layout>
```

---

### `resources/views/about.blade.php`

```blade
<x-layout>
    <x-slot:heading>
        About Page
    </x-slot:heading>

    <h1>Hello from about page</h1>
</x-layout>
```

---

### `resources/views/contact.blade.php`

```blade
<x-layout>
    <x-slot:heading>
        Contact Page
    </x-slot:heading>

    <h1>Hello from contact page</h1>
</x-layout>
```

### How Named Slots Work Here

Each page view passes two pieces of content into `<x-layout>`:

| Slot | Syntax | Where it renders |
|---|---|---|
| `$heading` | `<x-slot:heading>Page Title</x-slot:heading>` | Inside the `<header>` `<h1>` tag |
| `$slot` (default) | Any content directly inside `<x-layout>` | Inside the `<main>` content area |

This pattern keeps individual page views extremely minimal — they only declare what's unique about that page (the heading and the body content), while all chrome (nav, header, footer) lives in one place.

---

## Summary

| Concept | Key Takeaway |
|---|---|
| `request()->is('path')` | Checks if the current URL matches a pattern |
| Ternary in Blade | `{{ $condition ? 'a' : 'b' }}` for inline conditional output |
| `@props([])` | Declares component props, preventing them from leaking into HTML |
| Colon prefix (`:active`) | Passes a PHP expression instead of a string literal |
| `$attributes` | Echoes all non-prop HTML attributes onto the element |
| `$slot` | Renders the content between component tags |
| `aria-current="page"` | Accessibility attribute indicating the current page link |

---
