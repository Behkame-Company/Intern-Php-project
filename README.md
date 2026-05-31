# پروژه ارزیابی کارآموز

## سیستم احراز هویت کاربران (KYC System)

**مدت زمان:** 8 ساعت  
**سطح:** مبتدی–متوسط  
**فناوری‌ها:** HTML · PHP OOP · MVC · MySQL

---

## ۱. هدف پروژه

ساخت یک وب‌اپلیکیشن ساده برای ثبت‌نام، ورود کاربران، ارسال مدارک هویتی و تأیید/رد آن‌ها توسط ادمین.

---

## ۲. ساختار پوشه‌بندی (MVC)

کارآموز باید پروژه را **دقیقاً** با این ساختار تحویل دهد:

```
kyc-system/
├── app/
│   ├── controllers/
│   │   ├── AuthController.php
│   │   ├── UserController.php
│   │   └── AdminController.php
│   ├── models/
│   │   ├── User.php
│   │   └── KycRequest.php
│   └── views/
│       ├── auth/
│       │   ├── login.php
│       │   └── register.php
│       ├── user/
│       │   ├── dashboard.php
│       │   └── kyc_form.php
│       └── admin/
│           ├── dashboard.php
│           └── kyc_detail.php
├── core/
│   ├── Database.php        ← Singleton PDO wrapper
│   ├── Controller.php      ← Base controller
│   ├── Model.php           ← Base model
│   └── Router.php          ← Simple router
├── public/
│   ├── index.php           ← Entry point
│   └── uploads/
│       └── documents/      ← محل ذخیره عکس‌ها
├── config/
│   └── config.php          ← DB credentials, constants
├── database/
│   └── schema.sql          ← فایل SQL ساخت جداول
└── README.md
```

---

## ۳. دیتابیس

کارآموز باید فایل `schema.sql` را بنویسد که شامل این جداول باشد:

### جدول `users`
| فیلد | نوع | توضیح |
|---|---|---|
| id | INT PK AUTO_INCREMENT | |
| full_name | VARCHAR(100) | نام کامل |
| email | VARCHAR(150) UNIQUE | |
| password | VARCHAR(255) | hashed |
| role | ENUM('user','admin') | پیش‌فرض: user |
| created_at | TIMESTAMP | |

### جدول `kyc_requests`
| فیلد | نوع | توضیح |
|---|---|---|
| id | INT PK AUTO_INCREMENT | |
| user_id | INT FK → users.id | |
| national_code | VARCHAR(10) | کد ملی |
| birth_date | DATE | تاریخ تولد |
| address | TEXT | آدرس |
| document_path | VARCHAR(255) | مسیر فایل آپلود شده |
| status | ENUM('not_started','pending','approved','rejected') | پیش‌فرض: not_started |
| admin_note | TEXT NULL | توضیح رد یا قبول |
| submitted_at | TIMESTAMP | |
| reviewed_at | TIMESTAMP NULL | |

---

## ۴. قابلیت‌های مورد نیاز

### ۴.۱ بخش کاربر

- **ثبت‌نام** (`/register`): فرم با فیلدهای نام، ایمیل، پسورد، تکرار پسورد + validation
- **ورود** (`/login`): احراز هویت با ایمیل و پسورد + session
- **خروج** (`/logout`): destroy session
- **داشبورد کاربر** (`/dashboard`): نمایش وضعیت فعلی KYC (not_started / pending / approved / rejected)
- **فرم KYC** (`/kyc/submit`):
  - فیلدهای: کد ملی، تاریخ تولد، آدرس
  - آپلود عکس شناسنامه (فقط jpg/png/pdf، حداکثر ۲ مگابایت)
  - اگر قبلاً ارسال کرده: نمایش وضعیت به جای فرم

### ۴.۲ بخش ادمین

- **ورود ادمین** (`/admin/login`): صفحه لاگین مجزا
- **داشبورد ادمین** (`/admin/dashboard`): لیست تمام درخواست‌های KYC با وضعیت‌شان + فیلتر بر اساس وضعیت
- **جزئیات درخواست** (`/admin/kyc/{id}`):
  - نمایش اطلاعات کاربر و عکس آپلودشده
  - دکمه‌های تأیید و رد + فیلد توضیح (admin_note)

---

## ۵. الزامات فنی (معیارهای ارزیابی)

### ✅ PHP OOP
- [ ] هر Model یک کلاس جداگانه با متدهای مرتبط دارد
- [ ] استفاده از `__construct` برای تزریق وابستگی
- [ ] هیچ query مستقیمی داخل Controller نوشته نشده
- [ ] استفاده از `static` یا `Singleton` برای Database connection

### ✅ MVC
- [ ] هیچ HTML داخل Controller وجود ندارد
- [ ] هیچ منطق تجاری داخل View وجود ندارد
- [ ] هر Controller فقط مسئول یک بخش است
- [ ] Base Controller وجود دارد و از آن ارث‌بری شده

### ✅ امنیت
- [ ] پسورد با `password_hash()` ذخیره شده
- [ ] ورودی‌ها با `htmlspecialchars()` یا PDO Prepared Statements sanitize شده
- [ ] چک‌های Session برای صفحات محافظت‌شده
- [ ] ادمین نمی‌تواند به بخش کاربر برود و بالعکس

### ✅ دیتابیس
- [ ] تمام queryها از PDO Prepared Statements استفاده می‌کنند (هیچ query رشته‌ای مستقیم نباشد)
- [ ] Foreign Key درست تعریف شده
- [ ] فایل `schema.sql` قابل اجرا در phpMyAdmin باشد

### ✅ آپلود فایل
- [ ] اعتبارسنجی نوع فایل (MIME type، نه فقط پسوند)
- [ ] rename فایل با نام یکتا (مثلاً `uniqid()`)
- [ ] ذخیره در پوشه `public/uploads/documents/`

### ✅ HTML/UI
- [ ] فرم‌ها اعتبارسنجی سمت سرور دارند و خطاها نمایش داده می‌شوند
- [ ] نمایش وضعیت KYC با رنگ‌بندی مناسب (not_started=خاکستری، pending=زرد، approved=سبز، rejected=قرمز)

---

## ۶. نکات اضافی (امتیاز بیشتر)

این موارد اجباری نیستند اما نشان‌دهنده درک عمیق‌تر هستند:

- نوشتن `Router.php` که URL را parse کند و به Controller مناسب هدایت کند
- استفاده از `.htaccess` برای پنهان کردن `index.php` از URL
- پیام flash (نمایش یک‌بار با session)

---

## ۷. تحویل پروژه

کارآموز باید موارد زیر را تحویل دهد:

1. کد پروژه به صورت ZIP   
2. فایل `database/schema.sql`
3. فایل `README.md` شامل:
   - مراحل نصب و راه‌اندازی
   - اطلاعات ورود ادمین پیش‌فرض
   - توضیح کوتاه معماری انتخابی

---


## 8. جدول زمانی پیشنهادی (۸ ساعت)

| بازه | کار |
|---|---|
| ساعت 0-1 | ساختار پروژه، config، Database، schema.sql |
| ساعت 1-3 | ثبت‌نام و ورود کاربر (Model + Controller + View) |
| ساعت 3-5 | داشبورد کاربر + فرم KYC + آپلود فایل |
| ساعت 5-6 | پنل ادمین (لاگین + لیست درخواست‌ها) |
| ساعت 6-7 | صفحه جزئیات ادمین + تأیید/رد |
| ساعت 7-8 | امنیت، validation، بهبود |
