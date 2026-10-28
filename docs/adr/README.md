# ثبت تصمیم‌های معماری (ADR)

هر تصمیم مهم یک فایل دارد. تغییر تصمیم = ADR جدید که قبلی را «جایگزین‌شده» علامت می‌زند.

- [ADR-0001](0001-monorepo.md): ساختار مخزن: Monorepo
- [ADR-0002](0002-backend-fastapi.md): بک‌اند: Python 3.12 + FastAPI
- [ADR-0003](0003-postgres-integer-rials.md): دیتابیس PostgreSQL و مبلغ به صورت عدد صحیح ریال
- [ADR-0004](0004-ledger-void-not-delete.md): دفتر کل تغییرناپذیر و ابطال به‌جای حذف
- [ADR-0005](0005-jalali-dates.md): مدیریت تاریخ شمسی
- [ADR-0006](0006-auth-tokens.md): احراز هویت: Argon2id + JWT کوتاه‌مدت + Refresh چرخشی
- [ADR-0007](0007-ai-provider-boundary.md): لایه هوش مصنوعی: پیشنهاد، نه تصمیم
- [ADR-0008](0008-celery-redis.md): وظایف پس‌زمینه: Celery + Redis
- [ADR-0009](0009-speech-faster-whisper.md): تشخیص گفتار فارسی: faster-whisper
- [ADR-0010](0010-tenant-isolation.md): جداسازی کاربران: فیلتر برنامه + کلید خارجی مرکب + RLS
- [ADR-0011](0011-frontend-stack.md): فرانت‌اند: React + TypeScript + Vite + Tailwind
- [ADR-0012](0012-deploy-docker-caddy.md): استقرار: Docker Compose + Caddy + GitHub Actions
