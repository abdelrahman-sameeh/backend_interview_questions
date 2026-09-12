# 🟢 المستوى الأول — Junior

> **ده اللي لازم تعرفه.** لو اتسألت في أي سؤال هنا ومعرفتش، دي علامة حمرا.
> الإجابات قصيرة عن قصد — ده المستوى المطلوب منك بالظبط، مش أكتر.
> **الهدف:** تجاوب في 30–60 ثانية بثقة، مش تحاضر 5 دقايق.

---

## 🔹 أساسيات PHP

### 1) إيه هو PHP؟ وليه لحد دلوقتي بيستخدم؟

لغة **Server-Side** بتشتغل على السيرفر، بتستقبل Request وترجّع Response (HTML أو JSON).

**ليه مستخدمة؟** أداء اتحسّن جدًا من PHP 7 و 8 · Ecosystem ضخم (Laravel, WordPress) · Hosting رخيص ومتوفر · نسبة كبيرة جدًا من مواقع النت شغالة بيها.

---

### 2) إيه الفرق بين PHP و JavaScript؟

- **PHP** → على السيرفر، بتتعامل مع الداتابيز والملفات.
- **JavaScript** → في المتصفح، بتتعامل مع الـ DOM وتفاعل المستخدم (وبـ Node.js تقدر تعمل Backend).

**لو حبيت تزوّد نقطة:** PHP كل Request بيبدأ من الصفر وبيموت بعده (Shared-Nothing)، عكس Node اللي بيفضل شغال.

---

### 3) Lifecycle بتاع PHP Request بيشتغل إزاي؟

```
المتصفح → Nginx/Apache → PHP-FPM → تنفيذ الكود → Response → تفريغ الذاكرة
```

1. المتصفح يبعت Request.
2. الـ Web Server يستقبله ويمرّره لـ PHP.
3. PHP تقرأ الملف وتحوّله لـ **Opcodes** (والـ OPcache بيخزّنها عشان مايعملش كده كل مرة).
4. الكود يتنفّذ (Routing → Controller → Database).
5. الناتج يترجع للمتصفح.
6. **كل الذاكرة تتفرّغ** — عشان كده بنستخدم Session أو DB لحفظ البيانات.

---

### 4) إيه الفرق بين GET و POST؟

| | GET | POST |
|---|---|---|
| البيانات فين | في الـ URL | في الـ Body |
| الحجم | محدود | كبير |
| يتحفظ في History/Cache | ✅ | ❌ |
| الاستخدام | بحث، فلترة، عرض | Login، إضافة، رفع ملف |

⚠️ **نقطة مهمة:** POST **مش أأمن** من GET! الاتنين بيتبعتوا نص عادي من غير HTTPS. **الأمان الحقيقي = HTTPS.**

---

### 5) إيه الفرق بين include و require؟

- `include` → لو الملف مش موجود: **Warning** والكود **يكمّل**.
- `require` → لو الملف مش موجود: **Fatal Error** والكود **يقف**.

**القاعدة:** الملف ضروري (Config, DB)؟ → `require`. اختياري (Sidebar)؟ → `include`.

---

### 6) إمتى تستخدم include_once و require_once؟

لما الملف فيه **تعريف Class أو Function** — عشان لو اتحمّل مرتين هيديك `Cannot redeclare` وده Fatal Error.

```php
require_once 'helpers.php';
require_once 'helpers.php'; // اتجاهل، مافيش مشكلة
```

**بس عمليًا:** في المشاريع الحديثة مابنستخدمهاش أصلًا — **Composer Autoloading** بيعمل ده لوحده.

---

### 7) إيه الفرق بين == و ===؟

- `==` → يقارن **القيمة بس** (وبيحوّل الأنواع تلقائيًا).
- `===` → يقارن **القيمة + النوع**.

```php
1 ==  "1"   // true
1 === "1"   // false
0 ==  false // true
```

**استخدم `===` دايمًا كافتراضي.** وأشهر مثال عملي:

```php
$pos = strpos('Hello', 'H'); // بترجع 0
if ($pos === false) { }  // ✅ صح
if ($pos == false)  { }  // ❌ غلط، لأن 0 == false
```

---

### 8) إيه الفرق بين echo و print؟

- `echo` → مابيرجّعش قيمة، وياخد أكتر من قيمة: `echo $a, $b;`
- `print` → بيرجّع دايمًا `1`، وياخد قيمة واحدة بس.

**عمليًا:** استخدم `echo`، وفي الـ Views استخدم `<?= $var ?>`.

---

### 9) إيه هو Session؟ وإيه الفرق بينه وبين Cookie؟

**Session** = بيانات المستخدم متخزنة **على السيرفر**، والمتصفح بيمسك بس **Session ID** في Cookie اسمها `PHPSESSID`.

| | Session | Cookie |
|---|---|---|
| مخزنة فين | السيرفر | المتصفح |
| الأمان | أعلى | أقل (المستخدم يقدر يعدّلها) |
| الحجم | كبير | ~4KB |
| الاستخدام | بيانات الدخول، السلة | "تذكّرني"، اللغة، الثيم |

```php
session_start();
$_SESSION['user_id'] = 42;
```

⚠️ **متحطّش أبدًا** حاجة زي `is_admin` في Cookie عادية.

---

## 🔹 التنظيم والأدوات

### 10) إيه هو Composer؟ وليه مهم؟

**مدير الاعتماديات (Dependencies)** بتاع PHP.

**بيعمل 3 حاجات:** يحمّل المكتبات · يظبط الإصدارات · يعمل **Autoloading** تلقائي.

```bash
composer require monolog/monolog
composer install    # في Production
```

**فرق مهم بيتسأل كتير:**
- `composer.json` = اللي **إنت طلبته** (نطاق إصدارات).
- `composer.lock` = اللي **اتثبّت فعلًا** (إصدارات محددة) ← **لازم يتعمله Commit** عشان كل البيئات تشتغل بنفس الكود.

---

### 11) إيه هو Autoloading؟

تحميل الـ Class **تلقائيًا** أول ما تحتاجه، بدل `require` يدوي لكل ملف.

```php
require 'vendor/autoload.php';
$user = new App\Models\User(); // PHP بتلاقي الملف لوحدها
```

بيشتغل بمعيار **PSR-4**: `App\Models\User` → `src/Models/User.php`

---

### 12) إيه هو Namespace؟

مساحة أسماء بتمنع **تعارض الأسماء** بين كودك والمكتبات.

```php
namespace App\Services;

use App\Models\User;

class Mailer {}   // الاسم الكامل: App\Services\Mailer
```

من غيره، لو عندك `Logger` والمكتبة عندها `Logger` → **Fatal Error**.

---

## 🔹 OOP

### 13) إيه الفرق بين Class و Object؟

- **Class** → التصميم/الخريطة. مش بياخد مكان في الذاكرة.
- **Object** → نسخة فعلية (instance) اتعملت من الكلاس. كل object ليه بياناته المستقلة.

```php
class Car { public string $color; }

$car1 = new Car();  $car1->color = 'red';
$car2 = new Car();  $car2->color = 'blue';   // مستقل تمامًا عن الأول
```

**تشبيه:** الكلاس = ورقة الرسم الهندسي للعربية، الـ Object = العربية اللي نزلت من المصنع.

---

### 14) إيه هي مبادئ الـ OOP الأربعة؟

| المبدأ | معناه في سطر |
|---|---|
| **Encapsulation** | أخفي البيانات وأتحكم في الوصول ليها |
| **Inheritance** | كلاس يرث خصائص وسلوك كلاس تاني |
| **Polymorphism** | نفس الميثود، تنفيذ مختلف حسب الكلاس |
| **Abstraction** | أعرض **إيه** بيعمل، وأخفي **إزاي** بيعمله |

⚠️ **الفرق اللي بيتلخبط فيه الناس:** الـ **Abstraction** فكرة تصميم (بتخفي التعقيد)، والـ **Encapsulation** تنفيذ (بتخفي البيانات نفسها بـ `private`).

---

### 15) إيه هو Encapsulation؟ وليه بنستخدمه؟ ⭐

إني أخلي بيانات الكلاس **`private`**، والتعديل عليها يعدّي إجباريًا من خلال methods بتتحكم في الشروط.

```php
class BankAccount {
    private float $balance = 0;

    public function deposit(float $amount): void {
        if ($amount <= 0) throw new InvalidArgumentException('مبلغ غير صالح');
        $this->balance += $amount;
    }

    public function getBalance(): float { return $this->balance; }
}

$acc->balance = -5000;   // ❌ Error — مستحيل الرصيد يبقى بالسالب
```

**الفايدة:** الـ Validation في مكان واحد · الـ object دايمًا في حالة صحيحة · تقدر تغيّر التنفيذ الداخلي من غير ما تكسر الكود اللي بيستخدم الكلاس.

⚠️ **فخ شائع في الإنترفيو:** لو عملت getter و setter **فاضيين** لكل property، ده encapsulation **شكلي** بس — عمليًا زي `public` بالظبط. الفكرة إنك تعرض **سلوك** (`withdraw()`) مش **بيانات** (`setBalance()`).

---

### 16) إيه الفرق بين Abstract Class و Interface؟

| | Abstract Class | Interface |
|---|---|---|
| فيه كود جاهز؟ | ✅ آه | ❌ لأ (تعريفات بس) |
| Properties | ✅ | ❌ (ثوابت بس) |
| الوراثة | واحدة بس | تقدر تطبّق **أكتر من واحدة** |
| العلاقة | **is-a** (علاقة قوية) | **can-do** (عقد/اتفاق) |

```php
abstract class Animal {
    public function sleep() { echo "Zzz"; }   // كود مشترك
    abstract public function makeSound();      // لازم الابن ينفّذها
}

interface Payable {
    public function pay(float $amount): bool;  // عقد بس
}
```

**القاعدة:** فيه كود مشترك؟ → Abstract. عايز تفرض عقد على كلاسات مختلفة؟ → Interface.

---

### 17) إيه الفرق بين Trait و Inheritance؟

- **Inheritance** → علاقة **is-a**، ووراثة **واحدة بس**.
- **Trait** → إعادة استخدام كود بين كلاسات **مالهاش علاقة ببعض**، وتقدر تستخدم **أكتر من Trait**.

```php
trait Loggable {
    public function log(string $msg) { echo $msg; }
}

class User    { use Loggable; }
class Product { use Loggable; }   // مافيش علاقة وراثة بينهم
```

الـ Trait بيحل مشكلة إن PHP مابتدعمش الوراثة المتعددة.

---

### 18) إيه الفرق بين public و private و protected؟ ⭐

| | من جوّه الكلاس | من الكلاس الوارث | من بره |
|---|---|---|---|
| `public` | ✅ | ✅ | ✅ |
| `protected` | ✅ | ✅ | ❌ |
| `private` | ✅ | ❌ | ❌ |

```php
class ParentOne {
    private   string $secret = 'a';
    protected string $shared = 'b';
}

class ChildOne extends ParentOne {
    public function test() {
        return $this->shared;   // ✅ شغال
        // return $this->secret; // ❌ Error
    }
}
```

**القاعدة العملية:** ابدأ بـ `private` دايمًا، وارفع الصلاحية بس لما تحتاج فعلًا.

---

### 19) إيه هو Polymorphism؟ اديني مثال

نفس الميثود بأسماء واحدة، لكن كل كلاس ينفّذها بطريقته — والكود اللي بينادي مايهموش هو بيتعامل مع مين.

```php
interface Shape { public function area(): float; }

class Circle implements Shape {
    public function __construct(private float $r) {}
    public function area(): float { return M_PI * $this->r ** 2; }
}

class Square implements Shape {
    public function __construct(private float $side) {}
    public function area(): float { return $this->side ** 2; }
}

foreach ([new Circle(2), new Square(3)] as $shape) {
    echo $shape->area();   // نفس النداء، نتيجة مختلفة
}
```

**الفايدة:** لما تضيف شكل جديد، مش هتعدّل الـ `foreach` — ده اللي بيخلي الكود قابل للتوسّع.

---

### 20) إيه الفرق بين `$this` و `self`؟

- `$this` → بتشاور على **الـ object الحالي** (تستخدمها مع الـ non-static).
- `self::` → بتشاور على **الكلاس نفسه** (تستخدمها مع `static` والـ constants).

```php
class Counter {
    public  int $id = 1;
    public static int $count = 0;
    const   TYPE = 'counter';

    public function show() {
        echo $this->id;        // property للـ object
        echo self::$count;     // property للكلاس كله (مشتركة)
        echo self::TYPE;       // constant
    }
}
```

⚠️ الـ `static` property بتبقى **مشتركة بين كل الـ objects** — لو غيّرتها من object، هتتغيّر عند الكل.

---

### 21) إيه هو Constructor؟ وإيه الـ Constructor Property Promotion؟

الـ `__construct()` ميثود بتشتغل **تلقائيًا** أول ما تعمل `new` — بتستخدمها عشان الـ object يتولد وهو جاهز وبياناته صحيحة.

```php
// الطريقة القديمة
class User {
    private string $name;
    public function __construct(string $name) { $this->name = $name; }
}

// PHP 8 — Property Promotion (نفس الكلام في سطر)
class User {
    public function __construct(private string $name) {}
}
```

فيه كمان `__destruct()` بتشتغل لما الـ object يتمسح من الذاكرة.

---

## 🔹 المعمارية والداتابيز

### 22) إيه هو MVC؟

نمط بيفصل المشروع لـ 3 طبقات:

- **Model** → البيانات ومنطق الشغل والتعامل مع DB.
- **View** → العرض (HTML / Blade).
- **Controller** → بيستقبل الـ Request، ينادي الـ Model، ويرجّع الـ View.

**الفايدة:** كل جزء مسؤوليته واضحة → أسهل في الصيانة والاختبار وشغل الفريق.

---

### 23) إيه هو ORM؟

**Object-Relational Mapping** — بيخلّيك تتعامل مع الداتابيز كـ **Objects** بدل SQL.

```php
// بدل: SELECT * FROM users WHERE id = 1
$user = User::find(1);
$user->name = 'Ahmed';
$user->save();
```

في Laravel الـ ORM اسمه **Eloquent**.

---

### 24) إيه الفرق بين Raw SQL و ORM؟

| | ORM | Raw SQL |
|---|---|---|
| السرعة في الكتابة | أسرع | أبطأ |
| الأداء | أبطأ شوية | أسرع |
| التحكم | محدود | كامل |
| الحماية من SQL Injection | تلقائي | لازم تعملها بنفسك |

**عمليًا:** استخدم ORM في 90% من الشغل، وانزل Raw SQL في التقارير المعقدة والاستعلامات الثقيلة.

---

### 25) إيه هو PDO؟ وليه أفضل من mysqli؟

**PDO** = PHP Data Objects، طبقة موحّدة للتعامل مع الداتابيز.

**ليه أفضل؟**
1. **بيدعم 12+ نوع داتابيز** (MySQL, PostgreSQL, SQLite) — mysqli لـ MySQL بس.
2. **Named Parameters** أوضح بكتير.
3. Object-oriented وبيرمي **Exceptions**.

```php
$stmt = $pdo->prepare("SELECT * FROM users WHERE email = :email");
$stmt->execute(['email' => $email]);
```

---

## 🔹 الأمان ⭐ (أهم قسم — ركّز فيه)

### 26) إزاي تمنع SQL Injection؟

**الحل الأساسي: Prepared Statements** — بتفصل الاستعلام عن البيانات، فالمُدخَل بيتعامل كـ **قيمة** مش كـ **كود**.

```php
// ❌ خطر
$pdo->query("SELECT * FROM users WHERE email = '$email'");

// ✅ آمن
$stmt = $pdo->prepare("SELECT * FROM users WHERE email = ?");
$stmt->execute([$email]);
```

**كمان:** استخدم ORM · اعمل Validation للمدخلات · مبدأ **Least Privilege** ليوزر الداتابيز.

---

### 27) إزاي تمنع XSS؟

**XSS** = المهاجم بيحقن JavaScript في صفحتك وتتنفّذ عند المستخدمين (وبيسرق بيها الـ Session).

**الحل: Escaping عند العرض.**

```php
echo htmlspecialchars($comment, ENT_QUOTES, 'UTF-8');
```

في **Blade**: `{{ $comment }}` بيعمل Escape تلقائيًا — و `{!! $comment !!}` **خطر** لأنه مابيعملهوش.

**كمان:** `HttpOnly` على الـ Cookies · **Content-Security-Policy** header.

---

### 28) إيه هو CSRF؟ وإزاي تمنعه؟

**CSRF** = موقع خبيث بيخلّي متصفح المستخدم يبعت Request لموقعك وهو مسجّل دخول، من غير علمه (مثلًا تحويل فلوس).

**الحل: CSRF Token** — قيمة عشوائية سرية في كل Form، السيرفر بيتحقق منها.

```blade
<form method="POST">
    @csrf
    ...
</form>
```

**كمان:** `SameSite=Strict` على الـ Cookies · استخدم POST للعمليات اللي بتغيّر بيانات.

---

### 29) إيه هو Password Hashing؟ وليه `password_hash()`؟

**Hashing** = تحويل الباسورد لنص مشفّر **باتجاه واحد** (مايتفكّش) — عشان لو الداتابيز اتسربت، الباسوردات تفضل محمية.

```php
$hash = password_hash($password, PASSWORD_DEFAULT);

if (password_verify($input, $hash)) {
    // صح
}
```

**ليه `password_hash()` تحديدًا؟**
- بيستخدم **bcrypt/Argon2** (بطيئة عن قصد ← بتصعّب الـ Brute Force).
- بيضيف **Salt عشوائي تلقائيًا** ← نفس الباسورد بيدي Hash مختلف كل مرة.
- بيتحدّث تلقائيًا لأقوى خوارزمية متاحة.

⚠️ **متستخدمش `md5()` أو `sha1()` أبدًا** — سريعة جدًا وبتتكسر بسهولة.

---

### 30) إيه الفرق بين Authentication و Authorization؟

- **Authentication (المصادقة)** → **مين إنت؟** (Login بإيميل وباسورد).
- **Authorization (التصريح)** → **مسموحلك تعمل إيه؟** (هل تقدر تمسح البوست ده؟).

**الترتيب:** Authentication الأول، بعدها Authorization.
**في Laravel:** Sanctum/Passport للأولى، Policies/Gates للتانية.

---

## 🔹 APIs

### 31) إيه هو REST API؟

أسلوب لتصميم APIs بيعتمد على **Resources** و **HTTP Methods**.

| Method | العملية | مثال |
|---|---|---|
| GET | قراءة | `GET /users` |
| POST | إنشاء | `POST /users` |
| PUT/PATCH | تعديل | `PATCH /users/1` |
| DELETE | حذف | `DELETE /users/1` |

**أهم مبادئه:** **Stateless** (السيرفر مابيحفظش حالة بين الطلبات) · Resource-based URLs (أسماء مش أفعال) · استخدام صحيح للـ Status Codes.

---

### 32) إيه هي HTTP Status Codes المهمة؟

| الكود | المعنى | إمتى |
|---|---|---|
| **200** | OK | نجاح عادي |
| **201** | Created | تم إنشاء مورد جديد |
| **204** | No Content | نجاح بدون بيانات (Delete) |
| **301 / 302** | Redirect | دائم / مؤقت |
| **400** | Bad Request | بيانات ناقصة أو غلط |
| **401** | Unauthorized | **مش مسجّل دخول** |
| **403** | Forbidden | **مسجّل دخول بس مش مصرّح له** |
| **404** | Not Found | مش موجود |
| **422** | Unprocessable | فشل الـ Validation |
| **429** | Too Many Requests | تخطّى Rate Limit |
| **500** | Server Error | خطأ في السيرفر |

⚠️ **الفرق بين 401 و 403** بيتسأل كتير جدًا.

---

### 33) إيه الفرق بين PUT و PATCH؟

- **PUT** → **استبدال كامل** للمورد (لازم تبعت كل الحقول).
- **PATCH** → **تعديل جزئي** (تبعت اللي عايز تغيّره بس).

```
PUT   /users/1  →  {"name":"Ahmed", "email":"a@b.com", "age":25}   // كله
PATCH /users/1  →  {"name":"Ahmed"}                                 // الاسم بس
```

**PUT** بيعتبر Idempotent (تكراره بيدي نفس النتيجة)، **PATCH** مش بالضرورة.

---

## 🔹 مفاهيم عامة

### 34) إيه هو Caching؟

تخزين نتيجة عملية **مكلفة** مؤقتًا عشان مانعيدهاش كل مرة → سرعة أعلى وضغط أقل على الداتابيز.

```php
$users = Cache::remember('users', 3600, function () {
    return User::all();   // بينفّذ الاستعلام أول مرة بس
});
```

**أنواعه:** Database Query Cache · Full Page Cache · Object Cache (Redis) · OPcache (كاش الـ Opcodes).

**أصعب جزء فيه:** **Cache Invalidation** — إمتى تمسح الكاش عشان المستخدم مايشوفش بيانات قديمة.

---

### 35) إيه الفرق بين Unit Testing و Integration Testing؟

| | Unit Test | Integration Test |
|---|---|---|
| بيختبر إيه | وحدة واحدة معزولة (دالة/كلاس) | أكتر من جزء مع بعض |
| الاعتماديات | **Mocked** (مزيّفة) | حقيقية (DB, API) |
| السرعة | سريع جدًا | أبطأ |
| مثال | دالة حساب الخصم | Endpoint التسجيل كامل مع DB |

**Unit** بيقولك: الجزء ده شغال صح؟
**Integration** بيقولك: الأجزاء بتشتغل مع بعض صح؟

---

# ✅ Checklist قبل الإنترفيو

- [ ] الفرق بين `==` و `===` + حالة `strpos() === false`
- [ ] Session vs Cookie
- [ ] `include` vs `require`
- [ ] Abstract vs Interface (**أشهر سؤال OOP**)
- [ ] Trait ليه موجود
- [ ] MVC — تشرحه في 3 جمل
- [ ] **Prepared Statements** ضد SQL Injection ⭐
- [ ] `htmlspecialchars` ضد XSS ⭐
- [ ] CSRF Token ⭐
- [ ] `password_hash` مش `md5` ⭐
- [ ] Authentication vs Authorization
- [ ] الفرق بين 401 و 403 ⭐
- [ ] PUT vs PATCH
- [ ] `composer.json` vs `composer.lock`

> **آخر نصيحة:** الـ 4 نجوم ⭐ (الأمان) هي اللي بتفرق فعلًا. مطوّر Junior بيعرف يحمي كوده = مطوّر بيتوظّف.
