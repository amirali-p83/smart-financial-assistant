# ۵. قرارداد API

پایه: `/api/v1`. فرمت: JSON با `UTF-8`. مستندات خودکار: `/docs` (Swagger) و `/redoc` (در محیط تولید پشت احراز هویت یا غیرفعال). تایپ‌های TypeScript فرانت از `openapi.json` تولید می‌شوند.

## ۵.۱ قراردادهای عمومی

### مبلغ
عدد صحیح **ریال** (`"amount_rial": 200000000`). واحد نمایش (تومان) فقط در UI است. ورودی اعشاری ⇒ `422`.

### تاریخ
- درخواست: ISO میلادی `"2026-12-16"`.
- پاسخ: هر تاریخ دو فیلد دارد: `occurred_on` و `occurred_on_jalali` (`"1405/09/25"`).
- فرانت با تقویم شمسی کار می‌کند و میلادی می‌فرستد؛ تست تطبیق فرانت/بک‌اند روی بازه ۱۳۰۰–۱۵۰۰ نوشته می‌شود.

### قالب موفق
```json
{ "data": { }, "meta": { "request_id": "..." } }
```
لیست‌ها: `meta.page`، `meta.page_size` (حداکثر ۱۰۰)، `meta.total`.

### قالب خطا
```json
{
  "error": {
    "code": "OVERPAYMENT",
    "message": "مبلغ پرداخت از مانده بدهی بیشتر است.",
    "details": [{ "field": "amount_rial", "issue": "exceeds_remaining", "remaining_rial": 200000000 }],
    "request_id": "..."
  }
}
```
`message` فارسی و قابل نمایش؛ `code` پایدار و مخصوص برنامه.

### کدهای وضعیت و خطا

| HTTP | code | معنا |
|---|---|---|
| 400 | `BAD_REQUEST` | درخواست نامعتبر |
| 401 | `UNAUTHENTICATED`، `TOKEN_EXPIRED` | نیاز به ورود |
| 403 | `FORBIDDEN` | دسترسی ندارید |
| 404 | `NOT_FOUND` | یافت نشد (برای داده کاربر دیگر هم همین؛ نشت وجود رکورد ندارد) |
| 409 | `IDEMPOTENCY_KEY_REUSED`، `INVALID_STATE_TRANSITION`، `ALREADY_VOIDED` | تعارض |
| 422 | `VALIDATION_ERROR`، `INVALID_DATE`، `OVERPAYMENT`، `AMOUNT_OUT_OF_RANGE`، `SAME_ACCOUNT_TRANSFER`، `FUTURE_DATE_NOT_ALLOWED` | قاعده کسب‌وکار یا اعتبارسنجی |
| 429 | `RATE_LIMITED` | محدودیت درخواست (`Retry-After`) |
| 500 | `INTERNAL_ERROR` | خطای پیش‌بینی‌نشده (بدون افشای جزئیات) |

### Idempotency-Key
هدر **الزامی** روی همه `POST` های ثبت/تغییر مالی (تراکنش، انتقال، بدهی، تسویه، چک، تغییر وضعیت چک، ابطال، تأیید دستیار). مقدار: UUID تولیدشده در کلاینت برای هر «قصد کاربر». ماندگاری ۲۴ ساعت.

### Rate limit (پیش‌فرض‌ها)
| مسیر | محدودیت |
|---|---|
| `POST /auth/login` | ۵ تلاش در دقیقه به ازای IP+ایمیل، قفل پیش‌رونده |
| `POST /auth/register` | ۵ در ساعت به ازای IP |
| `POST /assistant/*` | ۳۰ در دقیقه به ازای کاربر |
| `POST /assistant/transcribe` | ۱۰ در دقیقه، حداکثر ۶۰ ثانیه و ۲MB |
| سایر نوشتن‌ها | ۱۲۰ در دقیقه به ازای کاربر |

## ۵.۲ Endpointها

### احراز هویت
| متد | مسیر | شرح |
|---|---|---|
| POST | `/auth/register` | ثبت‌نام و ساخت دسته‌های پیش‌فرض |
| POST | `/auth/login` | بازگشت access token (بدنه) + refresh در کوکی httpOnly |
| POST | `/auth/refresh` | چرخش refresh و صدور access جدید |
| POST | `/auth/logout` | ابطال خانواده refresh |
| GET | `/auth/me` | پروفایل |
| PATCH | `/me/settings` | واحد نمایش، افست یادآوری |

### حساب‌ها، دسته‌ها، اشخاص
| متد | مسیر |
|---|---|
| GET/POST | `/accounts` |
| GET/PATCH | `/accounts/{id}` (شامل `balance_rial`) |
| POST | `/accounts/{id}/archive` |
| GET/POST | `/categories` |
| PATCH | `/categories/{id}` |
| GET/POST | `/parties` |
| GET/PATCH | `/parties/{id}` (شامل خلاصه بدهی/طلب) |

### تراکنش‌ها
| متد | مسیر | شرح |
|---|---|---|
| GET | `/transactions` | فیلتر: `kind`، `account_id`، `category_id`، `party_id`، `from`، `to`، `include_voided` |
| POST | `/transactions` | درآمد/هزینه (`kind`: income/expense) |
| POST | `/transfers` | انتقال وجه |
| GET | `/transactions/{id}` | |
| POST | `/transactions/{id}/void` | ابطال (با `reason`) |
| PATCH | `/transactions/{id}` | فقط توضیح و دسته |
| POST | `/transactions/{id}/replace` | ابطال + ثبت جدید اتمیک (ویرایش مبلغ/حساب/تاریخ) |

### بدهی و مطالبات
| متد | مسیر |
|---|---|
| GET/POST | `/debts` (فیلتر `direction`، `status`، `party_id`، `due_before`) |
| GET | `/debts/{id}` (شامل تسویه‌ها و مانده) |
| POST | `/debts/{id}/settlements` |
| POST | `/debts/{id}/settlements/{sid}/void` |
| POST | `/debts/{id}/cancel` |

### چک‌ها
| متد | مسیر | شرح |
|---|---|---|
| GET/POST | `/cheques` | فیلتر `direction`، `status`، `due_before` |
| GET | `/cheques/{id}` | |
| POST | `/cheques/{id}/clear` | بدنه: `account_id`، `cleared_on` |
| POST | `/cheques/{id}/bounce` | |
| POST | `/cheques/{id}/cancel` | |
| POST | `/cheques/{id}/revert-clearing` | ابطال سند وصول |

### گزارش‌ها (فقط‌خواندنی)
| متد | مسیر |
|---|---|
| GET | `/dashboard` (کارت‌های خلاصه، چک‌های نزدیک، خلاصه بدهی‌ها، ۸ تراکنش اخیر) |
| GET | `/reports/monthly?jalali_year=1405&jalali_month=9` |
| GET | `/reports/income-expense-series?months=6` |
| GET | `/reports/cash-flow?from=&to=` |
| GET | `/reports/obligations?until=` (تعهدات آینده، R-14) |
| GET | `/reports/profit-loss?from=&to=` (R-15) |

### یادآوری و اعلان
| متد | مسیر |
|---|---|
| GET | `/notifications` |
| POST | `/notifications/{id}/read` |
| POST | `/notifications/read-all` |
| GET | `/reminders` |
| POST | `/reminders/{id}/dismiss` |

### دستیار هوشمند
| متد | مسیر | شرح |
|---|---|---|
| POST | `/assistant/transcribe` | فایل صوتی ⇒ `{ "transcript": "..." }` (کاربر قبل از ارسال می‌تواند ویرایش کند) |
| POST | `/assistant/commands` | `{ "text": "..." }` ⇒ یکی از: `needs_clarification` (با `question`)، `awaiting_confirmation` (با `preview` و `operation_id`)، `answer` (پرسش‌وپاسخ)، `unsupported` |
| POST | `/assistant/operations/{id}/answer` | پاسخ به سؤال تکمیلی |
| POST | `/assistant/operations/{id}/confirm` | اجرای عملیات (Idempotency-Key الزامی) |
| POST | `/assistant/operations/{id}/reject` | |

نمونه پیش‌نمایش:
```json
{
  "data": {
    "status": "awaiting_confirmation",
    "operation_id": "…",
    "expires_at": "2026-10-25T12:40:00Z",
    "preview": {
      "intent": "create_cheque",
      "title": "ثبت چک صادره",
      "fields": [
        { "label": "طرف حساب", "value": "شرکت میهن" },
        { "label": "مبلغ", "value": "۲۰٬۰۰۰٬۰۰۰ تومان", "amount_rial": 200000000 },
        { "label": "سررسید", "value": "۲۵ آذر ۱۴۰۵", "date": "2026-12-16" }
      ],
      "warnings": []
    }
  }
}
```

### سلامت
`GET /health/live` و `GET /health/ready` (بررسی دیتابیس و Redis؛ بدون داده کاربر).

## ۵.۳ قواعد نسخه‌بندی
تغییر شکننده ⇒ `/api/v2`. افزودن فیلد اختیاری شکننده نیست. فیلدهای حذف‌شده یک نسخه با هشدار `Deprecation` باقی می‌مانند.
