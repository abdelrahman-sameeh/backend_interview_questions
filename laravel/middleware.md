# Laravel Middleware Interview Questions

## Table of Contents

- [🟢 Junior — Basics](#-junior--basics)
  - [1. What is Middleware in Laravel?](#1-what-is-middleware-in-laravel)
  - [2. Why do we use Middleware?](#2-why-do-we-use-middleware)
  - [3. How do you create Middleware?](#3-how-do-you-create-middleware)
  - [4. What does `$next($request)` do?](#4-what-does-nextrequest-do)
  - [5. Can Middleware stop a Request?](#5-can-middleware-stop-a-request)
  - [6. How do you assign Middleware to a Route?](#6-how-do-you-assign-middleware-to-a-route)
  - [7. What is Global Middleware?](#7-what-is-global-middleware)
  - [8. What is Middleware Group?](#8-what-is-middleware-group)

- [🟡 Important Questions](#-important-questions)
  - [9. Global Middleware vs Route Middleware](#9-what-is-the-difference-between-global-middleware-and-route-middleware)
  - [10. How can Middleware receive parameters?](#10-how-can-middleware-receive-parameters)
  - [11. Can Middleware have multiple parameters?](#11-can-middleware-have-multiple-parameters)
  - [12. How do you apply multiple Middleware to a Route?](#12-how-do-you-apply-multiple-middleware-to-a-route)
  - [13. Can Middleware modify the Request?](#13-can-middleware-modify-the-request)
  - [14. Can Middleware modify the Response?](#14-can-middleware-modify-the-response)

- [🔴 Mid-Level Questions](#-mid-level-questions)
  - [15. What is the difference between `handle()` and `terminate()`?](#15-what-is-the-difference-between-handle-and-terminate)
  - [16. Explain Middleware execution order](#16-explain-middleware-execution-order)
  - [17. Middleware vs Controller](#17-whats-the-difference-between-middleware-and-controller)
  - [18. Middleware vs Form Request Validation](#18-middleware-vs-form-request-validation)
  - [19. Where should Authentication and Authorization happen?](#19-where-should-authentication-and-authorization-happen)
  - [20. Why shouldn't every check be placed inside Middleware?](#20-why-shouldnt-every-check-be-placed-inside-middleware)

- [🔥 Scenario-Based Interview Questions](#-scenario-based-interview-questions)
  - [21. Only admins can access `/admin/*`](#21-you-want-only-admins-to-access-admin-what-would-you-do)
  - [22. Limit an API to 100 requests per minute](#22-you-want-to-limit-an-api-to-100-requests-per-minute-what-would-you-use)
  - [23. Prevent disabled users from accessing the system](#23-a-user-is-authenticated-but-their-account-is-disabled-how-do-you-prevent-access)

- [⭐ Top Questions To Focus On](#-top-questions-to-focus-on)
- [🧠 Important Mental Model](#-important-mental-model)
- [🎯 The Main Idea](#-the-main-idea)


# 🟢 Junior — Basics

## 1. What is Middleware in Laravel?

Middleware is a layer between the HTTP Request and the Controller.

It is used to inspect, modify, or reject a request before it reaches the Controller.

Common use cases:

- Authentication
- Authorization
- Rate Limiting
- Logging
- CORS
- Checking account status

Example:

    Route::get('/profile', [ProfileController::class, 'index'])
        ->middleware('auth');


## 2. Why do we use Middleware?

Middleware is useful for logic that should be applied to multiple routes.

Examples:

- Check if the user is authenticated.
- Check if the user is an admin.
- Limit API requests.
- Log requests.
- Check if an account is active.


## 3. How do you create Middleware?

Command:

    php artisan make:middleware IsAdmin

Middleware:

    public function handle(Request $request, Closure $next)
    {
        // Middleware logic

        return $next($request);
    }


## 4. What does `$next($request)` do?

It allows the request to continue to the next middleware or to the Controller.

Example:

    public function handle(Request $request, Closure $next)
    {
        return $next($request);
    }

If you don't call `$next($request)` or return a response, the request will not continue.


## 5. Can Middleware stop a Request?

Yes.

Middleware can return a response instead of calling `$next()`.

Example:

    public function handle(Request $request, Closure $next)
    {
        if (!$request->user()->is_admin) {
            return response()->json([
                'message' => 'Forbidden'
            ], 403);
        }

        return $next($request);
    }

If the user is not an admin, the Controller will not execute.


## 6. How do you assign Middleware to a Route?

You can assign Middleware directly to a route:

    Route::get('/profile', [ProfileController::class, 'index'])
        ->middleware('auth');

Multiple Middleware:

    Route::get('/admin/orders', ...)
        ->middleware([
            'auth',
            'admin',
            'throttle:60,1'
        ]);


## 7. What is Global Middleware?

Global Middleware runs on every HTTP request.

In Laravel 10, global middleware is registered in:

    app/Http/Kernel.php

Example:

    protected $middleware = [
        // Global middleware
    ];

Common examples:

- CORS
- Maintenance mode
- Request processing
- Security-related middleware


## 8. What is Middleware Group?

A Middleware Group is a collection of middleware under one name.

Example:

    Route::middleware('api')->group(function () {

        Route::get('/products', ...);
        Route::get('/orders', ...);

    });

Instead of adding the same middleware to every route.


# 🟡 Important Questions

## 9. What is the difference between Global Middleware and Route Middleware?

| Global Middleware | Route Middleware |
|---|---|
| Runs on every request | Runs only on selected routes |
| Automatically applied | Explicitly assigned |
| Used for general concerns | Used for specific routes |
| Example: CORS | Example: auth/admin |

Simple idea:

    Global Middleware
        ↓
    Every Request

    Route Middleware
        ↓
    Specific Routes


## 10. How can Middleware receive parameters?

You can pass parameters through the route.

Example:

    Route::get('/admin', ...)
        ->middleware('role:admin');

Middleware:

    public function handle(
        Request $request,
        Closure $next,
        $role
    ) {
        if ($request->user()->role !== $role) {
            return response()->json([
                'message' => 'Forbidden'
            ], 403);
        }

        return $next($request);
    }


## 11. Can Middleware have multiple parameters?

Yes.

Example:

    Route::get('/products', ...)
        ->middleware('rate.limit:100,1');

Middleware:

    public function handle(
        Request $request,
        Closure $next,
        $limit,
        $minutes
    ) {
        // Logic

        return $next($request);
    }

Here:

    100 = limit
    1   = minute


## 12. How do you apply multiple Middleware to a Route?

You can use an array:

    Route::get('/admin/orders', ...)
        ->middleware([
            'auth',
            'admin',
            'throttle:60,1'
        ]);

Or use a group:

    Route::middleware(['auth', 'admin'])->group(function () {

        Route::get('/orders', ...);
        Route::get('/users', ...);

    });


## 13. Can Middleware modify the Request?

Yes.

You can add or modify request data.

Example:

    public function handle(Request $request, Closure $next)
    {
        $request->merge([
            'country' => 'Egypt'
        ]);

        return $next($request);
    }

Now the Controller can access:

    $request->country;


## 14. Can Middleware modify the Response?

Yes.

You can get the response from `$next()` and modify it.

Example:

    public function handle(Request $request, Closure $next)
    {
        $response = $next($request);

        $response->headers->set(
            'X-App-Version',
            '1.0'
        );

        return $response;
    }


# 🔴 Mid-Level Questions

## 15. What is the difference between `handle()` and `terminate()`?

`handle()` runs during the request lifecycle.

Example:

    public function handle(
        Request $request,
        Closure $next
    ) {
        return $next($request);
    }

`terminate()` can be used for work after the response has been sent in Laravel's terminable middleware flow.

Example:

    public function terminate(
        Request $request,
        $response
    ) {
        // Logging
        // Analytics
        // Cleanup
    }

Common use cases:

- Logging
- Analytics
- Cleanup


## 16. Explain Middleware execution order

If you have:

    Route::middleware([
        'auth',
        'admin',
        'throttle'
    ]);

The request passes through the middleware in order:

    Request
       ↓
    auth
       ↓
    admin
       ↓
    throttle
       ↓
    Controller

After the Controller finishes, the response travels back through the middleware stack.


## 17. What's the difference between Middleware and Controller?

Middleware handles common logic around HTTP requests.

Examples:

- Authentication
- Authorization
- Logging
- Rate Limiting

Controller handles endpoint/business operations.

Example:

    Middleware → Is the user allowed to access this endpoint?

    Controller → Create the Order.

Simple rule:

    Middleware
        ↓
    "Can this request continue?"

    Controller
        ↓
    "What should the application do?"


## 18. Middleware vs Form Request Validation?

Middleware answers:

> Is this request allowed to enter?

Form Request answers:

> Is the submitted data valid?

Typical flow:

    Request
       ↓
    auth middleware
       ↓
    admin middleware
       ↓
    Form Request Validation
       ↓
    Controller


## 19. Where should Authentication and Authorization happen?

Authentication answers:

> Who is the user?

Authorization answers:

> Is this user allowed to perform this action?

They are commonly handled using middleware.

Example:

    Route::middleware(['auth', 'admin'])->group(function () {

        Route::get('/admin/orders', ...);
        Route::delete('/admin/products/{product}', ...);

    });

Authentication:

    auth
      ↓
    "Is the user logged in?"

Authorization:

    admin
      ↓
    "Does the user have permission?"


## 20. Why shouldn't every check be placed inside Middleware?

Middleware should not contain all business logic.

Bad example:

    if ($cart->total > 5000) {
        // Business logic
    }

Middleware is better for cross-cutting concerns such as:

- Authentication
- Authorization
- Logging
- Rate Limiting

Business logic such as:

- Calculate Order
- Apply Coupon
- Validate Cart
- Create Payment

should normally be handled by Services, Controllers, or domain/business layers depending on the architecture.


# 🔥 Scenario-Based Interview Questions

## 21. You want only admins to access `/admin/*`. What would you do?

Create an `admin` middleware:

    Route::prefix('admin')
        ->middleware(['auth', 'admin'])
        ->group(function () {

            // Admin routes

        });

Request flow:

    Request
       ↓
    auth
       ↓
    admin
       ↓
    Controller


## 22. You want to limit an API to 100 requests per minute. What would you use?

Use Rate Limiting / Throttle Middleware:

    Route::get('/products', ...)
        ->middleware('throttle:100,1');

This means:

    100 requests
          ↓
    per 1 minute


## 23. A user is authenticated but their account is disabled. How do you prevent access?

Create an `active-account` middleware:

    public function handle(Request $request, Closure $next)
    {
        if (!$request->user()->is_active) {
            return response()->json([
                'message' => 'Account disabled'
            ], 403);
        }

        return $next($request);
    }

Then use it with `auth`:

    Route::middleware([
        'auth',
        'active-account'
    ])->group(function () {

        // Protected routes

    });

Request flow:

    Request
       ↓
    auth
       ↓
    active-account
       ↓
    Controller


# ⭐ Top Questions To Focus On

If you don't have much time before the interview, focus on these:

1. What is Middleware?
2. Why do we use Middleware?
3. How do you create Middleware?
4. What does `$next($request)` do?
5. Can Middleware stop a Request?
6. Global Middleware vs Route Middleware
7. What are Middleware Groups?
8. How do Middleware Parameters work?
9. Explain Middleware execution order.
10. Middleware vs Controller vs Form Request.


# 🧠 Important Mental Model

Remember the request lifecycle:

    Client
      ↓
    HTTP Request
      ↓
    Middleware
      ↓
    Middleware
      ↓
    Validation
      ↓
    Controller
      ↓
    Service
      ↓
    Database
      ↓
    Response


# 🎯 The Main Idea

Middleware is a layer between the HTTP Request and your application logic.

It allows you to:

- Inspect the request.
- Modify the request.
- Reject the request.
- Authenticate users.
- Authorize users.
- Limit requests.
- Log requests.
- Modify the response.

The key concept:

    Request
       ↓
    Middleware
       ↓
    Controller
       ↓
    Response

If Middleware calls:

    return $next($request);

the request continues.

If Middleware returns a response:

    return response()->json([
        'message' => 'Forbidden'
    ], 403);

the request stops and the Controller is not executed.
