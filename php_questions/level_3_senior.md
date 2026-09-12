# 🔴 المستوى الثالث — Senior / Architect

> **مش مطلوب منك دلوقتي كـ Junior.**
> الملف ده للقراءة العامة والفضول — مش للحفظ.
>
> لو اتسألت في حاجة من هنا في إنترفيو Junior:
> **"الفكرة العامة إنه كذا، بس دي حاجة مااشتغلتش عليها لسه"** ← إجابة كاملة ومحترمة.
> المُحاوِر بيحترم الصدق أكتر بكتير من إجابة محفوظة بتنهار مع أول سؤال متابعة.

---

## 🔹 داخل الـ PHP Engine

### 11) إيه هو Output Buffering؟

آلية بتخزّن المخرجات في **Buffer في الذاكرة** بدل ما تبعتها للمتصفح فورًا.

**أشهر فايدة:** حل مشكلة `Headers already sent` — لأن الـ HTTP Headers لازم تسبق أي Output.

```php
ob_start();
echo "Hello World";
$content = ob_get_clean();

header('Content-Length: ' . strlen($content));  // شغالة لأن مافيش Output اتبعت
echo $content;
```

| الدالة | الوظيفة |
|---|---|
| `ob_start()` | يبدأ Buffer جديد |
| `ob_get_contents()` | يقرأ بدون مسح |
| `ob_get_clean()` | يقرأ + يمسح + يقفل |
| `ob_end_flush()` | يبعت ويقفل |
| `ob_get_level()` | عدد الـ Buffers المتداخلة |

**الاستخدام العملي الأهم — أساس Template Engines:**

```php
function render(string $view, array $data = []): string {
    extract($data);
    ob_start();
    include $view;
    return ob_get_clean();
}
```

Blade و Twig بيشتغلوا بالمبدأ ده. كمان بيستخدم في **Full Page Caching** و **Compression** (`ob_start('ob_gzhandler')`).

---

## 🔹 البنية التحتية والأداء

### 40) إيه الفرق بين Redis و Memcached؟

| | Redis | Memcached |
|---|---|---|
| أنواع البيانات | **غنية** (List, Set, Sorted Set, Hash, Stream…) | String فقط |
| Persistence | ✅ (RDB + AOF) | ❌ (RAM بحت) |
| Replication | ✅ Master-Replica | ❌ |
| Clustering | ✅ مدمج | Client-side بس |
| Threading | Single-threaded (I/O متعدد الخيوط من v6) | **Multi-threaded** |
| Pub/Sub | ✅ | ❌ |
| Transactions / Lua | ✅ | ❌ |
| استهلاك الذاكرة للـ Keys البسيطة | أعلى شوية | **أكفأ** |
| Eviction | 8 سياسات (LRU, LFU, TTL…) | LRU بس |

**متى Memcached فعلًا؟** كاش بسيط جدًا (String → String)، حجم ضخم جدًا، وسيرفر بعدد أنوية عالي يستفيد من الـ Multi-threading.

**عمليًا في 2026:** **Redis هو الاختيار الافتراضي** في 95% من الحالات — لأنه بيعمل كل اللي Memcached بيعمله وزيادة (Queues, Locks, Rate Limiting, Pub/Sub). Memcached بقى Legacy إلى حد كبير.

**نقطة عميقة:** Redis كونه Single-threaded ده **مش عيب** — بالعكس، بيضمن **الذرية (Atomicity)** بدون Locks، وبيخلّي العمليات متوقعة الأداء. الاختناق غالبًا بيكون الشبكة مش المعالج.

---

### 44) إزاي تتعامل مع Large File Uploads؟

المشكلة: رفع ملف 5GB بالطريقة العادية = استهلاك ذاكرة ضخم + Timeout + لو الاتصال قطع تبدأ من الأول.

**1) Chunked Upload** — قسّم الملف لقطع صغيرة من العميل، ارفعها واحدة واحدة، ودمّجها على السيرفر.

```javascript
const CHUNK = 5 * 1024 * 1024;   // 5MB
for (let i = 0; i < Math.ceil(file.size / CHUNK); i++) {
    await upload(file.slice(i * CHUNK, (i + 1) * CHUNK), i);
}
```

**الميزة الحقيقية:** **Resumable Upload** — لو الاتصال قطع، تكمّل من آخر Chunk نجح.

**2) Streaming بدل تحميل الملف في الذاكرة**
```php
$in  = fopen('php://input', 'rb');
$out = fopen($path, 'wb');
stream_copy_to_stream($in, $out);   // ثابت في استهلاك الذاكرة
```

**3) Direct-to-Cloud (الأفضل معماريًا)**
السيرفر بيولّد **Pre-signed URL** والعميل يرفع **مباشرة لـ S3** — السيرفر مايشوفش الملف أصلًا.
```php
$url = Storage::disk('s3')->temporaryUploadUrl($path, now()->addMinutes(10));
```

**4) إعدادات لازمة**
```ini
upload_max_filesize = 5G
post_max_size       = 5G
max_execution_time  = 0
memory_limit        = 256M   ; مايتأثرش لو بتعمل Streaming
```
```nginx
client_max_body_size 5G;
client_body_timeout  300s;
```

**5) معالجة ما بعد الرفع في Queue** — التحقق، الفحص من الفيروسات، الضغط، توليد Thumbnails.

**6) بروتوكولات جاهزة:** **tus.io** (معيار مفتوح للرفع القابل للاستئناف) · **Uppy** في الواجهة.

---

## 🔹 المعمارية

### 45) إيه الفرق بين Monolithic و Microservices؟

| | Monolith | Microservices |
|---|---|---|
| البنية | تطبيق واحد | خدمات صغيرة مستقلة |
| النشر | مرة واحدة للكل | كل خدمة لوحدها |
| التوسّع (Scaling) | للتطبيق كله | لكل خدمة على حدة |
| الداتابيز | مشتركة | **قاعدة لكل خدمة** |
| التواصل | استدعاء دوال (سريع) | HTTP / gRPC / Queue (بطيء وقابل للفشل) |
| Transactions | ACID سهلة | **موزّعة ومعقّدة** (Saga) |
| الفريق | فريق واحد | فريق لكل خدمة |
| التعقيد التشغيلي | منخفض | **عالي جدًا** (K8s, Tracing, Service Mesh) |

**التحديات الحقيقية للـ Microservices:**
- **Distributed Transactions** → **Saga Pattern** (سلسلة معاملات محلية + تعويضات) أو **Outbox Pattern**.
- **Eventual Consistency** بدل الاتساق الفوري.
- **الفشل الشبكي** → Circuit Breaker · Retry مع Exponential Backoff · Timeouts.
- **المراقبة** → Distributed Tracing (OpenTelemetry, Jaeger).
- **إدارة البيانات** → CQRS · Event Sourcing.

⚠️ **الإجابة الناضجة في الإنترفيو:**
> **"ابدأ بـ Monolith."** — Microservices بتحل مشاكل **تنظيمية** (فِرَق كتيرة تتصادم في الـ Deploy) قبل ما تحل مشاكل تقنية. قانون Conway بيقول إن معمارية النظام بتعكس هيكل الفريق. لو فريقك 5 أفراد، الـ Microservices هتضيف تعقيد أكتر من قيمتها.

**الحل الوسط: Modular Monolith** — تطبيق واحد بحدود داخلية واضحة بين الوحدات، سهل تفصله لاحقًا لو احتجت.

---

### 46) إيه أشهر Design Patterns في PHP؟

**Creational (الإنشاء)**

| النمط | الفكرة | في Laravel |
|---|---|---|
| **Singleton** | نسخة واحدة بس | `$app->singleton()` |
| **Factory** | إنشاء الكائنات بدون تحديد الكلاس | Model Factories |
| **Builder** | بناء كائن معقد خطوة بخطوة | Query Builder |

**Structural (البنية)**

| النمط | الفكرة | مثال |
|---|---|---|
| **Adapter** | يوفّق بين واجهتين مختلفتين | Filesystem drivers |
| **Decorator** | يضيف سلوك بدون تعديل الكلاس | Middleware |
| **Facade** | واجهة بسيطة لنظام معقّد | `Cache::`, `DB::` |
| **Proxy** | وسيط يتحكم في الوصول | Lazy Loading |

**Behavioral (السلوك)**

| النمط | الفكرة | مثال |
|---|---|---|
| **Strategy** | تبديل الخوارزمية وقت التشغيل | طرق الدفع |
| **Observer** | إشعار عند حدوث حدث | Events & Listeners |
| **Repository** | تجريد الوصول للبيانات | — |
| **Chain of Responsibility** | تمرير الطلب عبر سلسلة | **Middleware Pipeline** |

```php
// Strategy — أشهر واحد عمليًا
interface PaymentStrategy {
    public function pay(float $amount): bool;
}

class OrderProcessor {
    public function __construct(private PaymentStrategy $strategy) {}
    public function checkout(float $amount): bool {
        return $this->strategy->pay($amount);
    }
}

new OrderProcessor(new StripePayment());
new OrderProcessor(new PaypalPayment());   // من غير تعديل سطر
```

⚠️ **نقطة نضج:** الـ Patterns **حلول لمشاكل**، مش هدف في حد ذاتها. تطبيق Pattern على مشكلة مش موجودة = **Over-engineering**. المُحاوِر الشاطر بيقدّر لما تقول "استخدمت Strategy هنا لأن كان فيه 4 طرق دفع وكان الـ switch بيكبر".

---

### 49) إيه هو Dependency Container؟

**كائن مسؤول عن إنشاء الكائنات وحلّ اعتمادياتها تلقائيًا.**

```php
// من غير Container 😩
$db      = new PDO(...);
$repo    = new UserRepository($db);
$mailer  = new SmtpMailer($config);
$service = new UserService($repo, $mailer);

// مع Container ✨
$service = $container->get(UserService::class);
```

**إزاي بيعرف؟ — Auto-wiring عبر Reflection:**

```php
class Container {
    private array $bindings = [];

    public function bind(string $abstract, callable|string $concrete): void {
        $this->bindings[$abstract] = $concrete;
    }

    public function get(string $id): object {
        if (isset($this->bindings[$id])) {
            $concrete = $this->bindings[$id];
            return is_callable($concrete) ? $concrete($this) : $this->get($concrete);
        }

        $reflector   = new ReflectionClass($id);
        $constructor = $reflector->getConstructor();

        if (!$constructor) {
            return new $id();
        }

        $deps = array_map(
            fn(ReflectionParameter $p) => $this->get($p->getType()->getName()),
            $constructor->getParameters()
        );

        return $reflector->newInstanceArgs($deps);   // Recursive resolution
    }
}
```

**المفاهيم الأساسية:**
- **Binding** — ربط Interface بتنفيذ معيّن.
- **Auto-wiring** — الحل التلقائي عبر Type Hints.
- **Singleton vs Transient** — نسخة واحدة مشتركة أم نسخة جديدة كل مرة.
- **Contextual Binding** — نفس الـ Interface بتنفيذ مختلف حسب السياق.
- **PSR-11** — المعيار الموحّد (`get()` / `has()`).

**في Laravel:**
```php
$this->app->bind(PaymentInterface::class, StripeGateway::class);
$this->app->singleton(Analytics::class);

$this->app->when(PhotoController::class)
          ->needs(Filesystem::class)
          ->give(fn() => Storage::disk('s3'));
```

**العلاقة بالـ DI:** الـ **DI** هو المبدأ (مرّر الاعتماديات من بره)، والـ **Container** هو الأداة اللي بتأتمت المبدأ ده. تقدر تعمل DI من غير Container، لكن مش العكس.

---

## 🔹 النشر

### 53) إزاي تعمل Deploy لتطبيق PHP بشكل احترافي؟

**1) Zero-Downtime Deployment (النشر بدون توقف)**

```
releases/
  ├── 2026-08-31-100000/
  ├── 2026-08-31-140000/   ← الإصدار الجديد
current -> releases/2026-08-31-140000   (Symlink)
shared/
  ├── .env
  └── storage/
```

الفكرة: جهّز الإصدار الجديد بالكامل في مجلد جديد، وبعد ما يخلص **بدّل الـ Symlink في عملية ذرية واحدة**. لو حصلت مشكلة، الرجوع (Rollback) = تبديل الـ Symlink للمجلد القديم.

**أدوات:** Deployer · Envoyer · Capistrano · Ansible.

**2) CI/CD Pipeline**

```yaml
# .github/workflows/deploy.yml
on:
  push:
    branches: [main]

jobs:
  test:
    steps:
      - run: composer install
      - run: ./vendor/bin/phpstan analyse    # تحليل ثابت
      - run: ./vendor/bin/pint --test        # معايير الكود
      - run: php artisan test                # الاختبارات

  deploy:
    needs: test          # ← مايعملش Deploy إلا لو الاختبارات نجحت
    steps:
      - run: dep deploy production
```

**3) تحسينات Production**
```bash
composer install --no-dev --optimize-autoloader --classmap-authoritative

php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan event:cache

npm run build
```

```ini
; php.ini
opcache.enable=1
opcache.validate_timestamps=0      ; مايفحصش تغيّر الملفات ← أسرع
opcache.memory_consumption=256
opcache.jit=tracing
```

**4) Database Migrations**
- `php artisan migrate --force` (بدون تفاعل).
- **Backward-compatible migrations** — أضف عمود جديد الأول، انشر الكود، بعدين احذف القديم في نشرة تانية (عشان يشتغل الإصداران معًا أثناء النشر).
- **متعملش Migration ثقيلة على جدول ضخم في وقت الذروة** — استخدم `pt-online-schema-change` أو ما شابه.

**5) البنية التحتية**
- **Load Balancer** + أكتر من App Server (Stateless — الـ Sessions في Redis).
- **Queue Workers** مُدارة بـ **Supervisor** أو **Laravel Horizon**.
- `php artisan queue:restart` بعد كل نشر (الـ Workers بتحتفظ بالكود القديم في الذاكرة).
- **Maintenance Mode** عند الضرورة: `php artisan down --secret=...`.

**6) المراقبة والرصد (Observability)**
- **Error Tracking:** Sentry / Bugsnag.
- **APM:** New Relic / Datadog / Laravel Telescope (في بيئة التطوير).
- **Logs مركزية:** ELK / Grafana Loki.
- **Health Check Endpoint** يقرأه الـ Load Balancer.
- **Uptime Monitoring** + تنبيهات.

**7) الأمان**
- `.env` **مش** في Git إطلاقًا · إدارة الأسرار بـ Vault / AWS Secrets Manager.
- `APP_DEBUG=false` في Production (⚠️ تسريب `APP_DEBUG=true` ثغرة خطيرة).
- HTTPS + HSTS · Security Headers · Firewall.
- صلاحيات ملفات صحيحة · تحديثات أمنية دورية (`composer audit`).

**8) استراتيجيات نشر متقدمة**
- **Blue-Green** — بيئتان متطابقتان، بدّل حركة المرور بينهما.
- **Canary** — وجّه 5% من المستخدمين للإصدار الجديد، راقب، ثم وسّع.
- **Feature Flags** — انشر الكود مقفول، وافتحه عند الجاهزية.

---

# 📌 خلاصة

الملف ده **مرجع**، مش مادة مذاكرة. الفايدة الحقيقية منه دلوقتي:

1. **تعرف إن المواضيع دي موجودة** — عشان لما تسمعها متبقاش صدمة.
2. **تفهم السياق** — ليه Redis أفضل من Memcached، وليه Monolith مش عيب.
3. **تعرف طريقك** — دي خريطة الـ 3–5 سنين الجاية في مسارك.

> **الجملة اللي تقولها في الإنترفيو ومحدش هيلومك عليها:**
> *"دي حاجة قريت عنها وعارف الفكرة العامة، بس مااشتغلتش عليها في مشروع حقيقي لسه."*
