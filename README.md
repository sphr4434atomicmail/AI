# دستیار هوش مصنوعی شخصی تلگرام

باتی خصوصی و مبتنی بر Python برای گفت‌وگوی فارسی/چندزبانه، جست‌وجوی وب همراه پیوند منبع، دریافت تصویر و پیام صوتی، خلاصه‌سازی سند، و یادآوری‌های زمان‌دار. بات با polling کار می‌کند و نیازی به دامنه یا webhook ندارد. دسترسی به پیام‌ها در سطح برنامه به شناسهٔ عددی یک کاربر محدود است.

## امکانات

- گفت‌وگوی متنی با OpenRouter و نگه‌داری تاریخچهٔ کوتاه در SQLite؛ `/clear` تاریخچه را پاک می‌کند.
- جست‌وجوی خودکار وب از طریق ابزار مدل و Tavily. کاربر می‌تواند `/search عبارت` یا «جست‌وجو کن: عبارت» بنویسد. پاسخ نهایی پیوند منابع را درج می‌کند.
- تحلیل عکس و پیام صوتی با مدل‌های OpenRouter سازگار با ورودی چندوجهی.
- استخراج متن از PDF، DOCX، TXT، MD، CSV، JSON و فایل‌های متنی متداول.
- یادآوری پایدار در SQLite. منطقهٔ زمانی پیش‌فرض `Asia/Tehran` است.

> برای پردازش عکس و صوت، مدلی را انتخاب کنید که در OpenRouter ورودی همان نوع داده را پشتیبانی کند. انتخاب مدل ممکن است هزینه داشته باشد؛ هزینه و محدودیت‌ها را در پنل ارائه‌دهنده بررسی کنید. جست‌وجوی وب نیازمند کلید Tavily است.

## فرمان‌ها

- `/start` یا `/help` — راهنما
- `/clear` — پاک‌کردن حافظهٔ گفت‌وگو
- `/search عبارت` — جست‌وجوی وب
- `/remind 30m تماس با ...` — یادآوری پس از مدت مشخص؛ واحدها `s`، `m`، `h` و `d` هستند.
- `/remind 2026-10-11 14:30 تماس با ...` — یادآوری در زمان محلی ایران با قالب ۲۴ساعته.

## آماده‌سازی تلگرام

1. در Telegram با `@BotFather`، اگر هنوز انجام نشده، یک bot بساز و token آن را بگیر.
2. شناسهٔ عددی کاربر را از یک ابزار قابل‌اعتماد مانند `@userinfobot` بردار.
3. token را در گفت‌وگو، GitHub یا فایل‌های commit‌شده قرار نده. فقط در تنظیمات محرمانهٔ Railway واردش کن.

## متغیرهای محیطی

این مقادیر را در Railway → Service → Variables بساز:

- `TELEGRAM_BOT_TOKEN` — توکن BotFather.
- `TELEGRAM_ALLOWED_USER_ID` — شناسهٔ عددی شخص مجاز؛ فقط همین حساب پردازش می‌شود.
- `OPENROUTER_API_KEY` — کلید API حساب OpenRouter.
- `OPENROUTER_MODEL` — مدل گفت‌وگو؛ پیش‌فرض `google/gemini-2.5-flash`.
- `OPENROUTER_AUDIO_MODEL` — مدل ورودی صوت؛ پیش‌فرض همان مدل گفت‌وگو.
- `TAVILY_API_KEY` — کلید Tavily برای جست‌وجوی وب؛ بدون آن جست‌وجو غیرفعال است.
- `TIMEZONE` — پیش‌فرض `Asia/Tehran`.
- `BOT_DATA_DIR` — مسیر داده، در Railway مقدار `/data`.

## استقرار روی Railway از GitHub

1. یک مخزن خصوصیِ خالی در GitHub بساز و این بسته را روی رایانه‌ات از حالت فشرده خارج کن. در پوشهٔ پروژه این فرمان‌ها را اجرا کن و نشانی مخزن خودت را جایگزین کن. هرگز `.env` واقعی را commit نکن:

   ```bash
   git init -b main
   git add .
   git commit -m "Initial personal Telegram AI assistant"
   git remote add origin https://github.com/YOUR-USERNAME/YOUR-REPOSITORY.git
   git push -u origin main
   ```
2. در Railway یک Project بساز و گزینهٔ Deploy from GitHub Repo را انتخاب کن. به Railway اجازهٔ دسترسی به همان مخزن را بده.
3. بات یک **worker service** است، نه وب‌سایت؛ دستور اجرا `python bot.py` است. فایل `Procfile` همین worker process را تعریف می‌کند.
4. در تنظیمات سرویس، متغیرهای بالا را وارد کن.
5. برای نگه‌داری تاریخچه و یادآوری‌ها پس از redeploy، یک Railway Volume به مسیر `/data` متصل کن و `BOT_DATA_DIR=/data` بگذار. بدون Volume، فایل SQLite ممکن است هنگام redeploy از بین برود.
6. سرویس را Deploy کن و در Logs بررسی کن که پیام `Telegram bot connected as @...` دیده شود.
7. در Telegram، به بات `/start` بفرست و گفت‌وگو، `/search` و یک یادآوری کوتاه را آزمایش کن.

Railway ممکن است بسته به طرح/مصرف و افزونه‌های انتخابی هزینه داشته باشد. OpenRouter و Tavily نیز بر اساس طرح خودشان محدودیت یا هزینه دارند.

## اجرای محلی برای توسعه

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
# مقادیر واقعی را فقط در .env محلی وارد کنید؛ این فایل در .gitignore است.
python bot.py
```

## نکات حریم خصوصی و امنیت

- پیش از پردازش محتوا، شناسهٔ تلگرامی با `TELEGRAM_ALLOWED_USER_ID` مقایسه می‌شود؛ گروه‌ها نادیده گرفته می‌شوند.
- پیام‌های متنی اخیر و یادآوری‌ها در SQLite ذخیره می‌شوند. برای پاک‌کردن مکالمه از `/clear` استفاده کنید؛ یادآوری‌ها جداگانه باقی می‌مانند.
- فایل‌های ورودی برای پردازش به OpenRouter فرستاده می‌شوند. متن جست‌وجو برای جست‌وجو به Tavily فرستاده می‌شود؛ سیاست‌های نگه‌داری دادهٔ این سرویس‌ها را بررسی کنید.
- برای پاک‌کردن همهٔ داده‌ها، سرویس را متوقف و فایل `assistant.sqlite3` را از Volume حذف کنید.
- برای جلوگیری از نشت کلید، logها مقدار متغیرهای محرمانه را چاپ نمی‌کنند. اگر کلیدی افشا شد، آن را فوراً در همان سرویس لغو و دوباره صادر کنید.

## منابع فنی

- [Telegram Bot API: `getUpdates`](https://core.telegram.org/bots/api#getupdates)
- [OpenRouter Chat Completions](https://openrouter.ai/docs/api/api-reference/chat/create-a-chat-completion)
- [OpenRouter audio input and output](https://openrouter.ai/docs/guides/overview/multimodal/audio)
- [Railway services](https://docs.railway.com/services)
