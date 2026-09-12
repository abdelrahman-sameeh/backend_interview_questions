# 🗺️ Roadmap — PHP & OOP لمستوى Junior قوي

> **الهدف:** توصل لمستوى تقدر تتوظّف بيه كـ Junior Backend Developer.
> **المدة:** 8 أسابيع بمعدل 2–3 ساعات يوميًا.
> **القاعدة الذهبية:** كل موضوع تذاكره → **اكتبه بإيدك** في كود. القراءة لوحدها = صفر.

---

## ⚖️ توزيع الوقت (مهم تفهمه الأول)

```
40%  ← كتابة كود ومشاريع        ⭐ الأهم
25%  ← OOP وتصميم الكود
20%  ← الداتابيز و SQL
10%  ← الأمان
 5%  ← أدوات (Git, Composer)
```

**الخطأ الأشهر عند المبتدئين:** يقضي 90% من وقته قراءة ومشاهدة فيديوهات، و10% كتابة كود. اعكس النسبة دي.

---

# 📅 المرحلة 1 — أساسيات PHP الصلبة
**⏱ أسبوعان**

## الأسبوع 1 — اللغة نفسها

### أ) الأساسيات
- [ ] Variables و **Data Types** (`int`, `float`, `string`, `bool`, `array`, `null`, `object`)
- [ ] **Type Juggling** والفرق بين `==` و `===`
- [ ] Operators (حسابية، منطقية، مقارنة، **Null Coalescing `??`**، **Spaceship `<=>`**)
- [ ] Control Structures: `if`, `switch`, **`match`** (PHP 8)
- [ ] Loops: `for`, `foreach`, `while`, و `break` / `continue`

### ب) الدوال (Functions) ⭐
- [ ] تعريف الدوال والـ Parameters والـ Return
- [ ] **Type Declarations** و `declare(strict_types=1)` ← اتعوّد عليها من البداية
- [ ] Default Parameters و **Named Arguments**
- [ ] **Variable Scope** و `global` و `static`
- [ ] **Anonymous Functions** و `use` keyword
- [ ] **Arrow Functions** `fn($x) => $x * 2`
- [ ] **Callbacks** و `callable`
- [ ] Variadic Functions `...$args`

### ج) المصفوفات (Arrays) ⭐⭐
دي أكتر حاجة هتستخدمها يوميًا:

- [ ] Indexed vs Associative vs **Multidimensional**
- [ ] **دوال لازم تحفظها:**

| المجموعة | الدوال |
|---|---|
| التحويل | `array_map` · `array_filter` · `array_reduce` |
| البحث | `in_array` · `array_search` · `array_key_exists` · `isset` |
| الترتيب | `sort` · `usort` · `ksort` · `asort` |
| الدمج | `array_merge` · `array_combine` · `array_diff` · `array_intersect` |
| الاستخراج | `array_column` · `array_keys` · `array_values` · `array_slice` |
| التجميع | `array_chunk` · `count` · `array_sum` · `array_unique` |

```php
// تمرين: افهم الفرق ده كويس
$users = [
    ['name' => 'Ahmed', 'age' => 25, 'active' => true],
    ['name' => 'Sara',  'age' => 30, 'active' => false],
    ['name' => 'Omar',  'age' => 22, 'active' => true],
];

$activeNames = array_column(
    array_filter($users, fn($u) => $u['active']),
    'name'
);
// ['Ahmed', 'Omar']
```

### د) النصوص (Strings)
- [ ] `sprintf` · `str_replace` · `substr` · `strpos` · `explode` / `implode`
- [ ] `trim` · `str_pad` · `str_contains` · `str_starts_with` (PHP 8)
- [ ] **Heredoc / Nowdoc**
- [ ] الفرق بين `'single'` و `"double"` quotes
- [ ] أساسيات **Regex** (`preg_match`, `preg_replace`) — مش لازم تحترفها

---

## الأسبوع 2 — PHP في سياق الويب

- [ ] **Superglobals:** `$_GET` · `$_POST` · `$_SERVER` · `$_SESSION` · `$_COOKIE` · `$_FILES`
- [ ] Forms والتعامل معها
- [ ] **Validation و Sanitization** (`filter_var`, `filter_input`)
- [ ] Sessions و Cookies عمليًا
- [ ] File Handling (`fopen`, `file_get_contents`, `file_put_contents`)
- [ ] **File Upload** بشكل آمن
- [ ] JSON (`json_encode`, `json_decode` + الـ Flags)
- [ ] Date & Time (**`DateTime`** و `DateTimeImmutable` — مش `date()` بس)
- [ ] **Error Handling:** `try/catch/finally` · `throw` · هرم الـ Exceptions
- [ ] `include` / `require` / `_once`
- [ ] `php.ini` وإعداداته الأساسية

```php
// تمرين مهم: Custom Exceptions
class InsufficientBalanceException extends RuntimeException {}

try {
    $account->withdraw(5000);
} catch (InsufficientBalanceException $e) {
    echo "الرصيد غير كافٍ: " . $e->getMessage();
} catch (Throwable $e) {
    logError($e);
} finally {
    $connection->close();
}
```

### 🎯 مشروع المرحلة 1
> **Contact Manager** — إضافة/عرض/تعديل/حذف جهات اتصال، مخزّنة في ملف JSON.
> بدون داتابيز وبدون OOP. الهدف: تتقن الأساسيات.

---

# 📅 المرحلة 2 — OOP ⭐⭐⭐
**⏱ أسبوعان — أهم مرحلة على الإطلاق**

> 90% من أسئلة الإنترفيو التقنية بتيجي من هنا. متعدّيش المرحلة دي وإنت مش مرتاح فيها.

## الأسبوع 3 — أساسيات OOP

### أ) المفاهيم الأساسية
- [ ] **Class** و **Object** و `new` و `$this`
- [ ] **Properties** و **Methods**
- [ ] **Constructor** `__construct` و **Destructor** `__destruct`
- [ ] **Constructor Property Promotion** (PHP 8) ⭐
```php
// الطريقة القديمة
class User {
    private string $name;
    public function __construct(string $name) {
        $this->name = $name;
    }
}

// PHP 8 — نفس الحاجة في سطر
class User {
    public function __construct(private string $name) {}
}
```

### ب) الأركان الأربعة للـ OOP ⭐ (سؤال إنترفيو مضمون)

**1. Encapsulation (التغليف)**
- [ ] `public` / `private` / `protected`
- [ ] Getters و Setters وإمتى تحتاجهم فعلًا
- [ ] `readonly` properties (PHP 8.1)

**2. Inheritance (الوراثة)**
- [ ] `extends` و `parent::`
- [ ] **Method Overriding**
- [ ] `final` classes و methods
- [ ] ⚠️ **متسيئش استخدامها** — "Composition over Inheritance"

**3. Polymorphism (تعدد الأشكال)**
- [ ] نفس الـ Method باستجابة مختلفة حسب الكلاس
```php
interface Shape { public function area(): float; }

class Circle implements Shape {
    public function __construct(private float $r) {}
    public function area(): float { return M_PI * $this->r ** 2; }
}

class Square implements Shape {
    public function __construct(private float $s) {}
    public function area(): float { return $this->s ** 2; }
}

// نفس الكود بيتعامل مع الاتنين
foreach ([new Circle(2), new Square(3)] as $shape) {
    echo $shape->area();
}
```

**4. Abstraction (التجريد)**
- [ ] **Abstract Classes** و **Abstract Methods**
- [ ] **Interfaces**
- [ ] **الفرق بينهم** ⭐⭐ (أشهر سؤال OOP في الدنيا)

### ج) مفاهيم إضافية
- [ ] **Static** properties و methods و `self::` و `static::`
- [ ] **Class Constants** و `const`
- [ ] **Enums** (PHP 8.1) — Pure و Backed
- [ ] **Traits** وحل تعارض الأسماء (`insteadof`, `as`)
- [ ] **Namespaces** و `use`
- [ ] **Magic Methods:** `__get` · `__set` · `__call` · `__toString` · `__invoke`
- [ ] **Late Static Binding** (`self` vs `static`)

---

## الأسبوع 4 — تصميم الكود

### أ) SPL و Interfaces مدمجة
- [ ] `Countable` · `Iterator` · `IteratorAggregate` · `ArrayAccess` · `JsonSerializable`
- [ ] `Stringable` · `Throwable`
- [ ] `ArrayObject` · `SplStack` · `SplQueue`

### ب) مبادئ التصميم ⭐
- [ ] **SOLID** — ركّز على **S** و **D** (دول اللي بيتسألوا عليهم أكتر)
- [ ] **DRY** (Don't Repeat Yourself)
- [ ] **KISS** (Keep It Simple)
- [ ] **YAGNI** (You Aren't Gonna Need It)
- [ ] **Composition over Inheritance** ⭐
- [ ] **Dependency Injection** (الفكرة والتطبيق)

### ج) أنماط بسيطة تعرفها
- [ ] **Singleton** (واعرف ليه بيعتبروه Anti-pattern أحيانًا)
- [ ] **Factory**
- [ ] **Strategy** ← الأكثر فائدة عمليًا
- [ ] **Repository** (الفكرة بس)

### د) MVC
- [ ] الفكرة والفصل بين الطبقات
- [ ] تطبيقها يدويًا في مشروع صغير قبل ما تدخل Laravel

### 🎯 مشروع المرحلة 2
> **نفس الـ Contact Manager لكن بـ OOP كامل** — Classes، Interfaces، Namespaces، Custom Exceptions، وتنظيم MVC يدوي.
> قارن بين النسختين، وهتفهم قيمة الـ OOP بنفسك.

---

# 📅 المرحلة 3 — الداتابيز
**⏱ أسبوع واحد**

### أ) SQL ⭐ (مهم جدًا وبيتسأل كتير)
- [ ] `SELECT` · `WHERE` · `ORDER BY` · `LIMIT` · `OFFSET`
- [ ] `INSERT` · `UPDATE` · `DELETE`
- [ ] **JOINs:** `INNER` · `LEFT` · `RIGHT` ⭐
- [ ] `GROUP BY` · `HAVING` · Aggregate functions (`COUNT`, `SUM`, `AVG`, `MAX`)
- [ ] **Subqueries**
- [ ] **Indexes** — إيه هي وليه مهمة للأداء ⭐
- [ ] **Transactions** (`BEGIN`, `COMMIT`, `ROLLBACK`) و **ACID**
- [ ] Foreign Keys و **Cascade**
- [ ] **Normalization** (1NF, 2NF, 3NF)
- [ ] العلاقات: One-to-One · One-to-Many · **Many-to-Many** (جدول وسيط)

### ب) PDO ⭐
- [ ] الاتصال وإعدادات الـ Error Mode
- [ ] **Prepared Statements** (الحماية من SQL Injection)
- [ ] Named vs Positional Parameters
- [ ] `fetch` · `fetchAll` · `fetchColumn` و **Fetch Modes**
- [ ] Transactions في PDO
- [ ] `lastInsertId()`

```php
$pdo = new PDO($dsn, $user, $pass, [
    PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    PDO::ATTR_EMULATE_PREPARES   => false,   // مهم للأمان الحقيقي
]);

$stmt = $pdo->prepare("SELECT * FROM users WHERE status = :status");
$stmt->execute(['status' => 'active']);
$users = $stmt->fetchAll();
```

### 🎯 مشروع المرحلة 3
> **Blog بسيط** — Users, Posts, Comments, Categories.
> علاقات كاملة + PDO + Prepared Statements + Transactions.

---

# 📅 المرحلة 4 — الأمان ⭐
**⏱ 3–4 أيام**

> ده اللي بيفرّق Junior محترم عن Junior عادي. **متعدّيهاش.**

- [ ] **SQL Injection** ← Prepared Statements
- [ ] **XSS** ← `htmlspecialchars()` عند العرض
- [ ] **CSRF** ← Tokens + `SameSite`
- [ ] **Password Hashing** ← `password_hash` / `password_verify` (**مش `md5`**)
- [ ] **Session Security** ← `HttpOnly` · `Secure` · `session_regenerate_id`
- [ ] **File Upload Security** ← Whitelist + تحقق من المحتوى + اسم عشوائي
- [ ] **Input Validation** ← تحقق دايمًا على السيرفر، مش الـ JS بس
- [ ] **Authentication vs Authorization**
- [ ] متحطّش أسرار في الكود ← `.env`
- [ ] `APP_DEBUG=false` في Production

### 🎯 تمرين
> ارجع لمشروع الـ Blog واعمله **Security Audit** — دوّر على كل ثغرة من دول واقفلها.

---

# 📅 المرحلة 5 — الأدوات و APIs
**⏱ أسبوع واحد**

### أ) Git ⭐ (لا غنى عنه)
- [ ] `init` · `add` · `commit` · `status` · `log` · `diff`
- [ ] **Branching:** `branch` · `checkout` · `merge`
- [ ] `push` · `pull` · `clone` · `remote`
- [ ] حل الـ **Conflicts**
- [ ] `.gitignore`
- [ ] **Pull Requests** على GitHub
- [ ] رسائل Commit محترمة

### ب) Composer
- [ ] `composer.json` vs `composer.lock` ⭐
- [ ] `require` · `install` · `update` · `dump-autoload`
- [ ] **PSR-4 Autoloading**
- [ ] Semantic Versioning (`^` و `~`)

### ج) REST APIs
- [ ] مبادئ REST
- [ ] **HTTP Methods** و **Status Codes** ⭐
- [ ] Headers و `Content-Type`
- [ ] JSON Request/Response
- [ ] بناء API بسيط بنفسك
- [ ] استهلاك API خارجي (**Guzzle** أو cURL)
- [ ] **Postman** — لازم تعرف تستخدمه

### د) بيئة العمل
- [ ] Linux basics (`cd`, `ls`, `chmod`, `grep`, `nano`)
- [ ] أساسيات Nginx أو Apache
- [ ] **Docker** — مستوى `docker compose up` كفاية دلوقتي
- [ ] IDE محترم (**PhpStorm** أو VS Code + إضافات PHP)

---

# 📅 المرحلة 6 — Laravel
**⏱ أسبوعان**

> ⚠️ **متدخلش هنا قبل ما تخلص OOP.** Laravel كله OOP — لو أساسك ضعيف، هتحفظ من غير ما تفهم، وده أسوأ حاجة.

### الترتيب الصحيح للتعلّم:
1. [ ] **Installation** وهيكل المشروع
2. [ ] **Routing** (Basic, Parameters, Named, Groups)
3. [ ] **Controllers** و **Resource Controllers**
4. [ ] **Blade** (Layouts, Components, Directives)
5. [ ] **Migrations** و **Seeders** و **Factories**
6. [ ] **Eloquent ORM** ⭐⭐
   - CRUD · Relationships · Scopes · Accessors/Mutators
   - **N+1 Problem** و **Eager Loading** ← بيتسأل كتير
7. [ ] **Validation** و **Form Requests**
8. [ ] **Middleware**
9. [ ] **Authentication** (Breeze / Sanctum)
10. [ ] **Authorization** (Gates & Policies)
11. [ ] **API Resources**
12. [ ] **Service Container** و **Service Providers**
13. [ ] **Events & Listeners**
14. [ ] **Queues & Jobs**
15. [ ] **Testing** (Feature + Unit)

### 🎯 مشروع التخرّج
> **Task Management API** أو **E-commerce Backend**
> - Authentication بـ Sanctum
> - Roles & Permissions
> - CRUD كامل مع علاقات
> - API Resources
> - Validation عبر Form Requests
> - Queue لإرسال الإيميلات
> - Tests
> - **README محترم** + Postman Collection
> - **Docker Compose** للتشغيل

---

# 🏆 الملخص: خطة الـ 8 أسابيع

| الأسبوع | الموضوع | المخرَج |
|---|---|---|
| **1** | أساسيات PHP | تمارين + مسائل |
| **2** | PHP للويب | Contact Manager (Procedural) |
| **3** | OOP الأساسيات ⭐ | تمارين Classes |
| **4** | OOP + تصميم ⭐ | Contact Manager (OOP + MVC) |
| **5** | SQL + PDO | Blog بعلاقات كاملة |
| **6** | الأمان + Git + APIs | Security Audit + REST API |
| **7** | Laravel أساسيات | CRUD كامل |
| **8** | Laravel متقدم | **مشروع التخرّج** |

---

# ✅ معايير "أنا جاهز للتقديم"

اقرأ الليستة دي بصدق — لو 80% منها ✅ يبقى قدّم:

**PHP**
- [ ] بكتب كود PHP من غير ما أبص على Google كل 5 دقايق
- [ ] بفهم رسائل الأخطاء وبعرف أصلّحها
- [ ] بستخدم `array_map` / `array_filter` بارتياح
- [ ] بستخدم Type Hints و `strict_types` بشكل طبيعي

**OOP** ⭐
- [ ] أشرح الأركان الأربعة **بأمثلة من كودي أنا**
- [ ] أعرف **إمتى** Interface و**إمتى** Abstract Class
- [ ] أستخدم Interfaces عشان أفصل الاعتماديات
- [ ] أشرح Dependency Injection وأطبّقه
- [ ] أشرح `S` و `D` من SOLID بمثال

**Database**
- [ ] أكتب JOIN بين 3 جداول
- [ ] أصمّم Schema لفكرة بسيطة من الصفر
- [ ] أستخدم Transactions صح
- [ ] أعرف يعني إيه Index وإمتى أحتاجه

**الأمان** ⭐
- [ ] كل استعلاماتي Prepared Statements
- [ ] بعمل Escape لكل مخرجات المستخدم
- [ ] بستخدم `password_hash` مش `md5`
- [ ] أعرف يعني إيه CSRF وإزاي أمنعه

**العملي**
- [ ] عندي **2–3 مشاريع على GitHub** بـ README محترم
- [ ] بستخدم Git يوميًا وبفهم Branching
- [ ] أعرف أبني REST API وأختبره بـ Postman
- [ ] **أقدر أشرح أي قرار في مشروعي وليه اخترته** ⭐⭐

---

# 🚫 أخطاء تجنّبها

| الخطأ | الصح |
|---|---|
| مشاهدة كورسات بلا كتابة كود | **اكتب كل سطر بإيدك** |
| القفز لـ Laravel قبل OOP | أساس OOP الأول |
| تجاهل SQL والاعتماد على Eloquent | اتعلّم SQL خام |
| نسخ الكود من ChatGPT/Stack Overflow بدون فهم | افهم كل سطر قبل ما تستخدمه |
| تجاهل Git | استخدمه من أول يوم |
| مشاريع Todo App بس | اعمل حاجة فيها منطق حقيقي |
| القفز بين 10 مصادر | مصدر واحد لحد ما تخلّصه |
| "لسه مش جاهز" لسنة | قدّم وإنت 70% جاهز — الإنترفيوهات نفسها تعليم |

---

# 📚 مصادر مقترحة

**عربي**
- قناة **علي عبد الرحمن (Codezilla)** — PHP و Laravel
- **Elzero Web School** — أساسيات ممتازة
- **Almabrouk / Codezilla** لـ OOP

**إنجليزي**
- **PHP The Right Way** (phptherightway.com) — مرجع مجاني ممتاز ⭐
- **Laracasts** — "PHP for Beginners" و "Object-Oriented Bootcamp" ⭐⭐
- **Laravel Docs** — من أحسن الوثائق في عالم البرمجة
- **Refactoring Guru** — Design Patterns و SOLID بأمثلة PHP

**تمارين**
- **Exercism.io** (مسار PHP) — تمارين مع مراجعة
- **Codewars** — مسائل يومية
- **LeetCode Easy** — للـ Logic

---

# 🎁 نصيحة أخيرة

> المشروع الحقيقي بيعلّمك أكتر من 10 كورسات.
>
> اختار فكرة **تهمّك إنت** (نظام لمذاكرتك، تتبّع مصاريفك، مكتبة كتب) واعملها بالكامل — من التصميم للداتابيز للنشر.
>
> ولما تروح الإنترفيو، **المشروع ده هو اللي هتتكلم عنه**، مش الكورسات اللي خلّصتها.

**كمان:**
- اكتب **README** لكل مشروع (المُحاوِر بيقراه).
- شارك تقدّمك على LinkedIn — بيفتح أبواب مش متوقعة.
- ساهم في مشروع Open Source صغير — ولو بإصلاح خطأ إملائي في التوثيق.
- **اتمرّن تشرح كودك بصوت عالي** — 50% من الإنترفيو شرح مش كتابة.
