# Ep 04: Make a Pretty Layout Using Tailwind CSS

## Overview

Episode 4 has two main goals. The first half reviews the Episode 3 homework — completing the `nav-link` component and learning how to pass HTML attributes (like `href`) into components using the `$attributes` object. The second half introduces **Tailwind CSS** and the **Tailwind UI stacked layout**, transforming the plain HTML app into a polished, professional-looking interface. It also introduces **named slots** — a way to inject content into multiple distinct areas of a layout component.

---

## 1. Homework Review — Completing the `nav-link` Component

### Creating the Component

From the Episode 3 homework, the `nav-link` component file should be at:

```
resources/views/components/nav-link.blade.php
```

> **Note:** The `components` directory name can be capitalised or lowercase — both work in Laravel. Lowercase is the convention.

Start by extracting a single `<a>` tag into the component:

```html
<!-- resources/views/components/nav-link.blade.php -->
<a href="#">{{ $slot }}</a>
```

And update the layout to use it with the slot technique for the link labels:

```html
<!-- resources/views/components/layout.blade.php -->
<nav>
    <x-nav-link>Home</x-nav-link>
    <x-nav-link>About</x-nav-link>
    <x-nav-link>Contact</x-nav-link>
</nav>
```

This renders three links, but they all point to `#`. The `href` needs to be dynamic.

---

## 2. The `$attributes` Object

The challenge: how do you pass an `href` (or any HTML attribute) into a component and have it appear on the rendered tag?

Every Blade component automatically has access to a special **`$attributes` object** that collects all HTML attributes passed to the component that are *not* declared as props. This includes `href`, `id`, `class`, `style`, and any other standard HTML attribute.

### Echoing All Attributes

The simplest approach is to echo the entire `$attributes` object directly onto the tag:

```html
<!-- resources/views/components/nav-link.blade.php -->
<a {{ $attributes }}>{{ $slot }}</a>
```

Now in the layout, pass the `href` just as you would on a normal HTML element:

```html
<x-nav-link href="/">Home</x-nav-link>
<x-nav-link href="/about">About</x-nav-link>
<x-nav-link href="/contact">Contact</x-nav-link>
```

Laravel collects `href="/"`, `href="/about"`, and `href="/contact"` into the `$attributes` object and renders them onto the `<a>` tag automatically.

### What `$attributes` Captures

Any attribute you pass that isn't declared as a `@prop` ends up in `$attributes`:

```html
<!-- Passing href -->
<x-nav-link href="/about">About</x-nav-link>
{{-- Renders: <a href="/about">About</a> --}}

<!-- Passing href + id -->
<x-nav-link href="/about" id="about-link">About</x-nav-link>
{{-- Renders: <a href="/about" id="about-link">About</a> --}}

<!-- Passing href + inline style -->
<x-nav-link href="/about" style="color: green;">About</x-nav-link>
{{-- Renders: <a href="/about" style="color: green;">About</a> --}}
```

### Why Not Just Hardcode the `href` Inside the Component?

Because that defeats the purpose of the component. A reusable `nav-link` needs to work for any URL. By relying on `$attributes`, the component stays flexible — the caller decides what attributes to pass.

### The `$attributes->merge()` Method

For more advanced cases, `$attributes` is a full object with methods. The most useful is `merge()`, which lets you define sensible default classes that can be overridden or extended:

```html
<!-- Apply default classes, merge with any passed class attribute -->
<a {{ $attributes->merge(['class' => 'rounded-md px-3 py-2 text-sm font-medium']) }}>
    {{ $slot }}
</a>
```

This isn't needed yet, but it becomes important in Episode 5 when active link styling is added.

---

## 3. Why Extract Nav Links into a Component?

A plain `<a>` tag seems too simple to justify a component. But in a real application, a navigation link is rarely just a tag — it needs:

- Different CSS classes depending on whether it is the **currently active page**
- Different styling based on **screen size** (desktop vs. mobile)
- Possibly an **icon** alongside the label
- Accessibility attributes like `aria-current`

Isolating all of this complexity into a single `nav-link.blade.php` file keeps the layout clean and gives you one place to update when requirements change.

---

## 4. Introducing Tailwind CSS

**Tailwind CSS** is a **utility-first CSS framework**. Instead of writing custom CSS in a separate stylesheet, you apply small, single-purpose CSS classes directly in your HTML.

### Utility Classes Explained

| Tailwind Class | CSS Equivalent | Description |
|---|---|---|
| `text-red-500` | `color: rgb(239, 68, 68)` | Red text (500 = mid-range shade) |
| `bg-gray-800` | `background-color: rgb(31, 41, 55)` | Dark gray background |
| `px-3` | `padding-left: 0.75rem; padding-right: 0.75rem` | Horizontal padding level 3 |
| `py-2` | `padding-top: 0.5rem; padding-bottom: 0.5rem` | Vertical padding level 2 |
| `mr-2` | `margin-right: 0.5rem` | Right margin level 2 |
| `flex` | `display: flex` | Flexbox container |
| `hidden` | `display: none` | Hide element |
| `md:block` | `display: block` (at md breakpoint) | Show on medium screens and above |
| `h-full` | `height: 100%` | Full height |
| `text-sm` | `font-size: 0.875rem` | Small text |
| `font-medium` | `font-weight: 500` | Medium font weight |
| `rounded-md` | `border-radius: 0.375rem` | Medium border radius |

> Tailwind is **not a prerequisite** for this course. It is used purely to quickly produce attractive layouts without writing custom CSS. You can follow along with the concepts even if you don't know Tailwind.

### Including Tailwind via CDN

For development and learning purposes, Tailwind can be loaded from a CDN with a single script tag — no build tools required:

```html
<head>
    <script src="https://cdn.tailwindcss.com"></script>
</head>
```

> In a production application you would install Tailwind via npm and run it through a build tool (Vite) to generate an optimised, purged CSS file. The CDN version is for prototyping only.

---

## 5. Tailwind UI — Pre-Built Layout Components

**Tailwind UI** (`tailwindui.com`) is a paid component library built by the Tailwind CSS team. It provides production-ready HTML+Tailwind snippets for common UI patterns. However, they offer several **free** examples, and the stacked layout used in this series is one of them.

### The Stacked Layout

The free stacked layout provides a complete application shell:

- Dark navigation bar (`bg-gray-800`) with logo, nav links, notification bell, and profile dropdown
- A white page header section with a large page title (`<h1>`)
- A main content area below

This is the layout pasted into `layout.blade.php` in this episode, replacing the basic hand-written HTML.

---

## 6. Integrating the Stacked Layout into Laravel

After pasting the Tailwind HTML into `layout.blade.php`, two integration steps are needed:

### Step 1 — Add the `{{ $slot }}` placeholder

Find the `<!-- Your content -->` comment in the main area of the template and replace it with the Blade slot:

```html
<main>
    <div class="mx-auto max-w-7xl px-4 py-6 sm:px-6 lg:px-8">
        {{ $slot }}
    </div>
</main>
```

### Step 2 — Update the nav link text and hrefs

The Tailwind template ships with placeholder link names like "Dashboard", "Team", "Projects". Replace these with your actual routes — and do it in **both** the desktop nav and the mobile nav (the template includes both):

```html
<!-- Desktop nav links -->
<x-nav-link href="/">Home</x-nav-link>
<x-nav-link href="/about">About</x-nav-link>
<x-nav-link href="/contact">Contact</x-nav-link>

<!-- Mobile nav links (separate block in the template) -->
<a href="/">Home</a>
<a href="/about">About</a>
<a href="/contact">Contact</a>
```

> The mobile links are inside an `<el-disclosure>` panel that is hidden on desktop. It's easy to miss — always check both sections when updating nav links.

---

## 7. Named Slots

The stacked layout introduces a new problem. The layout now has **two dynamic regions**:

1. The main content area (`$slot`) — already handled
2. The page title in the `<header>` section — not yet handled

A single `$slot` can only hold one piece of content. For multiple injectable regions, Laravel provides **named slots**.

### Defining a Named Slot in the Layout

In `layout.blade.php`, replace the hardcoded heading text with a named slot variable:

```html
<header class="bg-white shadow-sm">
    <div class="mx-auto max-w-7xl px-4 py-6 sm:px-6 lg:px-8">
        <h1 class="text-3xl font-bold tracking-tight text-gray-900">
            {{ $heading }}
        </h1>
    </div>
</header>
```

`$heading` is not a default variable like `$slot` — you have to populate it from the calling view.

### Passing Content into a Named Slot

In each page view, use `<x-slot:name>` to target a specific named slot:

```html
<!-- resources/views/home.blade.php -->
<x-layout>
    <x-slot:heading>
        Home Page
    </x-slot:heading>

    <h1>Hello from the homepage</h1>
</x-layout>
```

```html
<!-- resources/views/about.blade.php -->
<x-layout>
    <x-slot:heading>
        About Page
    </x-slot:heading>

    <h1>Hello from the about page</h1>
</x-layout>
```

```html
<!-- resources/views/contact.blade.php -->
<x-layout>
    <x-slot:heading>
        Contact Page
    </x-slot:heading>

    <h1>Hello from the contact page</h1>
</x-layout>
```

### Default Slot vs Named Slots

| Slot Type | Layout syntax | Usage syntax | Purpose |
|---|---|---|---|
| Default slot | `{{ $slot }}` | Content directly inside `<x-layout>` | Main page content |
| Named slot | `{{ $heading }}` | `<x-slot:heading>...</x-slot:heading>` | Other injectable regions like page title |

You can have as many named slots as you need. Common examples:

```html
{{ $slot }}      <!-- main content -->
{{ $heading }}   <!-- page title -->
{{ $sidebar }}   <!-- sidebar content -->
{{ $footer }}    <!-- page-specific footer content -->
{{ $scripts }}   <!-- page-specific JavaScript -->
```

---

## 8. The Alternative — Passing Heading as a Prop

Named slots are not the only option. The page heading could also be passed as a **component prop** (a custom attribute):

```html
<!-- As a prop (alternative approach) -->
<x-layout heading="Home Page">
    <h1>Hello from the homepage</h1>
</x-layout>
```

For short, simple strings like a page title, a prop can be cleaner. Named slots are better when the content is longer, contains HTML markup, or benefits from being on its own line. Both approaches are valid — the named slot approach is used in this series.

---

## 9. Final `layout.blade.php` Structure

```html
<!DOCTYPE html>
<html lang="en" class="h-full bg-gray-100">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="h-full">

<div class="min-h-full">
    <nav class="bg-gray-800">
        <div class="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8">
            <div class="flex h-16 items-center justify-between">
                <div class="flex items-center">
                    {{-- Logo --}}
                    <div class="shrink-0">
                        <img src="..." alt="Logo" class="size-8" />
                    </div>
                    {{-- Desktop nav links --}}
                    <div class="hidden md:block">
                        <div class="ml-10 flex items-baseline space-x-4">
                            <x-nav-link href="/">Home</x-nav-link>
                            <x-nav-link href="/about">About</x-nav-link>
                            <x-nav-link href="/contact">Contact</x-nav-link>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        {{-- Mobile nav (shown on small screens) --}}
        <div class="md:hidden">
            <div class="space-y-1 px-2 pb-3 pt-2">
                <a href="/" class="block ...">Home</a>
                <a href="/about" class="block ...">About</a>
                <a href="/contact" class="block ...">Contact</a>
            </div>
        </div>
    </nav>

    {{-- Page header with dynamic heading via named slot --}}
    <header class="bg-white shadow-sm">
        <div class="mx-auto max-w-7xl px-4 py-6 sm:px-6 lg:px-8">
            <h1 class="text-3xl font-bold tracking-tight text-gray-900">
                {{ $heading }}
            </h1>
        </div>
    </header>

    {{-- Main content area — default slot --}}
    <main>
        <div class="mx-auto max-w-7xl px-4 py-6 sm:px-6 lg:px-8">
            {{ $slot }}
        </div>
    </main>
</div>

</body>
</html>
```

---

## 10. What's Left — Active Link Styling

At the end of this episode, there is still one outstanding issue: the navigation link styling is **hardcoded** to make the "Home" link always appear active, regardless of which page you are on. When you visit `/contact`, the Home link is still highlighted.

This is the problem tackled in **Episode 5**, where you learn to use `request()->is()` to detect the current URL and apply conditional Tailwind classes dynamically.

---

## Key Takeaways

| Concept | Summary |
|---|---|
| `$attributes` | Auto-available object in Blade components; holds all non-prop HTML attributes passed to the component |
| `{{ $attributes }}` | Echoes all attributes onto the HTML tag (`href`, `id`, `class`, etc.) |
| `$attributes->merge()` | Merges default attribute values with passed ones — useful for default CSS classes |
| Tailwind CSS | Utility-first CSS framework; classes map directly to CSS properties |
| CDN Tailwind | `<script src="https://cdn.tailwindcss.com">` — for development only |
| Tailwind UI | Pre-built HTML+Tailwind component library; free stacked layout used in this series |
| Named slots | `{{ $heading }}` in layout + `<x-slot:heading>` in the view — inject content into multiple regions |
| Default slot | `{{ $slot }}` — the unnamed slot holding main page content |
| `<x-slot:name>` | Syntax for targeting a specific named slot when using a component |
| No homework | Episode 4 has no homework assignment |

---
