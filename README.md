# 🎓 أكاديمية التعلم — Flutter + Firebase

منصة تعليمية متكاملة تعمل على Android وiOS، مع لوحة إدارة كاملة ودعم فيديوهات يوتيوب.

---

## 📱 الميزات

### للطلاب
- ✅ تسجيل دخول / إنشاء حساب / استعادة كلمة المرور
- ✅ عرض الكورسات المنشورة مع صور وتفاصيل
- ✅ تشغيل فيديوهات يوتيوب داخل التطبيق
- ✅ التسجيل في الكورسات ومتابعة التقدم
- ✅ تصفية بالتصنيف والمستوى
- ✅ واجهة عربية بالكامل (RTL)

### للمشرف (Admin)
- ✅ لوحة تحكم مع إحصائيات (كورسات، طلاب، دروس)
- ✅ إضافة / تعديل / حذف الكورسات
- ✅ إضافة دروس بفيديوهات يوتيوب
- ✅ نشر أو إخفاء الكورسات بزر واحد
- ✅ إدارة الطلاب المسجلين

---

## 🚀 خطوات الإعداد

### 1. تثبيت Flutter
```bash
# تأكد من تثبيت Flutter 3.x
flutter --version
```

### 2. إنشاء مشروع Firebase

1. اذهب إلى [console.firebase.google.com](https://console.firebase.google.com)
2. أنشئ مشروعاً جديداً
3. فعّل الخدمات التالية:
   - **Authentication** → Email/Password
   - **Firestore Database** → Start in test mode
   - **Storage** (اختياري)

### 3. ربط Firebase بالتطبيق

#### الطريقة الأسهل (FlutterFire CLI):
```bash
# تثبيت FlutterFire CLI
dart pub global activate flutterfire_cli

# في مجلد المشروع
flutterfire configure
```
هذا سيولّد ملف `lib/firebase_options.dart` تلقائياً.

#### الطريقة اليدوية:
1. في Firebase Console → Project Settings → Add App → Android
2. أضف package name: `com.example.eduapp`
3. حمّل `google-services.json` وضعه في `android/app/`
4. عدّل `lib/firebase_options.dart` بالبيانات من Console

### 4. إعداد قواعد Firestore

في Firebase Console → Firestore → Rules:
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // المستخدمون يقرأون بياناتهم فقط
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
      allow read: if request.auth != null && 
                     get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin';
    }
    
    // الكورسات: الجميع يقرأ المنشورة، المشرف يكتب
    match /courses/{courseId} {
      allow read: if resource.data.isPublished == true || 
                     (request.auth != null && 
                      get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin');
      allow write: if request.auth != null && 
                      get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin';
    }
    
    // الدروس: نفس الكورسات
    match /lessons/{lessonId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null && 
                      get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin';
    }
  }
}
```

### 5. إنشاء حساب المشرف

بعد تشغيل التطبيق، سجّل حساباً عادياً ثم في Firestore Console:
1. اذهب إلى collection `users`
2. افتح document المستخدم
3. غيّر `role` من `student` إلى `admin`

### 6. تشغيل التطبيق

```bash
# تثبيت dependencies
flutter pub get

# تشغيل
flutter run

# بناء APK
flutter build apk --release
# الملف في: build/app/outputs/flutter-apk/app-release.apk

# بناء APK للتوزيع المباشر (أصغر حجم)
flutter build apk --split-per-abi
```

---

## 🗂️ هيكل المشروع

```
lib/
├── main.dart                    # نقطة الدخول
├── firebase_options.dart        # إعدادات Firebase
├── models/
│   └── app_models.dart          # AppUser, Course, Lesson
├── services/
│   ├── auth_service.dart        # Firebase Auth
│   ├── course_service.dart      # Firestore CRUD
│   └── app_provider.dart        # State Management
├── screens/
│   ├── auth/
│   │   ├── login_screen.dart
│   │   ├── register_screen.dart
│   │   └── forgot_password_screen.dart
│   ├── student/
│   │   ├── home_screen.dart      # الرئيسية + استكشاف + تعلمي + حسابي
│   │   └── course_detail_screen.dart  # تفاصيل + مشغل يوتيوب
│   └── admin/
│       ├── admin_dashboard.dart  # لوحة التحكم الكاملة
│       ├── add_course_screen.dart
│       └── add_lesson_screen.dart
├── widgets/
│   ├── course_card.dart
│   ├── gradient_button.dart
│   ├── custom_text_field.dart
│   └── shimmer_loader.dart
└── utils/
    ├── app_theme.dart            # الألوان والـ Theme
    └── app_router.dart           # GoRouter
```

---

## 🎨 تخصيص الألوان

في `lib/utils/app_theme.dart`:
```dart
static const Color primary = Color(0xFF6C63FF);    // البنفسجي الرئيسي
static const Color secondary = Color(0xFFFF6584);   // الوردي
static const Color accent = Color(0xFF43E97B);      // الأخضر
static const Color background = Color(0xFF0F0E17);  // الخلفية الداكنة
```

---

## 📦 المكتبات المستخدمة

| المكتبة | الغرض |
|---------|-------|
| `firebase_auth` | تسجيل الدخول |
| `cloud_firestore` | قاعدة البيانات |
| `youtube_player_flutter` | تشغيل الفيديوهات |
| `go_router` | التنقل بين الشاشات |
| `provider` | إدارة الحالة |
| `google_fonts` | خط Cairo العربي |
| `cached_network_image` | تخزين الصور مؤقتاً |
| `shimmer` | تأثير التحميل |
| `percent_indicator` | شريط التقدم |

---

## ❓ مشاكل شائعة

**`google-services.json` غير موجود:**
→ حمّله من Firebase Console وضعه في `android/app/`

**خطأ `minSdkVersion`:**
→ تأكد أن `android/app/build.gradle` فيه `minSdkVersion 21`

**الفيديو لا يشتغل:**
→ تأكد من إضافة `INTERNET` permission في AndroidManifest.xml

**لا يظهر كمشرف:**
→ غيّر قيمة `role` في Firestore يدوياً إلى `admin`
