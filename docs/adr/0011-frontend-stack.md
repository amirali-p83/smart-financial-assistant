# ADR-0011: فرانت‌اند: React + TypeScript + Vite + Tailwind

- **وضعیت:** پیشنهادشده (پس از تأیید صاحب محصول: پذیرفته‌شده)
- **تاریخ:** ۲۰۲۶-۱۰-۲۵

## زمینه
نیاز به UI ریسپانسیو RTL، فرم‌های دقیق و تایپ‌سیف بودن در اتصال به API.

## تصمیم
React 18، TypeScript، Vite، Tailwind CSS با RTL، shadcn/ui، TanStack Query، React Hook Form + Zod، Recharts، `lucide-react`، فونت وزیرمتن. تایپ کلاینت از OpenAPI. pnpm.

## گزینه‌های ردشده
Next.js (SSR لازم نیست؛ یک SPA پشت احراز هویت است)؛ Vue (مناسب ولی تجربه/اکوسیستم کامپوننت RTL کمتر)

## پیامدها
SPA نیاز به مدیریت دقیق توکن در حافظه و بازیابی نشست دارد.
