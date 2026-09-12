# Laravel Routing — أهم 10 أسئلة Interview

## 1. What is Routing in Laravel?

الـ Routing في Laravel هو المسؤول عن تحديد الـ URL والـ HTTP Method اللي هيروحوا لتنفيذ كود معين، سواء Closure أو Controller.

مثال:

```php
Route::get('/products', [ProductController::class, 'index']);
```

معناه إن:

```text
GET /products
```

هيشغل:

```php
ProductController@index
```

---

## 2. ما الفرق بين GET و POST و PUT و PATCH و DELETE في Routes؟

- `GET`: جلب البيانات.
- `POST`: إنشاء بيانات جديدة.
- `PUT`: تحديث Resource بالكامل.
- `PATCH`: تحديث جزء من Resource.
- `DELETE`: حذف Resource.

مثال CRUD:

```php
Route::get('/products', ...);
Route::post('/products', ...);
Route::put('/products/{product}', ...);
Route::patch('/products/{product}', ...);
Route::delete('/products/{product}', ...);
```

في الـ REST API بنستخدم الـ HTTP Methods للتعبير عن العملية المطلوبة بدل ما نعتمد على أسماء URLs فقط.

---

## 3. ما هو Route Parameter؟

هو قيمة متغيرة موجودة داخل الـ URL.

مثال:

```php
Route::get('/products/{id}', function ($id) {
    return $id;
});
```

لو طلبنا:

```text
/products/15
```

فـ `$id` هتكون:

```text
15
```

ممكن يكون عندنا أكثر من Parameter:

```php
Route::get('/users/{user}/posts/{post}', function ($user, $post) {
    return "$user - $post";
});
```

---

## 4. ما هو Optional Route Parameter؟

هو Parameter ممكن المستخدم يبعته أو لا.

بنستخدم `?`:

```php
Route::get('/user/{name?}', function ($name = 'Guest') {
    return $name;
});
```

الطلب:

```text
/user/Ahmed
```

يرجع:

```text
Ahmed
```

والطلب:

```text
/user
```

يرجع:

```text
Guest
```

لازم يكون للـ Optional Parameter قيمة افتراضية.

---

## 5. ما هي Named Routes؟ وما فائدتها؟

Named Route هو Route له اسم نستخدمه بدل كتابة الـ URL مباشرة.

```php
Route::get('/products', [ProductController::class, 'index'])
    ->name('products.index');
```

بعد كده نقدر نجيب الـ URL باستخدام:

```php
route('products.index');
```

وفي Blade:

```php
<a href="{{ route('products.index') }}">
    Products
</a>
```

### فائدتها

لو غيرت الـ URL من:

```text
/products
```

إلى:

```text
/store/products
```

مش محتاج تغير كل الـ Links طالما اسم الـ Route ما زال:

```text
products.index
```

---

## 6. ما هو Route Model Binding؟

هو Feature في Laravel بيخلي Laravel يجيب الـ Model تلقائياً بناءً على Parameter الموجود في الـ Route.

بدل:

```php
Route::get('/products/{id}', function ($id) {
    $product = Product::findOrFail($id);

    return $product;
});
```

نقدر نكتب:

```php
Route::get('/products/{product}', function (Product $product) {
    return $product;
});
```

لو المستخدم طلب:

```text
/products/15
```

Laravel هيحاول يجيب:

```php
Product::findOrFail(15);
```

ويحقنه في `$product`.

### ليه ده مفيد؟

- يقلل الكود.
- يخلي الـ Controller أنظف.
- يتعامل تلقائياً مع عدم وجود Model عن طريق 404 في الـ implicit binding.

---

## 7. ما هي Route Constraints؟

هي قواعد بنحدد بيها شكل الـ Parameter المسموح به.

مثلاً عايزين `id` يكون رقم فقط:

```php
Route::get('/products/{id}', function ($id) {
    return $id;
})->whereNumber('id');
```

أو باستخدام Regex:

```php
Route::get('/products/{id}', function ($id) {
    return $id;
})->where('id', '[0-9]+');
```

بالتالي:

```text
/products/15
```

يشتغل.

لكن:

```text
/products/abc
```

مش هيطابق الـ Route ده.

---

## 8. ما هي Route Groups؟ وما فائدتها؟

Route Group بيسمح لنا نطبق إعدادات مشتركة على مجموعة Routes.

مثلاً Prefix:

```php
Route::prefix('admin')->group(function () {

    Route::get('/products', ...);

    Route::get('/users', ...);

});
```

الـ URLs هتكون:

```text
/admin/products
/admin/users
```

ممكن كمان نستخدم Group مع Middleware:

```php
Route::middleware('auth')->group(function () {

    Route::get('/dashboard', ...);

    Route::get('/profile', ...);

});
```

يعني كل الـ Routes الموجودة داخل الـ Group لازم تعدي على `auth` Middleware.

---

## 9. ما هو Resource Routing؟

Laravel بيوفر `Route::resource()` لإنشاء Routes الخاصة بالـ CRUD بشكل تلقائي.

```php
Route::resource('products', ProductController::class);
```

ده بيعمل Routes للعمليات الأساسية:

| Method | URL | Controller Method |
|---|---|---|
| GET | `/products` | `index` |
| GET | `/products/create` | `create` |
| POST | `/products` | `store` |
| GET | `/products/{product}` | `show` |
| GET | `/products/{product}/edit` | `edit` |
| PUT/PATCH | `/products/{product}` | `update` |
| DELETE | `/products/{product}` | `destroy` |

### أهم نقطة في Interview

لو سألوك:

> Why use Resource Routes?

الإجابة:

لأنها بتوفر طريقة Convention-based لإنشاء CRUD Routes القياسية بدل كتابة كل Route بشكل منفصل.

---

## 10. كيف تعرف كل الـ Routes الموجودة في Laravel؟

باستخدام:

```bash
php artisan route:list
```

الأمر ده بيعرض معلومات مثل:

```text
Method
URI
Name
Action
Middleware
```

مثلاً ممكن تشوف:

```text
GET|HEAD   products
POST       products
GET|HEAD   products/{product}
PUT|PATCH  products/{product}
DELETE     products/{product}
```

### أسئلة Follow-up ممكن تتسأل

ممكن تستخدم Filters مع `route:list` حسب احتياجك، مثل:

```bash
php artisan route:list --path=products
```

ولو عايز تشوف Routes الخاصة بـ API:

```bash
php artisan route:list --path=api
```

---

# Bonus: سؤال مهم جداً

## ما الفرق بين Route Parameter و Query Parameter؟

### Route Parameter

جزء من الـ URL نفسه:

```text
/products/15
```

والـ Route:

```php
Route::get('/products/{id}', ...);
```

### Query Parameter

بييجي بعد `?`:

```text
/products?page=2&category=phones
```

والـ Route ممكن يكون:

```php
Route::get('/products', function (Request $request) {
    return $request->query('page');
});
```

هنا:

```text
page = 2
category = phones
```

### قاعدة سهلة تحفظها

```text
/products/15
         ↑
   Route Parameter

/products?page=2
          ↑
    Query Parameter
```

---

# Interview Cheat Sheet

احفظ الترتيب ده:

```text
Routing
  ↓
HTTP Methods
  ↓
Parameters
  ↓
Optional Parameters
  ↓
Named Routes
  ↓
Route Constraints
  ↓
Route Groups
  ↓
Middleware
  ↓
Route Model Binding
  ↓
Resource Routes
  ↓
route:list
```

لو فهمت الـ 10 دول كويس، هتكون مغطّي الجزء الأساسي من Laravel Routing في Junior Backend Interview.
