# ۲. معماری سیستم

## ۲.۱ نمای کلی

```mermaid
flowchart LR
    subgraph Client["مرورگر"]
      UI["React SPA<br/>RTL + Jalali"]
      MIC["MediaRecorder"]
    end
    subgraph Edge["لبه"]
      CADDY["Caddy<br/>HTTPS"]
    end
    subgraph Backend["بک‌اند (FastAPI)"]
      API["لایه API<br/>اعتبارسنجی، Auth، خطا"]
      BL["منطق کسب‌وکار<br/>سرویس‌های مالی"]
      REP["گزارش‌گیری"]
      AI["لایه هوش مصنوعی<br/>Parser + Provider"]
      DAL["دسترسی به داده<br/>Repositories"]
    end
    subgraph Workers["پس‌زمینه"]
      BEAT["Celery beat"]
      WRK["Celery worker"]
    end
    PG[("PostgreSQL")]
    RD[("Redis")]
    LLM["LLM Provider<br/>(خارجی/قابل تعویض)"]
    STT["faster-whisper"]

    UI -->|REST /api/v1| CADDY --> API
    MIC -->|صدا| API
    API --> BL
    API --> REP
    API --> AI
    BL --> DAL
    REP --> DAL
    DAL --> PG
    AI -->|فقط متن و فهرست نام‌ها| LLM
    AI --> STT
    AI -->|خروجی ساخت‌یافته| BL
    API --> RD
    BEAT --> RD
    WRK --> RD
    WRK --> BL
```

**قاعده‌های بنیادین:**
- فرانت‌اند مستقیماً به دیتابیس وصل نمی‌شود.
- لایه هوش مصنوعی به دیتابیس دسترسی ندارد؛ فقط خروجی ساخت‌یافته تولید می‌کند و اجرا همیشه از مسیر سرویس‌های مالی است.
- قواعد مالی فقط در لایه منطق کسب‌وکار (و قیود دیتابیس) هستند، نه در فرانت و نه در پاسخ مدل.

## ۲.۲ لایه‌ها و مرز مسئولیت

| لایه | مسئولیت | ممنوعیت‌ها |
|---|---|---|
| UI | نمایش، فرم، اعتبارسنجی اولیه، پیش‌نمایش | محاسبه موجودی/بدهی؛ اتصال به دیتابیس |
| API | Schema (Pydantic)، Auth، مجوز، قالب پاسخ و خطا، Rate limit، Idempotency-Key | منطق مالی |
| منطق کسب‌وکار | قواعد مالی، تراکنش اتمیک، ابطال، ایدمپوتنسی، سابقه تغییرات | وابستگی به FastAPI/HTTP |
| دسترسی به داده | Repository ها، همه کوئری‌ها با `user_id`، قفل ردیف | قاعده کسب‌وکار |
| هوش مصنوعی | تفسیر زبان، استخراج پارامتر، پرسش تکمیلی، تبدیل به Schema | دسترسی دیتابیس؛ محاسبه مالی؛ اجرای مستقل |
| پس‌زمینه | زمان‌بندی، یادآوری، retry، ثبت وضعیت | وابستگی به مرورگر |
| گزارش‌گیری | فقط‌خواندنی؛ محاسبات از ledger | تخمین مدل زبانی |
| امنیت و نظارت | Auth، RLS، لاگ ساختاریافته، audit، متریک | (در همه لایه‌ها اعمال می‌شود) |

## ۲.۳ ساختار پوشه بک‌اند (برای فاز ۱)

```
backend/
├── app/
│   ├── api/v1/            # routerها، dependencyها، schemaهای درخواست/پاسخ
│   ├── core/              # config، security، logging، errors
│   ├── domain/            # Value objectها (Money, JalaliDate)، enumها، قواعد خالص
│   ├── services/          # ledger، debts، cheques، reports، reminders
│   ├── repositories/      # دسترسی به داده (فقط SQL/ORM)
│   ├── models/            # مدل‌های SQLAlchemy
│   ├── assistant/         # parser، providers، entity resolution، preview
│   ├── jobs/              # taskهای Celery
│   └── main.py
├── alembic/
├── tests/{unit,property,integration,api,assistant}
└── pyproject.toml
```

## ۲.۴ جریان ثبت یک عملیات (نمونه: انتقال وجه)

```mermaid
sequenceDiagram
    participant U as UI
    participant A as API
    participant S as LedgerService
    participant D as PostgreSQL
    U->>A: POST /transfers (Idempotency-Key)
    A->>A: Auth + Schema validation
    A->>S: transfer(user, from, to, amount, date)
    S->>D: BEGIN
    S->>D: بررسی کلید ایدمپوتنسی (INSERT ... ON CONFLICT)
    S->>D: قفل حساب‌ها (SELECT ... FOR UPDATE به ترتیب id)
    S->>D: INSERT transaction + دو ledger_entry (مجموع = ۰)
    S->>D: INSERT audit_log
    S->>D: COMMIT
    S-->>A: نتیجه
    A-->>U: 201 + ذخیره پاسخ برای تکرار
```

## ۲.۵ جریان دستیار هوشمند

```mermaid
sequenceDiagram
    participant U as کاربر
    participant A as Assistant API
    participant P as Parser (LLM + قواعد قطعی)
    participant V as اعتبارسنجی سرویس مالی
    participant S as سرویس مالی
    U->>A: متن یا صدا
    A->>P: متن نرمال‌شده + فهرست نام حساب/شخص/دسته
    P-->>A: قصد + پارامترهای خام
    A->>A: تبدیل مبلغ/تاریخ توسط ماژول قطعی، تطبیق نام‌ها
    A->>V: اعتبارسنجی همان Schema سرویس
    alt اطلاعات ناقص یا مبهم
        A-->>U: سؤال تکمیلی
    else کامل
        A-->>U: پیش‌نمایش + pending_id
        U->>A: تأیید (confirm)
        A->>S: اجرای عملیات از مسیر سرویس واقعی
        S-->>U: نتیجه
    end
```

## ۲.۶ استقرار هدف

```
VPS ── Docker Compose
        ├── caddy        (80/443، HTTPS خودکار)
        ├── frontend     (فایل‌های ثابت)
        ├── api          (uvicorn/gunicorn)
        ├── worker       (celery)
        ├── beat         (celery beat، فقط یک نمونه)
        ├── postgres     (volume + بکاپ)
        └── redis
```
Postgres و Redis روی اینترنت باز نمی‌شوند؛ فقط شبکه داخلی Docker.
