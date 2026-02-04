# TYPO3 12 - حل مشكلة Rate Limiting (قفل تسجيل الدخول)

## المشكلة
عند محاولة تسجيل الدخول إلى TYPO3 Backend، تظهر رسالة الخطأ: 
```
#1616175867 TYPO3\CMS\Core\RateLimiter\RequestRateLimitedException
The login is locked until [DATE] due to too many failed login attempts from your IP address.
```

---

## الحلول

### الحل 1: الانتظار حتى انتهاء مدة القفل
انتظر حتى ينتهي وقت القفل المحدد في رسالة الخطأ (عادة 5 دقائق حسب الإعدادات الافتراضية).

---

### الحل 2: الدخول إلى Install Tool وإعادة تعيين كلمة المرور

#### الخطوة 1: تفعيل Install Tool بدون كلمة مرور
```bash
ddev exec touch var/transient/ENABLE_INSTALL_TOOL
```

#### الخطوة 2: فتح Install Tool في المتصفح
```
https://typo3-12. ddev.site/typo3/install. php
```

#### الخطوة 3: إنشاء مستخدم إداري جديد
1. اذهب إلى **Maintenance** في القائمة اليسرى
2. اضغط على **"Create Administrator"** في بطاقة "Create Administrative User"
3. أدخل البيانات التالية:
   - Username: اسم المستخدم
   - Email: البريد الإلكتروني
   - Password: كلمة مرور قوية
4. اضغط **Create Administrator**

#### الخطوة 4: تغيير كلمة مرور Install Tool (اختياري)
1. اذهب إلى **Settings** في القائمة اليسرى
2. ابحث عن **"Change Install Tool Password"**
3. أدخل كلمة مرور جديدة وقوية
4. احفظ التغييرات

---

### الحل 3: إعادة تعيين كلمة مرور مستخدم موجود

#### الطريقة الأولى: عبر Command Line
```bash
ddev exec vendor/bin/typo3 backend:resetpassword https://typo3-12.ddev.site your-email@gmail.com
```

**ملاحظة:** رسائل البريد الإلكتروني في بيئة DDEV لا تُرسل للبريد الحقيقي، بل تُحفظ في **MailHog**. 

#### الوصول إلى MailHog لقراءة رسالة إعادة تعيين كلمة المرور: 
```
https://typo3-12.ddev.site:8026
```
أو: 
```
http://localhost:8026
```

---

### الحل 4: تعديل إعدادات Rate Limiting

#### الطريقة الأولى:  عبر Install Tool
1. افتح Install Tool:  `https://typo3-12.ddev.site/typo3/install.php`
2. اذهب إلى **Settings** → **Configuration Presets**
3. ابحث عن **Backend** → **loginRateLimit**
4. غيّر القيمة من `20` إلى رقم أكبر (مثل `100`) أو `0` لتعطيل الحد
5. احفظ التغييرات

#### الطريقة الثانية: تعديل ملف LocalConfiguration.php مباشرة
افتح الملف:  `config/system/settings.php` أو `typo3conf/LocalConfiguration.php`

ابحث عن:
```php
'BE' => [
    'loginRateLimit' => 20,
    'loginLockTime' => 300,
],
```

غيّر القيم إلى:
```php
'BE' => [
    'loginRateLimit' => 100,  // عدد المحاولات المسموحة
    'loginLockTime' => 60,    // مدة القفل بالثواني (1 دقيقة بدلاً من 5)
],
```

---

## تغيير كلمة مرور Install Tool يدوياً

### الخطوة 1: إنشاء Hash لكلمة المرور الجديدة
```bash
ddev exec php -r "echo password_hash('كلمة_المرور_الجديدة', PASSWORD_ARGON2I);"
```

مثال:
```bash
ddev exec php -r "echo password_hash('mynewpassword', PASSWORD_ARGON2I);"
```

### الخطوة 2: نسخ الـ Hash الناتج
سيظهر مثل:
```
$argon2i$v=19$m=65536,t=4,p=1$N0M0UlVKTW1lMy5GVmJyVA$1EqewOPv0cCjDRiCSJYsFQtPjecMZMcwWHfRWLWoS/8
```

### الخطوة 3: تعديل ملف الإعدادات
افتح الملف: `config/system/settings.php`

ابحث عن: 
```php
'installToolPassword' => '$argon2i$.. .',
```

استبدل القيمة القديمة بالـ Hash الجديد: 
```php
'installToolPassword' => '$argon2i$v=19$m=65536,t=4,p=1$N0M0UlVKTW1lMy5GVmJyVA$1EqewOPv0cCjDRiCSJYsFQtPjecMZMcwWHfRWLWoS/8',
```

---

## نصائح إضافية

### تعطيل Rate Limiting مؤقتاً في بيئة التطوير
في ملف الإعدادات، ضع:
```php
'BE' => [
    'loginRateLimit' => 0,  // 0 = تعطيل الحد تماماً
],
```

**تحذير:** لا تفعل هذا في بيئة الإنتاج (Production)!

### مسح الكاش بعد أي تعديل
```bash
ddev exec vendor/bin/typo3 cache:flush
```

أو من Install Tool:
1. اذهب إلى **Maintenance**
2. اضغط **"Flush cache"**

---

## روابط مفيدة

- **Backend Login**: `https://typo3-12.ddev.site/typo3`
- **Install Tool**: `https://typo3-12.ddev.site/typo3/install.php`
- **MailHog (البريد المحلي)**: `https://typo3-12.ddev. site:8026`
- **Frontend**:  `https://typo3-12.ddev.site`

---

## استكشاف الأخطاء

### إذا لم يعمل ENABLE_INSTALL_TOOL
تحقق من أن المسار صحيح:
```bash
ddev exec ls -la var/transient/
```

### إذا لم تستلم بريد إعادة تعيين كلمة المرور
تحقق من MailHog:
```
https://typo3-12.ddev.site:8026
```

### إذا ظهرت أخطاء في Cache
امسح الكاش:
```bash
ddev exec rm -rf var/cache/*
ddev exec vendor/bin/typo3 cache:flush
```

---

## الأوامر المفيدة

```bash
# بدء DDEV
ddev start

# إيقاف DDEV
ddev stop

# الدخول إلى Container
ddev ssh

# عرض قاعدة البيانات
ddev describe

# إعادة بناء المشروع
ddev restart

# مسح الكاش
ddev exec vendor/bin/typo3 cache:flush

# تفعيل Install Tool
ddev exec touch var/transient/ENABLE_INSTALL_TOOL
```

---

## المؤلف
تم إنشاء هذا الدليل لحل مشاكل Rate Limiting في TYPO3 12 مع بيئة DDEV. 

التاريخ: 2026-01-15


ddev ssh
php -r "echo password_hash('@Password1', PASSWORD_DEFAULT);"
