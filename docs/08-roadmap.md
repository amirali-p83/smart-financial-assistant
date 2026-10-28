# ۸. نقشه راه فازها

قالب ثابت پایان هر فاز:

```bash
git checkout -b <branch>
git add .
git commit -m "<commit message>"
git push -u origin <branch>
# گیت‌هاب: Pull Request به main ← CI سبز ← Squash and merge
git checkout main && git pull
git tag -a <tag> -m "<tag message>" && git push origin <tag>
```

| فاز | شاخه | تگ | کامیت |
|---|---|---|---|
| ۰ تحلیل و طراحی | `phase/0-design` | `v0.0.0` | `docs: add architecture, data model, financial rules and phased roadmap` |
| ۱ زیرساخت | `phase/1-infrastructure` | `v0.1.0` | `chore: scaffold monorepo with FastAPI, React RTL, Docker and CI pipeline` |
| ۲ هسته مالی | `phase/2-financial-core` | `v0.2.0` | `feat(core): implement ledger, debts, settlements, cheques and idempotent operations with tests` |
| ۳ API و Auth | `phase/3-api-auth` | `v0.3.0` | `feat(api): add JWT auth, user-scoped financial endpoints, rate limiting and OpenAPI docs` |
| ۴ فرانت‌اند | `phase/4-frontend` | `v0.4.0` | `feat(web): build responsive RTL dashboard and financial screens connected to the API` |
| ۵ دستیار هوشمند | `phase/5-ai-assistant` | `v0.5.0` | `feat(assistant): add Persian command parsing, voice input, preview and confirm flow` |
| ۶ یادآوری و گزارش | `phase/6-reminders-reports` | `v0.6.0` | `feat(jobs): add background reminders and monthly, cash-flow and obligation reports` |
| ۷ تست و انتشار | `phase/7-release` | `v1.0.0` | `chore(release): add e2e tests, security hardening, production deploy and backup tooling` |

## معیار پذیرش هر فاز

| فاز | معیار |
|---|---|
| ۰ | همه سندهای `docs/` موجود و بازبینی‌شده؛ سؤال‌های باز بسته یا ثبت‌شده |
| ۱ | `docker compose up` دیتابیس و Redis را بالا بیاورد؛ بک‌اند `/health/live` و فرانت صفحه RTL را نشان دهد؛ CI سبز |
| ۲ | همه TC-01…TC-26 و INV-01…06 اجرا و سبز |
| ۳ | TC-27 و تست‌های خطای API سبز؛ OpenAPI قابل تولید؛ تایپ فرانت تولید شود |
| ۴ | داشبورد در ۳۹۰/۸۲۰/۱۴۴۰ با ماکاپ مقایسه شود؛ سناریوی «ثبت چک ← داشبورد» با Playwright سبز |
| ۵ | TC-28/29 سبز؛ گزارش صادقانه کیفیت LLM و Whisper با نمونه واقعی |
| ۶ | تکرار وظیفه یادآوری ردیف جدید نسازد؛ اعداد گزارش‌ها با مقادیر دستی مطابق |
| ۷ | E2E کامل؛ اسکن امنیتی؛ بازیابی بکاپ موفق؛ استقرار روی سرور واقعی |

## ریسک‌ها

| ریسک | کاهش |
|---|---|
| دسترسی سرور به LLM خارجی | Provider قابل تعویض + حالت `Fake`/محلی |
| سرعت/دقت Whisper روی CPU | اندازه‌گیری در فاز ۵؛ مدل کوچک‌تر یا GPU؛ ورودی متنی همیشه موجود |
| پیچیدگی RLS با Celery | spike کوتاه در فاز ۲؛ در صورت مشکل جدی، ADR-0010 بازبینی می‌شود (کنترل‌های برنامه و FK مرکب می‌مانند) |
| تفاوت ماکاپ و پیاده‌سازی | مقایسه تصویری در فاز ۴؛ دریافت فایل Figma/تصاویر جدا |
| تغییر قواعد مالی در میانه کار | هر تغییر با به‌روزرسانی سند ۴ و تست همراه است |
