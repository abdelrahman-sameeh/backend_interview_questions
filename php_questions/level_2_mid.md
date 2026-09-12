# 🟡 المستوى الثاني — Mid-Level

> **ده اللي بيفرق بين Junior و Mid.**
> كـ Junior: اعرف **الفكرة العامة** ومتى تُستخدم — كفاية.
> كـ Mid: لازم تكون **اشتغلت بيها فعلًا** وتقدر تشرح قرارك.
>
> لو اتسألت في حاجة هنا وإنت Junior، إجابة زي *"الفكرة إنه كذا، بس مااشتغلتش عليه عمليًا"* **إجابة محترمة** — أحسن بكتير من إنك تخترع.

---

## 🔹 مبادئ التصميم

### 1) إيه هو Dependency Injection؟

بدل ما الكلاس **يصنع** اعتمادياته بنفسه، بنمررهاله من بره.

```php
// ❌ Tight Coupling — مستحيل تختبره أو تغيّر الـ Mailer
class OrderService {
    private $mailer;
    public function __construct() {
        $this->mailer = new SmtpMailer();   // مربوط بكلاس معيّن
    }
}

// ✅ Dependency Injection
class OrderService {
    public function __construct(private MailerInterface $mailer) {}
}
```

**الفوايد:**
- **قابلية الاختبار** ← تقدر تمرّر Mock في الـ Tests.
- **مرونة** ← تبدّل التنفيذ من غير ما تلمس الكلاس.
- **Loose Coupling** ← الكلاس معتمد على **Interface** مش على تنفيذ معيّن.

**3 أنواع:** Constructor Injection (الأفضل والأشهر) · Setter Injection · Method Injection.

**في Laravel:** الـ Container بيعمل ده تلقائيًا — تكتب الـ Type Hint وهو يبعتلك الكائن.

---

### 2) إيه هي SOLID Principles؟

خمس مبادئ لكتابة كود قابل للصيانة والتوسّع.

**S — Single Responsibility**
> للكلاس **سبب واحد بس** للتغيير.

```php
// ❌ الكلاس ده بيعمل 3 حاجات
class User {
    public function save() {}
    public function sendEmail() {}
    public function generatePdf() {}
}
// ✅ UserRepository, UserMailer, UserPdfExporter
```

**O — Open/Closed**
> مفتوح للتوسّع، مقفول للتعديل.

بدل `switch` على أنواع الدفع، اعمل `PaymentInterface` وأضف كلاس جديد لكل طريقة — من غير ما تلمس الكود القديم.

**L — Liskov Substitution**
> أي كلاس ابن لازم يقدر يحل محل الأب من غير ما يكسر البرنامج.

```php
// ❌ كسر لـ Liskov
class Penguin extends Bird {
    public function fly() { throw new Exception("مابطيرش!"); }
}
```

**I — Interface Segregation**
> Interfaces صغيرة متخصصة أحسن من واحد ضخم.

متجبرش كلاس ينفّذ Methods هو مش محتاجها.

**D — Dependency Inversion**
> اعتمد على **Abstractions** مش على **Implementations**.

```php
public function __construct(private PaymentInterface $gateway) {}
// مش: private StripeGateway $gateway
```

⚠️ **نصيحة للإنترفيو:** لو مش فاكر الخمسة، اشرح **S** و **D** كويس — دول الأكتر استخدامًا عمليًا.

---

### 3) إيه هو PSR؟ وليه بنلتزم بيه؟

**PHP Standard Recommendation** — معايير من **PHP-FIG** عشان المكتبات والأطر تشتغل مع بعض.

| PSR | الوظيفة |
|---|---|
| **PSR-1 / PSR-12** | معايير كتابة الكود (Naming, Style) |
| **PSR-3** | واجهة موحّدة للـ Logging |
| **PSR-4** | Autoloading (Namespace ↔ مسار المجلد) |
| **PSR-7 / PSR-15** | HTTP Messages / Middleware |
| **PSR-11** | Container Interface |

**السبب الأهم — Interoperability:**

```php
// بيشتغل مع Monolog أو أي Logger يطبّق PSR-3
public function __construct(private \Psr\Log\LoggerInterface $logger) {}
```

تقدر تبدّل المكتبة من غير ما تغيّر سطر في كودك.

**الأدوات:** `php-cs-fixer` · `phpcs` · **Laravel Pint**.

---

## 🔹 الأمان المتقدم

### 4) إزاي تمنع Session Hijacking؟

المهاجم بيسرق الـ Session ID ويتقمّص شخصية الضحية من غير ما يعرف الباسورد.

**1) إعدادات Cookie آمنة**
```php
session_set_cookie_params([
    'secure'   => true,      // HTTPS بس
    'httponly' => true,      // JS مايقدرش يقراها → حماية من XSS
    'samesite' => 'Strict',  // حماية من CSRF
]);
session_start();
```

**2) تجديد الـ ID عند تسجيل الدخول** (يمنع Session Fixation)
```php
session_regenerate_id(true);
```

**3) إعدادات `php.ini`**
```ini
session.use_only_cookies = 1
session.use_strict_mode  = 1   ; يرفض أي Session ID مش السيرفر اللي ولّده
```

**4) Session Timeout** (Idle + Absolute)

**5) Fingerprinting** — اربط الجلسة بـ User-Agent (⚠️ **مش** بالـ IP وحده، لأنه بيتغير على الموبايل).

**6) امنع XSS أصلًا** — لأنه أشهر طريق لسرقة الـ Session.

---

### 5) إيه هو File Upload Security؟

**رفع الملفات من أخطر النقاط في أي تطبيق** — لأن ملف PHP مرفوع = سيطرة كاملة على السيرفر.

| الخطر | الحماية |
|---|---|
| رفع `shell.php` | **Whitelist** للامتدادات (مش Blacklist) |
| تزوير الـ MIME من العميل | تحقّق من السيرفر بـ `finfo` |
| `image.php.jpg` أو Null byte | **ولّد اسم عشوائي جديد** للملف |
| ملف ضخم يستهلك القرص | حدّد `upload_max_filesize` |
| Path Traversal (`../../`) | `basename()` + مسار ثابت |
| تنفيذ الملف | **خزّنه بره `public/`** أو امنع التنفيذ في مجلد الرفع |

```php
$allowed = ['jpg' => 'image/jpeg', 'png' => 'image/png'];

$finfo = new finfo(FILEINFO_MIME_TYPE);
$mime  = $finfo->file($_FILES['photo']['tmp_name']);   // من المحتوى مش من العميل

if (!in_array($mime, $allowed, true)) {
    throw new RuntimeException('نوع ملف غير مسموح');
}

$name = bin2hex(random_bytes(16)) . '.' . array_search($mime, $allowed, true);
move_uploaded_file($_FILES['photo']['tmp_name'], STORAGE_PATH . '/' . $name);
```

**كمان:** امنع تنفيذ PHP في مجلد الرفع من إعدادات Nginx، وافحص الصور بإعادة معالجتها (Re-encoding).

---

## 🔹 المصادقة والـ APIs

### 6) إزاي تطبق JWT Authentication؟

**JWT** = JSON Web Token، توكن **موقّع** بيحمل بيانات المستخدم، والسيرفر مابيخزّنش حاجة (**Stateless**).

**تركيبه:** `Header.Payload.Signature`

```
eyJhbGciOiJIUzI1NiJ9 . eyJzdWIiOiI0MiIsImV4cCI6MTcwMH0 . dBjftJeZ4CVP...
   الخوارزمية              البيانات (sub, exp)              التوقيع
```

**الفلو:**
1. المستخدم يبعت الإيميل والباسورد.
2. السيرفر يتحقق ويولّد Token موقّع بـ Secret Key.
3. العميل يخزّنه ويبعته في كل Request:
   `Authorization: Bearer <token>`
4. السيرفر يتحقق من **التوقيع** والـ **`exp`** — من غير ما يرجع للداتابيز.

⚠️ **نقاط بيسألوا عنها:**
- الـ **Payload مش مشفّر** — أي حد يقدر يفكّه بـ Base64. **متحطّش فيه بيانات حساسة.**
- **مايتلغيش (Can't revoke)** قبل ما ينتهي → عشان كده بنخلّي مدته قصيرة + Refresh Token.
- خزّنه في **HttpOnly Cookie** أأمن من `localStorage` (اللي معرّض لـ XSS).
- **متقبلش `alg: none`** — ثغرة كلاسيكية.

---

### 7) إيه الفرق بين Access Token و Refresh Token؟

| | Access Token | Refresh Token |
|---|---|---|
| المدة | قصيرة (15 دقيقة) | طويلة (أيام/أسابيع) |
| بيتبعت فين | مع **كل** Request | عند تجديد التوكن **بس** |
| مخزّن في السيرفر؟ | ❌ | ✅ (عشان نقدر نلغيه) |
| لو اتسرق | الضرر محدود بالمدة القصيرة | خطير → لازم Rotation |

**ليه الفصل؟** توازن بين **الأمان** (مدة قصيرة) و**تجربة المستخدم** (مايعملش Login كل ربع ساعة).

```
Access انتهى  →  ابعت Refresh Token  →  خد Access جديد
```

**Refresh Token Rotation:** كل مرة تستخدمه، السيرفر يديك واحد جديد ويلغي القديم — لو اتستخدم القديم تاني، ده معناه تسريب → ألغي كل الجلسات.

---

### 8) إيه هو Rate Limiting؟

تحديد عدد الطلبات المسموحة لمستخدم/IP في فترة زمنية.

**ليه؟** منع Brute Force على الـ Login · منع DDoS · حماية موارد السيرفر · منع إساءة استخدام الـ API.

```php
// Laravel
Route::middleware('throttle:60,1')->group(function () {  // 60 طلب/دقيقة
    Route::post('/login', [AuthController::class, 'login']);
});
```

**الـ Response:** `429 Too Many Requests` مع Headers:
```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 12
Retry-After: 45
```

**الخوارزميات:** Fixed Window (أبسط) · **Sliding Window** (أدق) · Token Bucket (بيسمح بـ Bursts) · Leaky Bucket.

**التخزين:** Redis — لأنه سريع وبيدعم عمليات ذرية (Atomic) والـ TTL.

---

## 🔹 الأداء والمهام الخلفية

### 9) إمتى تستخدم Redis؟

**Redis** = مخزن بيانات **In-Memory**، سريع جدًا (Sub-millisecond).

**استخداماته:**

| الاستخدام | ليه Redis |
|---|---|
| **Cache** | أسرع بكتير من الداتابيز |
| **Sessions** | مشترك بين أكتر من سيرفر (Load Balancer) |
| **Queues** | List operations ذرية |
| **Rate Limiting** | `INCR` + `EXPIRE` ذرية |
| **Real-time Leaderboards** | Sorted Sets |
| **Pub/Sub** | إشعارات فورية |
| **Distributed Locks** | `SET NX` |

**أنواع البيانات:** String · List · Set · Sorted Set · Hash · Bitmap · HyperLogLog · Stream.

**إمتى مانستخدموش؟** بيانات ضخمة جدًا (الذاكرة غالية) · بيانات لازم Relational Queries معقدة · بيانات لا يمكن فقدانها مطلقًا (رغم إن فيه Persistence بـ RDB/AOF).

---

### 10) إيه هو Queue؟ وإمتى RabbitMQ أو Redis Queue؟

**Queue** = تأجيل المهام الثقيلة عشان الـ Request يرجع للمستخدم بسرعة.

```php
// من غير Queue: المستخدم يستنى 5 ثواني
Mail::to($user)->send(new WelcomeMail());

// مع Queue: يرجع فورًا، والإيميل يتبعت في الخلفية
SendWelcomeEmail::dispatch($user);
```

**إمتى تستخدمه؟** إرسال إيميلات · معالجة صور/فيديو · تقارير · مناداة APIs خارجية · إشعارات جماعية.

**Redis Queue vs RabbitMQ:**

| | Redis Queue | RabbitMQ |
|---|---|---|
| التعقيد | بسيط جدًا | أعقد (Broker مستقل) |
| السرعة | أسرع | سريع لكن أقل |
| Routing | بسيط | قوي جدًا (Exchanges, Topics) |
| ضمان التسليم | أضعف | **قوي** (Acknowledgements, Durability) |
| مناسب لـ | مشاريع متوسطة، Laravel | أنظمة كبيرة، Microservices |

**القاعدة:** ابدأ بـ **Redis Queue** — لو احتجت Routing معقد أو ضمانات تسليم قوية أو تكامل بين خدمات بلغات مختلفة → **RabbitMQ**.

**مصطلحات لازم تعرفها:** Worker · Job · Failed Jobs · Retry · **Idempotency** (المهمة تتكرر من غير أثر جانبي).

---

### 11) إيه هو Cron Job؟

مهمة بتتنفّذ **تلقائيًا في وقت محدد** على مستوى نظام التشغيل.

```
* * * * *  command
│ │ │ │ │
│ │ │ │ └── يوم الأسبوع (0-7)
│ │ │ └──── الشهر (1-12)
│ │ └────── يوم الشهر (1-31)
│ └──────── الساعة (0-23)
└────────── الدقيقة (0-59)
```

```bash
0 2 * * *  /usr/bin/php /var/www/backup.php    # كل يوم 2 صباحًا
```

**في Laravel** — إدخال واحد بس في الـ crontab:
```bash
* * * * * cd /path && php artisan schedule:run >> /dev/null 2>&1
```
وباقي الجدولة في الكود:
```php
Schedule::command('reports:generate')->dailyAt('03:00');
Schedule::job(new CleanupJob)->hourly()->withoutOverlapping();
```

**فرق مهم:** **Cron** = تنفيذ حسب **الوقت**. **Queue** = تنفيذ حسب **الحدث** في أقرب فرصة.

---

## 🔹 الأنماط المعمارية

### 12) إيه هو Repository Pattern؟

طبقة وسيطة بتفصل **منطق الوصول للبيانات** عن باقي التطبيق.

```php
interface UserRepositoryInterface {
    public function findById(int $id): ?User;
    public function findActiveUsers(): Collection;
}

class EloquentUserRepository implements UserRepositoryInterface {
    public function findById(int $id): ?User {
        return User::find($id);
    }
    public function findActiveUsers(): Collection {
        return User::where('status', 'active')->get();
    }
}

// الـ Controller مايعرفش أي حاجة عن Eloquent
class UserController {
    public function __construct(private UserRepositoryInterface $users) {}
}
```

**الفوايد:** تركيز استعلامات الداتابيز في مكان واحد · قابلية الاختبار (Mock سهل) · تقدر تغيّر من Eloquent لـ MongoDB من غير ما تلمس باقي الكود.

⚠️ **رأي شائع بيتقال في الإنترفيو:** في Laravel الـ Repository أحيانًا بيكون **زيادة غير مبررة** لأن Eloquent هو أصلًا تطبيق لنمط Active Record. استخدمه لما يكون فيه استعلامات معقدة ومتكررة أو احتمال تغيير مصدر البيانات — مش لكل Model.

---

### 13) إيه هو Service Layer؟

طبقة بتحتوي **منطق الشغل (Business Logic)** — بتخلّي الـ Controller نحيف والـ Model نضيف.

```php
class OrderService {
    public function __construct(
        private OrderRepositoryInterface $orders,
        private PaymentGatewayInterface $payment,
        private MailerInterface $mailer,
    ) {}

    public function placeOrder(array $data): Order {
        return DB::transaction(function () use ($data) {
            $order = $this->orders->create($data);
            $this->payment->charge($order->total);
            $this->mailer->sendConfirmation($order);
            return $order;
        });
    }
}

// الـ Controller بقى 3 سطور
class OrderController {
    public function __construct(private OrderService $service) {}

    public function store(StoreOrderRequest $request) {
        $order = $this->service->placeOrder($request->validated());
        return new OrderResource($order);
    }
}
```

**الفايدة:** المنطق قابل لإعادة الاستخدام (من Controller أو Job أو Artisan Command) وقابل للاختبار من غير HTTP.

**القاعدة الذهبية:** **Thin Controllers, Fat Services.**

---

## 🔹 الاختبار والنشر

### 14) إزاي تستخدم PHPUnit؟

```bash
composer require --dev phpunit/phpunit
./vendor/bin/phpunit
```

```php
use PHPUnit\Framework\TestCase;

class DiscountCalculatorTest extends TestCase
{
    private DiscountCalculator $calculator;

    protected function setUp(): void {
        $this->calculator = new DiscountCalculator();
    }

    public function test_applies_ten_percent_for_gold_members(): void {
        // Arrange
        $order = new Order(total: 1000, tier: 'gold');

        // Act
        $result = $this->calculator->apply($order);

        // Assert
        $this->assertSame(900.0, $result);
    }

    public function test_throws_on_negative_total(): void {
        $this->expectException(InvalidArgumentException::class);
        $this->calculator->apply(new Order(total: -1));
    }
}
```

**مفاهيم لازم تعرفها:**
- نمط **AAA**: Arrange → Act → Assert.
- `setUp()` / `tearDown()` — قبل وبعد كل Test.
- **Data Providers** — نفس الاختبار ببيانات مختلفة.
- **Mocking** — تزييف الاعتماديات:
```php
$mailer = $this->createMock(MailerInterface::class);
$mailer->expects($this->once())->method('send');
```
- **Code Coverage** — نسبة الكود المغطّى (مؤشر، مش هدف في حد ذاته).

**في Laravel:** `php artisan test`، وفيه Feature Tests بتضرب Endpoints حقيقية، و **Pest** كبديل بصيغة أبسط.

---

### 15) إيه هو Docker؟ وليه مهم لمطور PHP؟

**Docker** = تشغيل التطبيق داخل **Container** معزول فيه كل اعتمادياته (PHP, Extensions, MySQL, Redis).

**بيحل مشكلة:** *"شغال عندي على الجهاز!"* — نفس البيئة بالظبط عند كل المطورين وفي Production.

**Container ≠ Virtual Machine:** الـ Container بيشارك الـ Kernel بتاع النظام → أخف وأسرع بكتير من VM.

```dockerfile
FROM php:8.3-fpm-alpine
RUN docker-php-ext-install pdo_mysql opcache
WORKDIR /var/www
COPY . .
RUN composer install --no-dev --optimize-autoloader
```

```yaml
# docker-compose.yml
services:
  app:
    build: .
  db:
    image: mysql:8
    environment:
      MYSQL_DATABASE: app
  redis:
    image: redis:alpine
```

**ليه مهم لمطور PHP؟**
- تشغيل أكتر من مشروع بإصدارات PHP مختلفة في نفس الوقت.
- بيئة تطوير جاهزة في دقيقة (`docker compose up`).
- نفس الـ Image بيروح للـ CI وللـ Production.
- أساس أي شغل حديث بـ Kubernetes أو أي Cloud.

**مصطلحات:** Image · Container · Dockerfile · Volume · Network · Multi-stage build.

---

# ✅ خلاصة المستوى ده

| الموضوع | لو Junior اعرف | لو Mid لازم |
|---|---|---|
| **DI** | إيه هو وليه | تطبّقه في كل كلاس |
| **SOLID** | S و D | الخمسة بأمثلة من شغلك |
| **PSR** | PSR-4 و PSR-12 | Interoperability و PSR-3/7/11 |
| **Session Hijacking** | HTTPS + HttpOnly | كل الطبقات + `use_strict_mode` |
| **File Upload** | تحقّق من النوع | كل الطبقات + إعدادات السيرفر |
| **JWT** | إيه هو | التطبيق + قيوده (revocation) |
| **Rate Limiting** | Middleware جاهز | الخوارزميات + Redis |
| **Redis** | Cache | 7 استخدامات + أنواع البيانات |
| **Queue** | تأجيل المهام | Workers + Retries + Idempotency |
| **Repository/Service** | الفكرة | إمتى تستخدمه **وإمتى لأ** |
| **PHPUnit** | AAA | Mocking + CI |
| **Docker** | إيه هو | Dockerfile + Compose |
