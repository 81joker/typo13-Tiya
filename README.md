## 🧩 لماذا نكتب `10.` بدلًا من مسار واحد فقط؟

لأن TYPO3 يسمح بوجود عدة مسارات، مثلاً:

- **رقم 10** = مسار من الإضافة (EXT)  
- **رقم 20** = مسار مخصّص من الموقع (Site)  
- **رقم 30** = مسار مخصص للمطور  

ويستخدم TYPO3 هذه المسارات لتحديد **الأولوية** بحيث يتم استخدام أصغر رقم أولًا.


##############################

## 🔧 شرح جزء الـ dataProcessing في TypoScript

يُستخدم `dataProcessing` داخل **FLUIDTEMPLATE** لمعالجة بيانات معيّنة وتمريرها إلى قالب Fluid ليصبح استخدامها أسهل داخل ملفات القوالب.

---

## 1️⃣ SiteProcessor

```typoscript
1 = TYPO3\CMS\Frontend\DataProcessing\SiteProcessor
1.as = site

##### very important
#### ddev exec vendor/bin/typo3 backend:user:create
#### ddev exec vendor/bin/typo3 cache:flush
### ddev exec mysql -e "SELECT uid, username, admin FROM be_users;"


ddev exec vendor/bin/typo3 cache:flush
ddev exec rm -rf var/cache/*
ddev exec vendor/bin/typo3 cache:warmup



## to reset password
## ddev exec vendor/bin/typo3 backend:resetpassword https://typo3-12.ddev.site tim26618@gmail.com
### ddev exec touch var/transient/ENABLE_INSTALL_TOOL

ddev exec backend:resetpassword tim26618@gmail.com





'content-text' - أيقونة نص
'actions-calendar' - أيقونة تقويم (مناسبة للأحداث)
'actions-document-new' - أيقونة مستند جديد
'content-elements-mailform' - أيقونة نموذج
'actions-star' - أيقونة نجمة