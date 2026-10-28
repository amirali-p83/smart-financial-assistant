# دستیار مالی هوشمند (Smart Financial Assistant)

وب‌اپلیکیشن مالی فارسی و راست‌چین برای مغازه‌داران و کسب‌وکارهای کوچک؛ مدیریت حساب‌ها، درآمد و هزینه، چک‌ها، بدهی و مطالبات، یادآوری سررسیدها و گزارش‌های مالی، به‌همراه دستیار هوشمند متنی و صوتی.

> **وضعیت فعلی: فاز ۰ (تحلیل و طراحی فنی).** هنوز کد برنامه‌ای در مخزن وجود ندارد. فقط مستندات طراحی و تصمیم‌های معماری ثبت شده‌اند.

## فناوری‌ها (خلاصه)

| بخش | انتخاب |
|---|---|
| بک‌اند | Python 3.12، FastAPI، SQLAlchemy 2.0، Alembic |
| دیتابیس | PostgreSQL 16 (مبالغ: عدد صحیح به ریال) |
| فرانت‌اند | React 18، TypeScript، Vite، Tailwind CSS (RTL)، shadcn/ui |
| وظایف پس‌زمینه | Celery + Redis |
| هوش مصنوعی | LLM با خروجی ساخت‌یافته پشت اینترفیس قابل تعویض |
| گفتار فارسی | faster-whisper (Whisper) |
| استقرار | Docker Compose، Caddy، GitHub Actions، GHCR |

جزئیات و دلیل هر انتخاب: [`docs/adr/`](docs/adr)

## ساختار مخزن

```
smart-financial-assistant/
├── backend/        # (فاز ۱ به بعد) FastAPI
├── frontend/       # (فاز ۱ به بعد) React + Vite
├── deploy/         # (فاز ۷) docker-compose تولیدی، Caddy، بکاپ
├── docs/           # مستندات طراحی (فاز ۰)
└── .github/        # قالب PR و workflowها
```

## فهرست مستندات

| فایل | محتوا |
|---|---|
| [`docs/01-requirements.md`](docs/01-requirements.md) | نیازمندی‌ها، محدوده، فرضیات |
| [`docs/02-architecture.md`](docs/02-architecture.md) | معماری لایه‌ای و مرز مسئولیت‌ها |
| [`docs/03-data-model.md`](docs/03-data-model.md) | ERD و جزئیات جداول و قیود |
| [`docs/04-financial-rules.md`](docs/04-financial-rules.md) | قواعد مالی همراه با مثال عددی و شناسه تست |
| [`docs/05-api-contract.md`](docs/05-api-contract.md) | قرارداد API |
| [`docs/06-security-threat-model.md`](docs/06-security-threat-model.md) | مدل تهدید و کنترل‌های امنیتی |
| [`docs/07-ai-assistant-design.md`](docs/07-ai-assistant-design.md) | طراحی دستیار هوشمند و مرز اعتماد |
| [`docs/08-roadmap.md`](docs/08-roadmap.md) | فازبندی، معیار پذیرش، شاخه و تگ هر فاز |
| [`docs/09-ui-design-system.md`](docs/09-ui-design-system.md) | توکن‌های طراحی و چیدمان ریسپانسیو |
| [`docs/adr/`](docs/adr) | ثبت تصمیم‌های معماری (ADR) |

## روش کار

هر فاز یک شاخه (`phase/N-...`)، یک Pull Request و یک تگ دارد. پیام کامیت‌ها طبق [Conventional Commits](https://www.conventionalcommits.org/) است. راهنما: [`CONTRIBUTING.md`](CONTRIBUTING.md)

## مجوز

تمام حقوق محفوظ است (فعلاً). جزئیات در [`LICENSE`](LICENSE).
