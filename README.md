# واصل (Wasel) — تطبيق تواصل اجتماعي

بُني بنفس أسلوب مشروع أواب: HTML/CSS/JS خام + Firebase (Auth + Realtime Database).

## الصفحات
- `login.html` — تسجيل الدخول (بريد/هاتف + كلمة سر)
- `signup.html` — إنشاء حساب (3 خطوات: البيانات الأساسية + السن 18+ → صورة البروفايل → تأكيد)
- `home.html` — الرئيسية: هيدر (لوجو، رسائل، إشعارات، بحث، بروفايل) + صندوق نشر + الفييد
- `search.html` — صفحة البحث نفسها (بحث سابق)
- `search-results.html` — صفحة نتائج البحث لوحدها
- `profile.html` — صورة البروفايل + الاسم + تبويبات: المنشورات / المحفوظات / الإعدادات

## خطوات التشغيل
1. افتح `js/firebase-config.js` وحط بيانات مشروعك من Firebase Console.
2. من Authentication فعّل **Email/Password**.
3. من Realtime Database اعمل الـ rules دي كبداية (عدّلها حسب حاجتك):

```json
{
  "rules": {
    "users": {
      ".read": "auth != null",
      "$uid": { ".write": "auth != null && auth.uid === $uid" }
    },
    "posts": {
      ".read": "auth != null",
      ".indexOn": ["createdAt", "uid"],
      "$postId": {
        ".write": "auth != null && (!data.exists() || data.child('uid').val() === auth.uid || newData.child('uid').val() === auth.uid)"
      }
    },
    "comments": {
      "$postId": {
        ".read": "auth != null",
        ".indexOn": ["createdAt"],
        ".write": "auth != null"
      }
    },
    "saves": {
      "$uid": { ".read": "auth != null && auth.uid === $uid", ".write": "auth != null && auth.uid === $uid" }
    },
    "notifications": {
      "$uid": { ".read": "auth != null && auth.uid === $uid", ".write": "auth != null" }
    },
    "recentSearches": {
      "$uid": { ".read": "auth != null && auth.uid === $uid", ".write": "auth != null && auth.uid === $uid" }
    },
    "friendRequests": {
      "$uid": {
        ".read": "auth != null && auth.uid === $uid",
        ".indexOn": ["createdAt"],
        "$fromUid": { ".write": "auth != null && (auth.uid === $uid || auth.uid === $fromUid)" }
      }
    },
    "sentFriendRequests": {
      "$uid": {
        ".read": "auth != null && auth.uid === $uid",
        "$otherUid": { ".write": "auth != null && (auth.uid === $uid || auth.uid === $otherUid)" }
      }
    },
    "friends": {
      "$uid": {
        ".read": "auth != null",
        "$otherUid": { ".write": "auth != null && (auth.uid === $uid || auth.uid === $otherUid)" }
      }
    },
    "follows": {
      "$uid": {
        ".read": "auth != null",
        "following": { "$otherUid": { ".write": "auth != null && auth.uid === $uid" } },
        "followers": { "$otherUid": { ".write": "auth != null && auth.uid === $otherUid" } }
      }
    },
    "userChats": {
      "$uid": {
        ".read": "auth != null && auth.uid === $uid",
        ".indexOn": ["lastMessageAt"],
        "$chatId": { ".write": "auth != null && $chatId.contains(auth.uid)" }
      }
    },
    "chats": {
      "$chatId": {
        ".read": "auth != null && $chatId.contains(auth.uid)",
        ".write": "auth != null && $chatId.contains(auth.uid)",
        "messages": { ".indexOn": ["createdAt"] }
      }
    },
    "config": {
      ".read": "auth != null",
      ".write": false
    }
  }
}
```

**مهم:** الـ Rules دي لازم تتنسخ **كاملة** كما هي وتتحط في Firebase Console → Realtime Database → Rules، وتدوس **Publish** فوق. لو الجزء بتاعك ناقص أي section من دول (زي `friendRequests` أو `chats` أو `follows`) هيفضل أي فيتشر بيستخدمها يديك "حصلت مشكلة" أو "مفيش نتائج" بصمت، لأن Firebase بيرفض أي مسار مش متعرّف له صلاحية صراحةً.

4. ارفع الفولدر كله على استضافة استاتيك (Firebase Hosting / أي سيرفر).

## ملاحظات عن قاعدة البيانات الضعيفة حاليًا
- صور البروفايل والمنشورات بتتضغط تلقائيًا (canvas resize + JPEG quality) وتتخزن كـ base64 جوه Realtime Database بدل Storage.
- في حد أقصى ~350KB لصورة المنشور بعد الضغط (`MAX_POST_IMAGE_BYTES` في `js/common.js`) — لو حبيت تزوده أو تنقله لـ Firebase Storage لاحقًا، ده مكان التعديل.

## أفكار إضافية (اختياري) لو حبيت تضيفها بعدين
- صفحة الرسائل (Chat) بنفس أسلوب الإشعارات اللحظي.
- صفحة إشعارات كاملة (دلوقتي فيه عداد بس في الهيدر، والبيانات بتتسجل في `notifications/{uid}`).
- ربط زر "تعديل البيانات الشخصية" في الإعدادات بفورم تعديل فعلي.
- Cloud Function لضغط الصور على السيرفر بدل المتصفح لو حبيت تقلل الحمل على قاعدة البيانات أكتر.
- حماية `submitExamResult`-style للمنشورات: نقل شرط الإنشاء والتحقق من العمر لقاعدة بيانات/Cloud Function بدل الاعتماد على الفرونت إند بس.
