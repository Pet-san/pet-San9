# PROJECT_REFERENCE — مرجع معمارية قالب المتجر الإلكتروني

> **إلى أي نموذج ذكاء اصطناعي يقرأ هذا الملف:**
> هذا الملف يصف المشروع **كما هو في الكود الحالي**. اعتمد عليه لفهم الهيكلة وتدفق البيانات وتحديد الملفات الواجب تعديلها، ولا تطلب من المستخدم رفع كل الملفات إلا إذا احتجت فعلاً لرؤية كود دالة محددة (اذكر اسم الملف والدالة).
> أسماء الدوال والمتغيرات والملفات في هذا المرجع مطابقة للكود. إذا تعارض هذا الملف مع كود يراه المستخدم لك، فالكود هو الصحيح، ونبّه المستخدم أن المرجع يحتاج تحديثاً.

- **آخر تحديث للمرجع:** 2026-09-30 (بعد إصلاحات: تهريب النصوص، منتقي أيقونة القسم، تقييد كميات السلة، حارس الاستعلامات، توحيد `?v=41`، وقواعد Firebase)
- **ملاحظة أمنية:** مفاتيح Firebase وImgBB وImageKit موجودة في `js/config.js`. لا تنسخها إلى أي مستند أو محادثة. ارجع لهذا الملف عند الحاجة فقط.

---

## 1) ما هو المشروع

قالب متجر إلكتروني **عربي (RTL)** يُستنسخ لكل عميل ثم يُعدَّل (ملابس، مواد كهربائية، كوزمتك...). لا يوجد دفع إلكتروني: الزبون يتصفح، يضيف للسلة، يدخل بيانات التوصيل في نافذة منبثقة، ثم تُفتح محادثة **واتساب** برسالة طلب جاهزة. الطلب يُسجَّل أيضاً في Firebase لتراه الإدارة.

**التقنيات:**
- HTML + CSS + JavaScript خام (Vanilla). لا React ولا أدوات بناء ولا `import/export`.
- كل السكربتات تُحمَّل بوسوم `<script>` عادية وتتشارك **النطاق العام (global scope)**.
- **Firebase Realtime Database** (قاعدة البيانات) + **Firebase Authentication** (دخول الأدمن) عبر نسخة `compat` رقم `10.12.2` من CDN.
- **localStorage** كنسخة محلية/كاش لكل البيانات.
- **ImgBB** لرفع الصور + **ImageKit** كـ CDN لتصغير الصور (اختياري).
- بدون خادم (Backend) خاص بنا: الواجهة تتحدث مباشرة مع Firebase، لذلك **الأمان يعتمد على Firebase Rules** (انظر القسم 10).

---

## 2) شجرة الملفات

> ملاحظة: قد تُرفع الملفات أحياناً في مجلد واحد مسطح، لكن ملفات HTML تشير إلى المسارات التالية، وهذه هي البنية الصحيحة على الاستضافة.

```
project-root/
├── firebase-rules.json   قواعد Firebase المقترحة (تُلصق في لوحة Firebase، ليست جزءاً من الموقع المرفوع)
├── index.html        الرئيسية (سلايدر الإعلانات، الأقسام، مميزة/عروض/جديد، مميزات، تواصل)
├── products.html     المتجر: بحث، ترتيب، تبويبات أقسام، تفرعات، تحميل لانهائي
├── product.html      تفاصيل منتج: لون/مقاس/كمية، إضافة للسلة، طلب واتساب مباشر
├── categories.html   بطاقات الأقسام الرئيسية مع رقائق التفرعات
├── cart.html         السلة وملخص الطلب وزر الطلب عبر واتساب
├── about.html        من نحن
├── contact.html      تواصل معنا (بطاقات ديناميكية من الإعدادات)
├── login.html        دخول الأدمن (Firebase Auth)
├── admin.html        لوحة التحكم (6 تبويبات + 3 نوافذ منبثقة)
├── css/
│   └── style.css     ملف التنسيق الوحيد (~383 سطراً، أغلبه مضغوط)
├── js/
│   ├── config.js     STORE_CONFIG: الإعدادات الأولية ومفاتيح الخدمات
│   ├── icons.js      ICONS + iconSvg(key) + CATEGORY_ICON_KEYS
│   ├── store.js      طبقة البيانات: كائن Store + Firebase + localStorage
│   ├── whatsapp.js   formatPrice + نافذة التوصيل + بناء رسائل واتساب + تسجيل الطلب
│   ├── app.js        escapeHtml/jsStr + الهيدر والفوتر والقائمة الجانبية والبحث العام وعداد السلة والتوست
│   ├── products.js   المتجر (products.html) + تفاصيل المنتج (product.html) + بطاقة المنتج
│   ├── cart.js       صفحة السلة فقط
│   ├── auth.js       حماية الأدمن + صفحة الدخول
│   └── admin.js      كل منطق لوحة التحكم + رفع الصور
└── assets/logo/logo.png   الشعار (مستخدم في الهيدر والفوتر والأيقونة المصغرة وصفحات الدخول والأدمن)
```

---

## 3) ما الذي يُحمَّل في كل صفحة (ترتيب السكربتات حرج)

الترتيب الأساسي دائماً: `config` ← `icons` ← Firebase SDKs ← `store` ← `whatsapp` ← `app` ← سكربت الصفحة.
السبب: `store.js` يقرأ `STORE_CONFIG` ويستخدم Firebase عند التحميل، و`whatsapp.js` يستخدم `Store` و`iconSvg`، و`app.js` يستخدم الثلاثة.

| الصفحة | سكربتات إضافية بعد `app.js` | Firebase SDKs | ملاحظات |
|---|---|---|---|
| `index.html` | لا شيء (سكربت inline طويل) | app + database | **لا يحمّل `products.js`**؛ عنده نسخ مكررة من دوال البطاقة |
| `products.html` | `products.js` | app + database | |
| `product.html` | `products.js` | app + database | |
| `categories.html` | لا شيء (inline) | app + database | |
| `cart.html` | `cart.js` | app + database | |
| `about.html` / `contact.html` | لا شيء (inline) | app + database | |
| `login.html` | `auth.js` | app + **auth** + database | |
| `admin.html` | `auth.js` ثم `admin.js` | app + **auth** + database | |

يجب إضافة `firebase-auth-compat.js` فقط في صفحتي `login.html` و`admin.html`.

---

## 4) مخطط تدفق البيانات

```
config.js (STORE_CONFIG)
   │  قيم أولية فقط (تُنسخ مرة واحدة إلى الإعدادات المحلية عبر seedIfNeeded)
   ▼
store.js ── Store API ──► localStorage (ws_*)  ◄── الواجهة تقرأ من هنا دائماً
   │                          ▲
   │ syncNodeToFirebase       │ pullFromFirebase / fetch*FromFirebase / _cacheLoadedProducts
   ▼                          │
Firebase Realtime DB (ws_categories, ws_products, ws_settings, ws_ads, ws_orders)
```

**القاعدة الذهبية:** الصفحات لا تلمس `localStorage` ولا `firebase` مباشرة (باستثناء استثناءات موثقة في القسم 12). كل قراءة/كتابة تمر عبر `Store`.

- **القراءة (الواجهة العامة):** `Store.getX()` تقرأ من localStorage فوراً (متزامنة). التحديث من Firebase يتم في الخلفية ثم يُطلق حدث `store:synced` لإعادة الرسم.
- **الكتابة (الأدمن):** `Store.saveX()` تكتب في localStorage ثم `syncNodeToFirebase(node, data)` تستبدل العقدة كاملة في Firebase (`set`). الاستثناء: الطلبات تُضاف بـ `push` كعنصر مستقل.
- إذا كان `STORE_CONFIG.firebaseDatabaseURL` فارغاً أو Firebase غير محمّل، يعمل كل شيء **محلياً فقط** (`firebaseEnabled = false`) ولا تتزامن البيانات بين الأجهزة.

---

## 5) طبقة البيانات: `js/store.js`

### 5.1 مفاتيح التخزين `DB_KEYS`

| الثابت | الاسم الفعلي | محلي (localStorage) | Firebase | ملاحظة |
|---|---|---|---|---|
| `categories` | `ws_categories` | نعم | عقدة `ws_categories` | مصفوفة |
| `products` | `ws_products` | نعم | عقدة `ws_products` | مصفوفة، محلياً قد تكون **جزئية** (تُملأ حسب الحاجة) |
| `settings` | `ws_settings` | نعم | عقدة `ws_settings` | كائن |
| `ads` | `ws_ads` | نعم | عقدة `ws_ads` | مصفوفة |
| `orders` | `ws_orders` | نعم | عقدة `ws_orders` | تُضاف بـ `push` |
| `cart` | `ws_cart` | نعم | **لا** | السلة محلية لكل جهاز فقط |
| `session` | `ws_admin_session` | **sessionStorage** | لا | علامة "الأدمن مسجّل" |
| `seeded` | `ws_seeded_v1` | نعم | لا | علامة أن التهيئة الأولية تمت |

مفاتيح محلية مساعدة أخرى:
- `ws_data_version`: آخر `dataVersion` طُبّق على هذا المتصفح.
- `last_meta_pull_time`: وقت آخر سحب للأقسام/الإعدادات/الإعلانات (كاش 15 دقيقة).
- `ws_products_all_time`: وقت آخر تحميل لكل المنتجات.
- `ws_products_filter_<name>` و`ws_products_filter_<name>_time`: كاش قوائم الرئيسية (الأسماء المستخدمة: `featured`، `offers`، `new`).
- `ws_products_category_<catId>` و`..._time`: كاش منتجات قسم معين.

### 5.2 مراحل الإقلاع (تتم عند تحميل `store.js`)

1. **تهيئة Firebase** (دالة فورية): إذا وُجد `firebaseDatabaseURL` والمكتبة محمّلة، تُهيّأ `database`، وإذا اكتملت مفاتيح الـ Auth تُهيّأ `firebaseAuth`. تُضبط `firebaseEnabled`.
2. **`pullFromFirebase()`** (تُستدعى فوراً): تسحب **الأقسام والإعدادات والإعلانات فقط** (ليس المنتجات) وتكتبها محلياً، ثم تُطلق `store:synced`. لها كاش 15 دقيقة (`last_meta_pull_time`)، وفي فترة الكاش تُطلق `store:synced` مباشرة دون شبكة. إذا كانت `sessionStorage.ws_admin_session === "1"` تسحب الطلبات أيضاً.
3. **`seedIfNeeded()`**: تقارن `STORE_CONFIG.dataVersion` مع `ws_data_version` المحلي. عند الاختلاف **تمسح** الأقسام والمنتجات والإعلانات وكل كاشات المنتجات و`last_meta_pull_time`، وتزيل علامة `seeded`. ثم (عند غياب علامة `seeded`) **تُنشئ من جديد**: أقسام/منتجات فارغة، إعدادات من `STORE_CONFIG`، **سلة فارغة**، **طلبات محلية فارغة**، إعلانات فارغة.
   - **نتيجة عملية لرفع `dataVersion`:** يُفرَّغ كاش الزائر وسلته ويُعاد سحب البيانات من Firebase. استعمله بعد تعديل شامل للبيانات أو عند تغيير بنية المنتج.

### 5.3 واجهة `Store` (الدوال الفعلية)

**الأقسام:** `getCategories()`, `saveCategories(list)`, `addCategory(cat)`, `updateCategory(id, patch)`, `deleteCategory(id)`, `getCategoryName(id)`

**المنتجات (محلي):** `getProducts()`, `saveProducts(list)`, `getProduct(id)`, `addProduct(prod)` (يضيف في **بداية** القائمة مع قيم افتراضية)، `updateProduct(id, patch)`, `deleteProduct(id)`, `_cacheLoadedProducts(arr)` (دمج حسب `id` في القائمة المحلية)

**المنتجات (جلب عند الطلب، تُخزَّن محلياً وتُدمج):**
- `loadProductsPage(pageSize, cursorKey)` ← كل المنتجات مرقّمة (`orderByKey`)
- `loadProductsPageByCategory(categoryId, pageSize, cursorKey)` (موجودة لكن المتجر يستخدم `loadProductsByCategory` حالياً)
- `loadProductsPageByField(field, value, pageSize, cursorKey)` ← للفلاتر (`featured`, `isOffer`, `isNew`)
- `loadProductsByField(field, value, cacheName, forceRefresh)` ← مع كاش 15 دقيقة (تستخدمها `index.html`)
- `loadProductsByCategory(categoryId, forceRefresh)` ← كاش 15 دقيقة لكل قسم
- `loadProductById(id, forceRefresh)` ← **يعيد النسخة المحلية فوراً إن وُجدت** ولا يتحقق من Firebase إلا إذا لم تُوجد أو طُلب `forceRefresh`
- `loadAllProductsFromFirebase(forceRefresh)` ← يجلب **كل** المنتجات (كاش 15 دقيقة). يُستخدم في: البحث العام، البحث/الترتيب في المتجر، ولوحة الأدمن

**المخزون والتوفر:** `hasVariantMatrix(product)`, `getTotalStock(product)`, `getVariantStock(product, color, size)`, `isProductAvailable(product)`

**الإعدادات:** `getSettings()`, `saveSettings(patch)` (دمج ثم مزامنة)

**الإعلانات:** `getAds()`, `saveAds(list)`, `addAd(data)`, `deleteAd(id)` (لا يوجد تعديل، فقط إضافة/حذف)

**السلة (محلية + حدث `cart:updated`):** `getCart()`, `saveCart(cart)`, `addToCart(itemKey, qty, meta)`, `setQty(itemKey, qty)` (كمية ≤ 0 تحذف السطر), `removeFromCart(itemKey)`, `clearCart()`, `cartCount()`

**الطلبات:** `getOrders()`, `loadOrdersFromFirebase()` (بدون كاش للأدمن), `logOrder(order)`

**الدخول:** `login(email, password)` (يتحقق من `adminEmails` ويخرج الحساب إن لم يكن مسموحاً), `isLoggedIn()`, `logout()`

### 5.4 دوال مساعدة عامة (خارج `Store`)

`uid(prefix)` · `buildCartItemKey(productId, meta)` · `inventoryKey(color, size)` · `sumInventory(inventory)` · `firebaseValueToArray(value)` (يحوّل كائن/مصفوفة Firebase إلى مصفوفة نظيفة) · `syncNodeToFirebase` · `fetchNode`

---

## 6) نماذج البيانات (Schemas)

### 6.1 المنتج `Product`

```js
{
  id: "prd_xxxxx",          // يُولَّد بـ uid("prd")
  name: "",
  description: "",
  price: 0,                 // رقم (دينار)، السعر الوحيد؛ لا يوجد حقل سعر قديم/خصم
  categoryId: "cat_xxxxx",
  image: null,              // رابط الصورة الرئيسية (قد تكون رابط ImageKit)
  images: [],               // موجود في الافتراضي لكنه غير مستخدم في أي واجهة
  stock: 0,                 // مجموع المخزون؛ في حالة وجود ألوان/مقاسات يُحسب تلقائياً من inventory
  available: true,          // تفعيل/إخفاء يدوي ("مفعّل / معروض")
  featured: false,          // يظهر في "منتجات مميزة"
  isNew: false,             // يظهر في "وصل حديثاً"
  isOffer: false,           // يظهر في "عروض خاصة" (شارة "عرض🔥")

  // مصفوفة الخيارات الحالية (الوحيدة التي يكتبها الأدمن)
  colors: [ { name: "أسود", hex: "#000000", image: "https://..." | null } ],
  sizes:  [ "S", "M", "L" ],
  inventory: { "أسود||S": 3, "أسود||M": 0 },   // المفتاح من inventoryKey(color,size)

  // خيارات نصية قديمة (Legacy): لا تُنشأ من لوحة الأدمن حالياً لكن الواجهة تدعمها
  variants: [],             // مثال: ["صغير","كبير"]
  variantImages: {}         // { "صغير": "https://..." }
}
```

**صيغة مفتاح المخزون `inventoryKey`:** `"اللون||المقاس"`، وعند غياب أحدهما يُستبدل بـ `_`:
`"أسود||M"` · `"أسود||_"` (لون فقط) · `"_||M"` (مقاس فقط).

**حسابات التوفر:**
- `hasVariantMatrix` = للمنتج ألوان أو مقاسات.
- `getTotalStock` = مجموع `inventory` إن وُجدت مصفوفة، وإلا `stock`.
- `getVariantStock(p,c,s)` = `inventory[key]` إن وُجدت مصفوفة، وإلا `stock`.
- `isProductAvailable` = `available && getTotalStock > 0`.

### 6.2 القسم `Category`

```js
{ id: "cat_xxxxx", name: "", icon: "box", image: "https://..." | null, parentId: "" }
```
`parentId` فارغ = قسم رئيسي. مستوى واحد فقط من التفرع (القسم الفرعي لا يظهر في خيارات "يتبع لقسم").

### 6.3 الإعلان `Ad`

```js
{ id: "ad_xxxxx", image: "https://...", link: "" , order: 0 }
```
يُرسم في سلايدر الرئيسية مرتباً بـ `order` تصاعدياً. يوجد أيضاً دعم لمتغير عام اختياري `CODE_ADS` (مصفوفة بنفس الشكل) تُدمج مع إعلانات الأدمن؛ **غير معرَّف حالياً في أي ملف** (الكود يتحقق بـ `typeof`).

### 6.4 سطر السلة `CartLine`

```js
{ itemKey: "prd_1|c:أسود|s:M", productId: "prd_1", qty: 2, color: "أسود"|null, size: "M"|null, variant: null }
```
`itemKey` من `buildCartItemKey` (أجزاء: `productId` ثم `c:` لون ثم `s:` مقاس ثم `v:` خيار نصي). `productId` يُستخرج من `itemKey.split('|')[0]`. **إضافة بسيطة من بطاقة المنتج (`quickAddToCart`) تستعمل `productId` كـ `itemKey`** مباشرة (منتج بلا خيارات).

### 6.5 الإعدادات `Settings`

مفاتيح `ws_settings`:

```js
{ storeName, storeTagline, storeDescription,
  whatsapp,            // أرقام فقط بصيغة دولية (whatsappDigitsOnly)
  instagram, tiktok, phone, address, workingHours, deliveryInfo, currencySymbol }
```
**انتبه للاسم:** في `config.js` الحقل `whatsappNumber` أما في الإعدادات المخزّنة فهو `whatsapp`. التحويل يتم في `seedIfNeeded`.
**أولوية القيم:** `STORE_CONFIG` هي قيم **أولية فقط**. بعد أول تهيئة تصبح الإعدادات المخزّنة (وما في Firebase) هي المرجع، وتعديلات `config.js` اللاحقة لا تظهر إلا بعد رفع `dataVersion` (أو إذا كانت Firebase فارغة).

### 6.6 الطلب `Order`

```js
{
  id: "ord_xxxxx", date: "ISO string",
  type: "single" | "cart",
  customer: { gov, area, landmark /* "لا يوجد" إن تُرك فارغاً */, phone },
  items: [ { productId, name, qty, price, color|null, size|null, variant|null } ],
  total: 0            // غير شامل أجور التوصيل
}
```
يُنشأ في `whatsapp.js` ويُسجَّل عبر `Store.logOrder` قبل فتح واتساب. **السعر يُحفظ وقت الطلب** (لقطة)، ولا يتأثر بتعديل المنتج لاحقاً.

---

## 7) بنية Firebase Realtime Database

```
/ws_categories   [ {Category}, ... ]            مصفوفة (مفاتيح 0,1,2...)
/ws_products     [ {Product}, ... ]             مصفوفة (مفاتيح 0,1,2...)
/ws_settings     { Settings }
/ws_ads          [ {Ad}, ... ]
/ws_orders       { "-Nxyz...": {Order}, ... }   مفاتيح push تلقائية
```

**نقاط مهمة:**
1. الأقسام والمنتجات والإعلانات تُحفظ **كمصفوفة كاملة** في كل تعديل (`set` للعقدة). أي كتابة تستبدل العقدة بأكملها. لذلك الأدمن يجلب **كل المنتجات** عند فتح اللوحة (`loadAllProductsFromFirebase`) قبل أي تعديل، وإلا ستُكتب قائمة جزئية فوق الكاملة. **لا تستدعِ `saveProducts` بقائمة جزئية أبداً.**
2. صفحات المتجر تستعلم عن المنتجات بـ `orderByChild(...)`:
   `id` · `categoryId` · `featured` · `isOffer` · `isNew`
   وهذه تتطلب **فهارس `.indexOn`** في Rules على `ws_products` (وإلا تُجلب البيانات كاملة للفلترة عند العميل مع تحذير في الـ Console). الترقيم (`pagination`) يستخدم مفتاح الطفل (`child.key`) كمؤشر (`cursor`).
3. الطلبات تُضاف بـ `push` فلا تتعارض بين زبونين.

---

## 8) الأنظمة الأساسية (كيف تعمل)

### 8.1 الهيدر والفوتر والقائمة الجانبية — `app.js`
- **دوال التهريب المشتركة (أعلى `app.js`، متاحة لكل الصفحات وللأدمن):**
  - `escapeHtml(str)` ← لأي نص يدخل في `innerHTML` أو في قيمة سمة (`src`, `alt`, `title`, `data-*`, `href`). تعيد `""` لـ `null`/`undefined`.
  - `jsStr(value)` ← لتمرير قيمة نصية داخل `onclick="fn(...)"`: `'onclick="removeCartLine(' + jsStr(key) + ')"'`. (تعالج علامات الاقتباس في أسماء الألوان داخل `itemKey`.)
  - معرّفات المنتجات والأقسام في الروابط تُمرَّر بـ `encodeURIComponent`.
  - **أي كود جديد يعرض نصاً من البيانات (اسم، وصف، لون، مقاس، إعدادات) يجب أن يستخدمها.**
- كل صفحة فيها `<div id="site-header" data-active="KEY">` و`<div id="site-footer">`. المفتاح يحدد الرابط النشط: `home` · `products` · `categories` · `about` · `contact`.
- `renderHeader()` و`renderFooter()` تُستدعيان عند `DOMContentLoaded` وعند `store:synced` (إعادة رسم كاملة).
- الروابط من المصفوفة `NAV_LINKS`. أسماء الأقسام في الفوتر (أول 5 أقسام) والقائمة الجانبية (الأقسام الرئيسية مع تفرعاتها) تأتي من `Store.getCategories()`.
- القائمة الجانبية `#sidebarNav` و`#mainSidebarOverlay` تُنشآن ديناميكياً في `body` (`initSidebarDOM`) وتُفتح بـ `toggleSidebar()`. داخل `products.html` تستدعي روابط الأقسام `updateCategory(id)` بدل إعادة تحميل الصفحة.
- البحث العام (`initGlobalSearch`): يستدعي `Store.loadAllProductsFromFirebase()` ويفلتر بالاسم والوصف والخيارات النصية القديمة.
- `showToast(message)` (إشعار صغير)، `updateCartBadge()` (يستمع لحدث `cart:updated`).
- **الفوتر يحتوي على سطر حقوق ثابت داخل الكود** (اسم المبرمج، انستغرام، رقم هاتف). يجب تعديله أو إزالته لكل عميل (White-label).

### 8.2 المتجر — `products.js` (صفحة `products.html`)
- الحالة في `shopState`: `search`, `categoryId`, `sort`, `filterMode` (`featured`/`offer`/`new`/فارغ)، و`pagination`.
- الرابط يدعم: `products.html?cat=<id>` و`?filter=featured|offer|new` و`?q=<نص>`. تغيير القسم يحدّث الرابط بـ `history.pushState`.
- `SHOP_PAGE_SIZE = 20` (حجم الصفحة). التحميل اللانهائي بـ `IntersectionObserver` على `#shopLoadMoreSentinel`.
- مصدر البيانات حسب الحالة في `fetchNextShopBatch`:
  - **بحث أو ترتيب غير افتراضي** ← تحميل **كل** المنتجات ثم فلترة/ترتيب عند العميل.
  - **فلتر** (مميزة/عروض/جديد) ← `loadProductsPageByField`.
  - **الكل** ← `loadProductsPage`.
  - **قسم معين** ← `loadProductsByCategory` لكل من القسم وتفرعاته (بدون ترقيم، `done = true`).
- شريط الأقسام `#filterCategories` (أزرار: الكل، المميزة، العروض، وصل حديثاً، ثم الأقسام الرئيسية). عند اختيار قسم له تفرعات يُنشأ ديناميكياً `#subCategoryScroller` (رقائق `cat-sub-chip`).
- **الرسم الفوري:** إذا وُجدت منتجات محلية والحالة "الكل" بلا بحث، تُرسم أول 20 فوراً قبل أي جلب.

### 8.3 بطاقة المنتج
- `renderProductCard(product)` في `products.js`.
- **نسخة مكررة:** `renderHomeProductCard` و`productMediaHtml` و`quickAddToCart` و`renderGridInto` موجودة **أيضاً** داخل سكربت `index.html` (لأنه لا يحمّل `products.js`). **أي تعديل على شكل البطاقة يجب أن يُطبَّق في الملفين.**
- الشارات: `isOffer` ← "عرض🔥" (أولوية أولى)، وإلا `isNew` ← "جديد"، وإلا `featured` ← "مميز"؛ وشارة "غير متوفر" تُضاف إذا `!isProductAvailable`. تحت الاسم: نقاط ألوان (أول 4 + `+N`) أو شارة "N خيارات" للخيارات النصية القديمة.
- `quickAddToCart(productId)`: إن كان للمنتج خيارات (ألوان/مقاسات/variants) تحوّل لصفحة المنتج، وإلا يضيف بكمية 1.

### 8.4 صفحة المنتج — `initProductDetailPage()` في `products.js`
- المعرف من `?id=`. `Store.loadProductById(id)`.
- يبني الواجهة في `#productDetail`: صورة، حالة المخزون، السعر، وصف قابل للطي، أزرار الألوان/المقاسات (أو قائمة الخيارات النصية القديمة)، عداد الكمية، زرا "أضف للسلة" و"طلب عبر واتساب".
- `refreshAvailabilityUI()` تعيد حساب المخزون للتركيبة المختارة (لون/مقاس) وتُظهر/تخفي الأزرار.
- تغيير اللون يبدّل الصورة الرئيسية إذا للون صورة.
- أول لون/مقاس يكون محدداً افتراضياً.

### 8.5 السلة — `cart.js` (صفحة `cart.html`)
- `initCartPage()` تجلب فقط المنتجات الموجودة في السلة (`Store.loadProductById` لكل سطر) ثم `renderCartPage()`.
- `renderCartPage()` تحسب المجاميع وتعرض تحذيراً `#cartWarning` إذا منتج غير متاح/مخزون صفر.
- **المنتج غير المتاح** (معطّل أو مخزون اللون/المقاس صفر) يظهر في السلة برسالة حمراء "غير متوفر حاليًا — لن يُضاف إلى الطلب" ولا يدخل في عدد القطع ولا المجموع. وزر الطلب يُعطَّل إذا لم يبق أي سطر قابل للطلب.
- الكمية المعروضة تُقيَّد بمخزون التركيبة. والمنطق نفسه يُطبَّق عند بناء الطلب عبر `getOrderableCartLines` في `whatsapp.js` (القسم 8.6)، فيتطابق ما تراه في السلة مع ما يُرسَل ويُسجَّل.
- ملاحظة "السعر غير شامل أجور التوصيل" ثابتة في `#cartDeliveryNote` مع `deliveryInfo` من الإعدادات.
- زر الطلب `#checkoutBtn` يتحقق من `isWhatsAppConfigured()` ثم `orderCartViaWhatsApp()`.

### 8.6 دورة الطلب عبر واتساب — `whatsapp.js`
1. `isWhatsAppConfigured()` ← وجود رقم `settings.whatsapp`. إن لم يوجد: توست بالتنبيه.
2. `showDeliveryModal(onConfirm)` ← ينشئ `#deliveryModal` مرة واحدة ويعيد استخدامه: المحافظة، المنطقة، أقرب نقطة (اختياري)، رقم الهاتف. يُعيد تعيين الحقول ويستبدل النموذج بنسخة (`cloneNode`) لإزالة المستمعات القديمة.
3. **للسلة فقط:** `getOrderableCartLines(cart, products)` تعيد `{ lines, skipped }`: تقيّد كل كمية بمخزون اللون/المقاس وتستبعد المنتجات المحذوفة/المعطّلة/ذات المخزون الصفري (لا تعدّل السلة الأصلية). إن لم يبق شيء: توست ولا يُفتح شيء. إن استُبعد شيء: توست بعدد المستبعَد.
4. `onConfirm(info)` ← `Store.logOrder({...})` ثم `window.open(url, "_blank")`. (في السلة تُستخدم السطور المقيّدة `orderLines` للتسجيل والرسالة.)
5. السلة: بعد الفتح `Store.clearCart()` (تُفرَّغ السلة كلها بما فيها المستبعَد).
6. الرسالة تُبنى في `buildProductWhatsAppLink` (منتج واحد) و`buildCartWhatsAppLink` (سلة)، والرابط من `buildWhatsAppUrl`: `https://wa.me/{digits}?text={encodeURIComponent(msg)}`.
7. `formatPrice(n)` ← `n.toLocaleString("en-US") + " " + رمز العملة` (أرقام لاتينية).

### 8.7 الأقسام — `categories.html` (inline)
- `renderAllCategoriesPage()` ترسم بطاقة لكل قسم رئيسي مع عدد منتجاته (من **المنتجات المحلية المخزّنة** فقط، قد يكون العدد ناقصاً إن لم تُحمَّل كل المنتجات) ورقائق التفرعات. تعيد الرسم عند `store:synced`.
- تحوّل رابط الصورة بالتعبير النمطي `tr:...` إلى `tr:w-700,q-85` (يفترض رابط ImageKit).

### 8.8 الرئيسية — `index.html` (inline)
- `renderAllHomeContent()`: يعوّض أيقونات `__ICON_*__` في قسم المميزات، يرسم روابط التواصل والأقسام، يرسم المنتجات من الكاش المحلي فوراً، يرسم السلايدر، ثم يحدّث المنتجات بـ `Store.loadProductsByField(...)` (مميزة/عروض/جديد، حتى 8 لكل قسم).
- **`renderAdsSlider()`**: يدمج `Store.getAds()` مع `CODE_ADS` (إن وُجد)، يبني `.promo-slider-container`، تبديل تلقائي كل 3500ms، نقاط تحكم، وسحب باللمس. بلا إعلانات يعرض الشعار الافتراضي `.hero-art-default`.
- أقسام الصفحة (بالترتيب): hero، الأقسام `#categories`، مميزة `#featuredGrid`، عروض `#offerGrid`، جديد `#newGrid`، "لماذا نحن" `.feature-grid`، شريط التواصل `#contactBand`.
- أزرار "عرض الكل" تشير إلى `products.html?filter=featured|offer|new`.

### 8.9 التواصل والمن نحن
- `contact.html`: ترسم بطاقات (واتساب، انستغرام، تيك توك [أيقونة SVG خاصة داخل الملف]، هاتف، عنوان) فقط للحقول غير الفارغة، ثم ساعات العمل ومعلومات التوصيل.
- `about.html`: الاسم والوصف من الإعدادات، و**نصوص القيم والفقرة التعريفية ثابتة في HTML**.

### 8.10 الدخول والحماية — `auth.js`, `login.html`
- `initAdminAuthGuard()` تراقب `firebase.auth().onAuthStateChanged`: إن كان المستخدم بريده ضمن `STORE_CONFIG.adminEmails` تضبط `sessionStorage.ws_admin_session = "1"` وتُطلق `admin:auth-ready`؛ وإلا تمسح العلامة وتحوّل لـ `login.html` إن كانت الصفحة الحالية لوحة الأدمن (`#adminApp`).
- `requireAdminAuth()` (تنتظر وعد `adminAuthReady`) تستخدمها `initAdminPage`.
- `initLoginPage()` تنفّذ `Store.login` ثم تحوّل لـ `admin.html`.
- **قائمة البريد المسموح بها (`adminEmails`) تُفحص في المتصفح فقط.** لا تعتبرها حماية حقيقية للبيانات؛ الحماية الفعلية في Firebase Rules.

---

## 9) لوحة التحكم — `admin.html` + `admin.js`

**التبويبات** (`.admin-nav button[data-panel]` ↔ `section#panel-<name>`): `dashboard` · `products` · `categories` · `ads` · `orders` · `settings`.

| التبويب | عناصر HTML المهمة | دوال JS الرئيسية |
|---|---|---|
| الإحصائيات | `#statsRow`, `#ordersTableBody` | `renderStats()` (منتجات، أقسام، غير متوفرة، عدد الطلبات) |
| المنتجات | `#productsTableBody`, `#adminProductSearch`, `#productModal`, `#productForm` | `renderProductsTable`, `openProductModal`, `saveProductForm`, `deleteProductConfirm` |
| الألوان/المقاسات/المخزون | `#colorsList`, `#productSizes`, `#inventoryWrap`, `#inventoryGrid`, `#productStock` | `renderColorsList`, `renderInventoryGrid`, `updateComputedStock`, `currentSizesFromInput` |
| الأقسام | `#categoriesTableBody`, `#categoryModal`, `#categoryForm` | `renderCategoriesTable`, `openCategoryModal`, `saveCategoryForm`, `deleteCategoryConfirm` |
| الإعلانات | `#adsTableBody`, `#adModal`, `#adForm` | `renderAdsTable`, `openAdModal`, `saveAdForm`, `deleteAdConfirm` |
| الطلبات | `#ordersTableBody2` | `renderOrdersTable` |
| الإعدادات | `#settingsForm` (حقول بأسماء `name=`) | `fillSettingsForm`, `wireSettingsForm` |

**التسلسل عند الفتح (`initAdminPage`):** `requireAdminAuth` ← `Store.loadAllProductsFromFirebase()` ← `Store.loadOrdersFromFirebase()` ← ربط القوائم ← رسم كل الجداول ← ربط النوافذ والإعدادات.

**تفاصيل مهمة:**
- **حقول المنتج (معرّفات عناصر النموذج):** `productName`, `productDescription`, `productPrice`, `productCategorySelect`, `productStock`, `productAvailable`, `productFeatured`, `productNew`, `productOffer`, `productSizes`, `productImageInput`.
- عند وجود ألوان أو مقاسات: يظهر جدول المخزون ويصبح `productStock` للقراءة فقط ويُحسب تلقائياً.
- `saveProductForm` تستدعي `Store.updateProduct(id, data)` (دمج `patch` فيبقى أي حقل غير موجود في `data` مثل `variants` القديمة) أو `Store.addProduct(data)`.
- `uploadToImgBB(file, isBanner)`: رفع إلى ImgBB ثم، إن كان `imageKitEndpoint` معرّفاً ورابط ImgBB من `i.ibb.co`، يحوّل الرابط إلى ImageKit بتحويل `tr:w-800,q-85,f-auto` (عادي) أو `tr:w-1200,q-90,f-auto` (إعلانات). وإلا يُعاد رابط ImgBB كما هو.
- `escapeHtml()` معرّفة في `app.js` (تُستخدم في جداول الأدمن وفي واجهة المتجر العامة). لا تُعرَّف نسخة ثانية في `admin.js`.
- **القسم (نافذة الأقسام):** حقل "يتبع لقسم" يُنشأ بالـ JS وقت الفتح (`#categoryParentContainer`) بعد `#categoryName`.
- **منتقي أيقونة القسم:** الحاوية `#categoryIconPicker` موجودة في نافذة الأقسام؛ `populateIconPicker()` تبني أزرار راديو من `CATEGORY_ICON_KEYS` و`syncIconPickerActive()` تميّز المختار (الراديو مخفي بالـ CSS `.icon-choice input`). `saveCategoryForm` تحفظ الأيقونة المختارة، وإن لم تُحدَّد أيقونة عند التعديل (أيقونة قديمة خارج القائمة) تحتفظ بالأيقونة الحالية بدل `box`. الأيقونة تظهر في الواجهة عند عدم وجود صورة للقسم.
- **حذف قسم** لا يحذف المنتجات ولا يعيد ربط الأقسام الفرعية.
- الطلبات تظهر في لوحتين (`#ordersTableBody` في الإحصائيات و`#ordersTableBody2`): سكربت inline في `admin.html` يلتف حول `renderOrdersTable` وينسخ المحتوى للجدول الثاني.

---

## 10) قواعد Firebase (ملف `firebase-rules.json`)

الفكرة: الزوار **يقرؤون** الكتالوج و**ينشئون** طلبات فقط؛ الأدمن فقط يكتب الكتالوج ويقرأ الطلبات. تُلصق من Firebase Console ← Realtime Database ← Rules بعد استبدال `ADMIN_EMAIL` ببريد الأدمن (نفس البريد في `STORE_CONFIG.adminEmails`). لأكثر من أدمن كرّر الشرط بـ `||`.

- `ws_categories`, `ws_settings`, `ws_ads`: قراءة عامة، كتابة للأدمن.
- `ws_products`: قراءة عامة، كتابة للأدمن، و`.indexOn`: `id, categoryId, featured, isOffer, isNew` (مطلوبة لاستعلامات `orderByChild`).
- `ws_orders`: **قراءة للأدمن فقط**. `$orderId`: يُسمح بالإنشاء لأي زائر (`!data.exists()`) مع تحقق من وجود الحقول `date, type, items, total, customer` وأن `total` رقم؛ والتعديل/الحذف للأدمن فقط.
- عند إضافة حقل جديد للمنتج تُفلتر به في الاستعلامات أضفه إلى `.indexOn`.
- **تحذير:** القواعد لم تُجرَّب على مشروعك؛ استعمل Rules Playground في Firebase للتأكد قبل الاعتماد عليها، ثم اختبر من الموقع: تصفح الزائر، إنشاء طلب، دخول الأدمن وتعديل منتج وقراءة الطلبات.

---

## 11) «أين أعدّل؟» — دليل التعديلات الشائعة

| المطلوب | الملفات | التفاصيل |
|---|---|---|
| ألوان الموقع وخطوطه ومسافاته | `css/style.css` | متغيرات `:root` أعلى الملف. **أسماء المتغيرات `--olive-*` تاريخية، والقيم الحالية أخضر مزرق داكن.** غيّر القيم لا الأسماء. الخط `Tajawal` (لا يوجد في الملفات تحميل للخط نفسه؛ تأكد من تحميله في الاستضافة إن لزم). |
| اسم المتجر/الوصف/التواصل الافتراضي | `js/config.js` ثم لوحة الأدمن | `config.js` قيم أولية فقط. للتطبيق على أجهزة زارت الموقع سابقاً ارفع `dataVersion` أو عدّل من لوحة الأدمن. |
| الشعار | `assets/logo/logo.png` | نفس الاسم والمسار مستخدم في كل الصفحات. |
| شكل بطاقة المنتج | `js/products.js` (`renderProductCard`) **و** `index.html` (`renderHomeProductCard`) | زامن الاثنين. التنسيق في `style.css` (قسم Product cards). |
| رسالة الواتساب | `js/whatsapp.js` | `buildProductWhatsAppLink`, `buildCartWhatsAppLink`. |
| حقول نافذة التوصيل | `js/whatsapp.js` | `showDeliveryModal` + حقول `customer` في `orderSingleProductViaWhatsApp` و`orderCartViaWhatsApp` + عرض العميل في `admin.js` (`renderOrdersTable`) + الرسالة في دالتي البناء. |
| رمز العملة/تنسيق السعر | `formatPrice` في `whatsapp.js` + حقل `currencySymbol` | |
| عدد المنتجات في الصفحة | `js/products.js` | `SHOP_PAGE_SIZE` |
| قواعد أمان Firebase | `firebase-rules.json` | تُطبَّق يدوياً في لوحة Firebase، لا تُحمَّل مع الموقع. |
| مدة الكاش | `js/store.js` | `cooldownMs` (15 دقيقة) مكرر في عدة دوال: `pullFromFirebase`, `loadProductsByField`, `loadProductsByCategory`, `loadAllProductsFromFirebase`. |
| روابط التنقل | `js/app.js` | `NAV_LINKS` (وفي الفوتر قائمة "روابط سريعة" ثابتة داخل `renderFooter`). |
| حقوق المبرمج في الفوتر | `js/app.js` | داخل `renderFooter` (`footer-bottom`). |
| نصوص ثابتة (hero، المميزات، القيم...) | `index.html`, `about.html`, `contact.html` | هي نصوص HTML مباشرة وليست من الإعدادات. |
| عنوان الصفحات (`<title>`) | كل ملف HTML | "متجرك الإلكتروني" مكتوب يدوياً (صفحة المنتج تغيّره ديناميكياً). وكذلك اسم العلامة في `admin.html`/`login.html`. |
| أيقونة جديدة | `js/icons.js` | أضف مفتاحاً في `ICONS`؛ وللأقسام أضفه إلى `CATEGORY_ICON_KEYS`. |
| سلايدر الإعلانات | `index.html` (`renderAdsSlider`) + CSS `.promo-*` | |
| وسيلة تواصل جديدة (سناب/يوتيوب) | `admin.html` (حقل في `#settingsForm`), `admin.js` (`fillSettingsForm` + `wireSettingsForm`), `store.js` (`seedIfNeeded` للقيمة الأولية), `app.js` (`renderFooter`), `contact.html`, `index.html` (`homeSocialRow`), `about.html` | |
| إضافة صفحة جديدة | ملف HTML جديد بنفس قالب الصفحات + `NAV_LINKS` | استخدم `site-header` و`site-footer` وترتيب السكربتات في القسم 3. |

### وصفة: إضافة حقل جديد للمنتج (مثل: الماركة، الضمان، المواصفات)
1. **`admin.html`**: أضف الحقل داخل `#productForm` بمعرّف واضح.
2. **`admin.js`**: في `openProductModal` املأ القيمة عند التعديل (وصفّرها عند الإضافة)، وفي `saveProductForm` أضفها إلى كائن `data`.
3. **`store.js`**: (اختياري) أضف قيمة افتراضية في `addProduct`.
4. **العرض**: `products.js` داخل `initProductDetailPage` (و`renderProductCard` إن ظهر على البطاقة، و**نسخته في `index.html`**).
5. **البحث**: إن أردت البحث به عدّل فلتر `renderShopResults` في `products.js` و`initGlobalSearch` في `app.js`.
6. **الواتساب**: إن أردت ظهوره في الرسالة عدّل `whatsapp.js` (ومرّره في `items` عند التسجيل).
7. **الفلترة عبر Firebase**: إن كان سيُستعلم عنه بـ `orderByChild` أضفه إلى `.indexOn`.
8. ارفع `dataVersion` إن غيّرت بنية البيانات بحيث لا تصلح النسخ المخزّنة عند الزوار.

### أفكار تخصيص حسب نوع المتجر (ليست مطبّقة)
- **ملابس:** الألوان والمقاسات والمخزون لكل تركيبة **مطبّقة بالفعل**.
- **مواد كهربائية:** غالباً تحتاج حقولاً مثل الماركة/الضمان/المواصفات (استخدم الوصفة أعلاه)، وقد لا تحتاج الألوان/المقاسات (تُترك فارغة فتصبح كمية بسيطة).
- **كوزمتك:** قد تحتاج تجميع الخيارات كأحجام (مل) عبر `sizes`، أو حقلاً للمكونات/نوع البشرة.

---

## 12) مزالق وتنبيهات للذكاء الاصطناعي (اقرأها قبل أي تعديل)

1. **لا ES Modules.** لا تكتب `import`/`export`. كل شيء عالمي. ترتيب السكربتات حرج (القسم 3).
2. **ازدواج الكود:** بطاقة المنتج و`quickAddToCart` و`renderGridInto` و`productMediaHtml` مكررة بين `products.js` و`index.html`. عدّل الاثنين.
3. **الواجهات مبنية بدمج نصوص HTML** داخل JS. انتبه لإغلاق الوسوم ولعلامات الاقتباس (`'` و`"`) وللهروب داخل `onclick="...('id')"`.
4. **`itemKey` ليس `productId`** للمنتجات ذات الخيارات (القسم 6.4).
5. **لا تكتب `Store.saveProducts` بقائمة جزئية** (تستبدل عقدة Firebase كاملة). للتعديل استخدم `updateProduct/addProduct/deleteProduct` بعد تحميل القائمة الكاملة.
6. **نسخة المنتج المحلية قد تكون قديمة:** `loadProductById` يفضّل المحلي. بعد تعديل الأدمن قد يرى الزوار بيانات قديمة حتى ينتهي الكاش أو يتغير `dataVersion`. صفحات القوائم تتحدث كل 15 دقيقة.
7. **رفع `dataVersion` يمسح سلة الزائر** وطلباته المحلية وكاش المنتجات والأقسام (القسم 5.2).
8. **`initHomeCollections()` في `products.js`** تنتهي مبكراً إذا لم توجد عناصر الصفحة الرئيسية (`#featuredGrid`, `#offerGrid`, `#newGrid`, `#homeCategories`)، فلا تُطلق استعلامات Firebase في `products.html` و`product.html`. (الرئيسية نفسها لا تحمّل `products.js` وتستعمل `renderAllHomeContent` في `index.html`.)
9. **أيقونات ناقصة:** `iconSvg("info")` (في `NAV_LINKS`) و`iconSvg("image")` (لوحة الأدمن وجدول الإعلانات) غير معرّفتين في `ICONS`، فتظهر أيقونة `box` البديلة. أضفهما إن أردت أيقونات صحيحة.
10. **`CODE_ADS`** مُشار إليه في `index.html` لكنه غير معرَّف (آمن بسبب `typeof`).
11. **أرقام الإصدار `?v=`:** موحّدة حالياً على `41` في كل صفحات HTML وكل ملفات `js/` و`css/` (بما فيها `config.js`). **عند تعديل أي ملف مشترك ارفع الرقم في كل الصفحات معاً** (بحث واستبدال)، وإلا تبقى صفحات بنسخ مخزّنة قديمة.
12. **وسم `viewport` مكرر** في بعض الصفحات (`about`, `contact`, `login`, `index`, `admin`). غير ضار، ويمكن حذف المكرر.
13. **تهريب النصوص:** كل بيانات المنتجات والأقسام والإعدادات تمر عبر `escapeHtml`/`jsStr` (القسم 8.1) في الواجهة العامة واللوحة. أي تطوير جديد يعرض نصاً من البيانات يجب أن يستخدمهما، وإلا تُفتح ثغرة XSS. ما يزال غير مهرَّب عمداً: روابط `href` الثابتة، وقيم `wa.me` (أرقام فقط عبر `whatsappDigitsOnly`).
14. **`window.open` للواتساب** يتم داخل معالج إرسال النموذج (تفاعل مباشر من المستخدم) لتفادي حظر النوافذ المنبثقة؛ لا تنقله لمؤقت أو استدعاء غير مباشر.
15. **نمط التواصل مع المستخدم:** الواجهة عربية RTL؛ النصوص الجديدة بالعربية وتتبع الأنماط الموجودة (`btn`, `btn-primary`, `btn-outline`, `btn-whatsapp`, `btn-sm`, `btn-block`, `field`, `pill`, `badge`, `modal-overlay`/`modal-card`, `empty-state`, `toast`).

---

## 13) خريطة `style.css` (حسب أقسام التعليقات)

متغيرات `:root` (ألوان، أنصاف أقطار، ظلال، خط، عرض الحاوية) ← الأزرار ← الهيدر ← نافذة البحث ← Hero ← الأقسام (`.section`, `.cat-chip`) ← بطاقات المنتجات والشارات ← نقاط الألوان ← الميزات/التواصل ← الفوتر ← رؤوس الصفحات ← المتجر ← تفاصيل المنتج ← السلة ← النماذج ← من نحن/تواصل ← الإدارة (Admin) ← التوست ← سلايدر الإعلانات ← القائمة الجانبية للمتجر والخلفية المعتمة ← التمرير الأفقي للأقسام ← حجم الصور في النوافذ ← تبويبات الأقسام والتفرعات ← تأثير تحديث الشبكة ← إصلاحات أيقونات الجداول.
بعض التنسيقات الخاصة بصفحات معينة موجودة داخل وسم `<style>` في الصفحة نفسها (`index.html`, `products.html`, `categories.html`, `admin.html`) وتُقدَّم على الملف العام أحياناً بـ `!important`.

---

## 14) قائمة تجهيز قالب لعميل جديد (Checklist)

1. نسخ المشروع، وتجهيز مشروع Firebase جديد (Realtime Database + Authentication بريد/كلمة مرور) وإنشاء حساب الأدمن.
2. تعبئة `js/config.js`: الاسم، الوصف، العملة، مفاتيح Firebase وعنوان قاعدة البيانات، `adminEmails`، مفتاح ImgBB، ونقطة ImageKit (اختياري)، وتصفير بيانات الاتصال أو تعبئتها.
3. تطبيق Firebase Rules وفهارس `.indexOn` (القسم 10).
4. استبدال `assets/logo/logo.png`.
5. تعديل الألوان في `:root`.
6. مراجعة النصوص الثابتة (`index.html`, `about.html`, عناوين `<title>`, اسم العلامة في `admin.html`/`login.html`).
7. تعديل/إزالة سطر حقوق المبرمج في `renderFooter`.
8. تعديل الحقول حسب نوع المتجر (القسم 11).
9. رفع رقم `?v=` في كل الصفحات إذا عُدّلت ملفات مشتركة (الرقم الحالي 41).
10. إدخال الأقسام والمنتجات من لوحة الأدمن، وضبط رقم واتساب ومعلومات التوصيل من الإعدادات، ثم تجربة طلب كامل من الهاتف.

---

**نهاية المرجع.** عند تغيير بنية المشروع (ملف جديد، حقل بيانات جديد، تغيير في ترتيب السكربتات أو في عقد Firebase) يجب تحديث هذا الملف.
