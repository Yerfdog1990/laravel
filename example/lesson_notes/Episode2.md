# Ep 02: Your First Route and View

## Overview

Episode 2 introduces two of the most fundamental concepts in Laravel — **routes** and **views**. Every web request your application receives passes through a route, and the route decides what to send back to the user. In most cases, that response is a view — an HTML template the user sees in their browser. By the end of this episode you have a working multi-page Laravel application with two routes and two views.

---

## 1. Viewing Your Project in the Browser

If you installed **Laravel Herd** in Episode 1, your project is already being served automatically. Simply visit:

```
http://example.test
```

If you chose a different setup (XAMPP, Laragon, etc.) and don't have automatic domain serving, you can boot up a local development server using Laravel's built-in Artisan CLI tool:

```bash
php artisan serve
```

This starts a server at `http://127.0.0.1:8000` by default. You'll use `artisan` constantly throughout this series — more on it in later episodes.

---

## 2. Don't Be Overwhelmed by the File Structure

Opening a fresh Laravel project for the first time can feel intimidating. You'll see folders like:

```
app/        bootstrap/    config/      database/
resources/  routes/       storage/     tests/
vendor/     public/       ...
```

The key mindset: **you don't need to learn all of this at once.** The series introduces each part only when it becomes relevant. For now, the only two directories you need to care about are:

- `routes/` — where you define what URLs your app responds to
- `resources/views/` — where your HTML templates live

---

## 3. What is a Route?

A **route** is a mapping between a URL and a response. When a user visits a URL in their browser, Laravel checks its list of registered routes, finds a match, and executes the associated code to produce a response.

All web routes are defined in:

```
routes/web.php
```

### The Default Route

Open `routes/web.php` and you'll find this already there:

```php
<?php

use Illuminate\Support\Facades\Route;

Route::get('/', function () {
    return view('welcome');
});
```

Even without knowing PHP deeply, you can parse what this says:

- `Route::get('/')` — listen for a **GET request** to the homepage (`/`)
- The closure (anonymous function) runs when that URL is visited
- `return view('welcome')` — send back the `welcome` view as the response

> A **GET request** is what happens when you type a URL into your browser and hit Enter. It is the most common type of HTTP request and simply means "give me this page."

---

## 4. What is a View?

A **view** is the HTML template that gets sent to the user's browser. Views live in:

```
resources/views/
```

The default welcome view is at:

```
resources/views/welcome.blade.php
```

Note the `.blade.php` extension rather than plain `.php`. **Blade** is Laravel's templating engine — a thin layer on top of PHP that adds useful features like template inheritance, components, and directives. It will be covered properly in later episodes. For now, just know that both `.blade.php` and plain `.php` files work as views.

### Editing the Welcome View

To confirm how routes and views connect, replace the contents of the `<body>` tag in `welcome.blade.php` with something simple:

```html
<body>
    Hello, World!
</body>
```

Visit `http://example.test` and you'll see "Hello, World!" — the route loaded the view, and the view rendered your HTML.

---

## 5. Declaring Your First Custom Route

Now let's create a new route from scratch. In `routes/web.php`, add:

```php
Route::get('/about', function () {
    return 'About Page';
});
```

Visit `http://example.test/about` and you'll see the text "About Page" rendered directly in the browser.

### What Can a Route Return?

Routes are flexible — they can return several different types of responses:

| Return Type | Example | Use Case |
|---|---|---|
| A string | `return 'Hello';` | Quick debugging, simple text responses |
| An array | `return ['foo' => 'bar'];` | Automatically converted to JSON — useful for APIs |
| A view | `return view('about');` | Standard page rendering for web apps |
| A redirect | `return redirect('/');` | Send the user to another URL |

```php
// Returns plain text
Route::get('/about', function () {
    return 'About Page';
});

// Returns JSON automatically (great for APIs)
Route::get('/api/data', function () {
    return ['foo' => 'bar', 'status' => 'ok'];
});

// Returns a view (the standard approach for web pages)
Route::get('/about', function () {
    return view('about');
});
```

---

## 6. Creating a View for the About Page

Once your route is set up to `return view('about')`, Laravel will look for a file at:

```
resources/views/about.php
```

or

```
resources/views/about.blade.php
```

Create a simple `about.php` file (no Blade required yet):

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>About</title>
</head>
<body>
    <h1>About Page</h1>
    <p>Hello from the about page.</p>
</body>
</html>
```

Now update your route to return this view:

```php
Route::get('/about', function () {
    return view('about');
});
```

Visit `http://example.test/about` and your HTML page renders. You've created your first custom route and view.

---

## 7. How Laravel Resolves View Names

When you call `view('about')`, Laravel searches for a matching file in `resources/views/` using this priority order:

1. `resources/views/about.blade.php` ← checked first (Blade template)
2. `resources/views/about.php` ← fallback (plain PHP)

You pass only the **base name** without the extension or the `resources/views/` path prefix.

For views in subdirectories, use dot notation:

```php
// Loads resources/views/pages/about.blade.php
return view('pages.about');
```

---

## 8. The Complete `routes/web.php` So Far

After Episode 2, your routes file looks like this:

```php
<?php

use Illuminate\Support\Facades\Route;

// Homepage
Route::get('/', function () {
    return view('welcome');
});

// About page
Route::get('/about', function () {
    return view('about');
});
```

And your views directory:

```
resources/views/
├── welcome.blade.php   ← Laravel's default landing page (modified)
└── about.php           ← Your new about page view
```

---

## 9. The Request–Route–View Lifecycle

Understanding what happens when a user visits a URL is fundamental to everything that follows in this series:

```
Browser                 Laravel
──────                  ───────
User visits             routes/web.php
/about         ──►      Route::get('/about', ...) matched
                        Closure executes
                        view('about') called
                ◄──     resources/views/about.php rendered
                        HTML response sent to browser
```

1. The browser sends a **GET request** to `/about`
2. Laravel loads `routes/web.php` and finds a matching route
3. The closure (callback function) associated with that route runs
4. `view('about')` locates the template file and renders it
5. The rendered HTML is returned to the browser

---

## 10. Homework — Create a Contact Route and View

The homework for Episode 2 reinforces everything covered:

1. Open `routes/web.php` and add a new route for `/contact`
2. Create a new view file at `resources/views/contact.php` (or `.blade.php`)
3. Have the route return that view
4. Visit `http://example.test/contact` to confirm it works

### Solution

```php
// routes/web.php
Route::get('/contact', function () {
    return view('contact');
});
```

```html
<!-- resources/views/contact.php -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Contact</title>
</head>
<body>
    <h1>Contact Page</h1>
    <p>Get in touch with us.</p>
</body>
</html>
```

---

## Key Takeaways

| Concept | Summary |
|---|---|
| `routes/web.php` | The file where all web routes are registered |
| `Route::get($uri, $callback)` | Registers a route that responds to GET requests at the given URI |
| GET request | What happens when a user visits a URL in their browser |
| Closure | The anonymous function that runs when a route is matched |
| `view('name')` | Loads and renders a Blade or PHP template from `resources/views/` |
| `resources/views/` | Where all HTML templates (views) live |
| `.blade.php` | The extension for Blade templates — Laravel's templating engine |
| `php artisan serve` | Boots a local dev server if not using Herd |
| Dot notation | Used to reference views in subdirectories: `view('pages.about')` |
| Array return | Returning an array from a route auto-converts it to a JSON response |

---
