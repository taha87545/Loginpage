# میزان — Mizan

وب‌اپلیکیشن مدیریت درآمد و هزینه‌های شخصی؛ فارسی، راست‌به‌چپ، امن، کاملاً واکنش‌گرا، با **دارک‌مود** — روی یک تک‌فایل فرانت‌اند.

## معماری

| لایه | تکنولوژی |
|---|---|
| فرانت‌اند | **یک** `index.html` + **یک** `app.css` + **یک** `app.js` (HTML5/CSS3/Vanilla JS) + Chart.js 4.4.7 + وزیرمتن — همه محلی، بدون CDN |
| بک‌اند | Next.js 16 Route Handlers (معماری نشست‌محور) |
| دیتابیس | PostgreSQL + Drizzle ORM (کوئری پارامتری) |
| احراز هویت | نشست سروری + کوکی HttpOnly + scrypt + CSRF double-submit + محدودسازی ورود |

تک‌فایل فرانت‌اند هر سه نما را نگه می‌دارد: `#viewHome` (لندینگ)، `#authView` (ورود/ثبت‌نام)، `#appView` (داشبورد). روتینگ سمت کلاینت با History API انجام می‌شود.

## مسیرها

| مسیر | توضیح |
|---|---|
| `/` | **صفحه اصلی (خانه)** — پیش‌فرض؛ لندینگ محصول |
| `/app` | اپلیکیشن (ورود → داشبورد) — همان تک‌فایل با نمای اپ |
| `/home` | ریدایرکت دائمی ۳۰۸ به `/` |
| `/api/*` | REST API |

## ویژگی‌های کلیدی UI

- **دارک‌مود** کامل (توکن‌محور، ذخیره در localStorage، هماهنگ با `prefers-color-scheme`، نمودارهای سازگار با تم)
- **شِل دسکتاپ**: سایدبار ثابت در عرض ≥۱۱۰۰px (برند، ناوبری + scrollspy، کاربر، تم) و تاپ‌بار شیشه‌ای در تبلت/موبایل
- جدول تراکنش‌ها در موبایل به **کارت** تبدیل می‌شود؛ در سایز میانی اسکرول نرم افقی
- تاریخ شمسی واقعی (ماه/سال)، واحد پول تومان/ریال، وضعیت خالی، اسکلت بارگذاری، Toast، مودال دسترس‌پذیر

## اجرا

```bash
npm install
npx drizzle-kit push
npm run dev                    # توسعه  → http://localhost:3000
npm run build && npm start     # production
```

پیش‌نیاز: `DATABASE_URL` در `.env`.

## تست

```bash
node scripts/test-api.mjs      # ۶۹ تست پذیرش API (سرور روی :3000)
node scripts/browser-qa.mjs    # تست مرورگر واقعی: دسکتاپ/تبلت/موبایل، لایت/دارک
```

## API — خلاصه

`POST /api/register` · `POST /api/login` · `POST /api/logout` · `GET /api/me` · `GET|POST /api/transactions` · `GET|PUT|DELETE /api/transactions/{id}` · `GET /api/dashboard?period=` · `GET /api/charts/expenses` · `GET /api/charts/cashflow`

درخواست‌های تغییردهنده به هدر `X-CSRF-Token` نیاز دارند. خطاها: `{error:{code,message,fields?}}`.

## production

- پشت HTTPS → کوکی `Secure` خودکار فعال می‌شود (تشخیص از `x-forwarded-proto`).
- هدرهای امنیتی + CSP سخت‌گیرانه (فقط `self`) از `next.config.ts` اعمال می‌شود.
- rate-limiter درون‌حافظه‌ای است؛ در مقیاس چند نمونه‌ای به Redis منتقل شود.
- CORS غیرفعال (same-origin کافی است).
