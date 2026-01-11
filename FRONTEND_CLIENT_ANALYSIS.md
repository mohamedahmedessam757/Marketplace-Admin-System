# 🔍 تحليل واجهة العميل (Frontend Client Analysis)

## 📋 معلومات المشروع

| البند | القيمة |
|-------|--------|
| **اسم المشروع** | e-tashleh.net (التشاليح) |
| **النوع** | منصة لبيع وشراء قطع غيار السيارات المستعملة |
| **التقنية** | Vue.js 3 + TypeScript |
| **الواجهة** | Tailwind CSS + PrimeVue |
| **الحالة** | Pinia |
| **الدفع** | Stripe |

---

## 🛠️ Technology Stack

```
┌─────────────────────────────────────────────────┐
│  Framework:    Vue.js 3 + Vite                  │
│  Language:     TypeScript                        │
│  UI Library:   PrimeVue 4 + Tailwind CSS 4      │
│  State:        Pinia                             │
│  Router:       Vue Router 4                      │
│  HTTP:         Axios                             │
│  Payments:     Stripe.js                         │
│  Icons:        Lucide Vue + PrimeIcons          │
└─────────────────────────────────────────────────┘
```

---

## 👥 أنواع المستخدمين (Account Types)

من ملف `src/types/index.ts`:

```typescript
account_type: 'user' | 'vendor' | 'customer'
```

| النوع | الوصف | Dashboard |
|-------|-------|-----------|
| **user** | مدير/أدمن | /admin/* (صفحة واحدة فقط) |
| **vendor** | تاجر/متجر | /vendor/dashboard |
| **customer** | عميل | /customer/* (13 صفحة) |

---

## 📄 الصفحات المتوفرة

### 1️⃣ صفحات عامة (Public)
| الصفحة | المسار | الوصف |
|--------|--------|-------|
| Welcome | `/` | الصفحة الترحيبية |
| Home | `/home` | الرئيسية |
| Login | `/login` | تسجيل دخول العميل |
| Admin Login | `/admin/login` | تسجيل دخول الأدمن |
| Privacy | `/privacy-policy` | سياسة الخصوصية |
| Terms | `/terms-of-service` | الشروط والأحكام |
| Return Policy | `/return-policy` | سياسة الإرجاع |
| About Us | `/about-us` | من نحن |
| How We Work | `/how-we-work` | كيف نعمل |
| Contact | `/contact-us` | تواصل معنا |
| Wholesale | `/wholesale` | طلبات الجملة |
| 404 | `/*` | صفحة غير موجودة |

### 2️⃣ صفحات العميل (Customer)
| الصفحة | المسار | الوصف |
|--------|--------|-------|
| Dashboard | `/customer/dashboard` | لوحة التحكم |
| New Order | `/customer/new-order` | طلب جديد |
| Orders | `/customer/orders` | طلباتي |
| Order Details | `/customer/orders/:id` | تفاصيل الطلب |
| Part Offers | `/customer/orders/:orderId/part/:partName` | عروض القطعة |
| Offer Chat | `/customer/orders/:orderId/offers/:offerId/chat` | محادثة العرض |
| Shipping | `/customer/orders/:orderId/offers/:offerId/shipping` | تأكيد الشحن |
| Confirmation | `/customer/orders/:orderId/offers/:offerId/confirm` | تأكيد الطلب |
| Payment | `/customer/orders/:orderId/offers/:offerId/payment` | الدفع |
| Invoice | `/customer/orders/:orderId/invoice` | الفاتورة |
| Final Report | `/customer/orders/:orderId/final-report` | التقرير النهائي |
| Profile | `/customer/profile` | الملف الشخصي |

### 3️⃣ صفحات التاجر (Vendor)
| الصفحة | المسار | الوصف |
|--------|--------|-------|
| Dashboard | `/vendor/dashboard` | لوحة تحكم التاجر |

### 4️⃣ صفحات الأدمن (Admin)
| الصفحة | المسار | الوصف |
|--------|--------|-------|
| Content Editor | `/admin/*` | محرر المحتوى (فقط) |

---

## 🔄 Flow الطلب (Order Flow)

```
1. العميل ينشئ طلب جديد (NewOrder.vue)
   - بيانات السيارة (manufacturer, carType, year)
   - القطع المطلوبة (parts[])
   - نوع الشحن (separate/combined)
   ↓
2. الطلب ينتظر العروض (status: pending)
   - مهلة 24 ساعة
   ↓
3. المتاجر ترسل عروض (Offers)
   - سعر، وصف، ضمان، وزن
   ↓
4. العميل يرى العروض (PartOffers.vue)
   ↓
5. العميل يفتح محادثة مع التاجر (OfferChat.vue)
   ↓
6. العميل يقبل العرض ويؤكد الشحن (OrderShipping.vue)
   ↓
7. العميل يؤكد الطلب النهائي (FinalOrderConfirmation.vue)
   ↓
8. العميل يدفع عبر Stripe (OrderPayment.vue)
   ↓
9. يتم إنشاء الفاتورة (OrderInvoice.vue)
   ↓
10. التقرير النهائي (OrderFinalReport.vue)
```

---

## 📊 حالات الطلب (Order Status)

```typescript
status: 'pending' | 'active' | 'expired' | 'completed' | 'cancelled'
```

| الحالة | الوصف |
|--------|-------|
| pending | الطلب جديد ينتظر العروض |
| active | يوجد عروض على الطلب |
| expired | انتهت مهلة الـ 24 ساعة |
| completed | الطلب مكتمل |
| cancelled | الطلب ملغي |

---

## 📊 حالات العرض (Offer Status)

```typescript
status: 'pending' | 'accepted' | 'rejected'
```

---

## 💳 حالات الدفع (Payment Status)

```typescript
status: 'paid' | 'unpaid'
method: 'stripe' | 'cash'
```

---

## 🔑 نظام المصادقة (Authentication)

من ملف `src/stores/auth.ts`:

```typescript
// الحفظ في localStorage
- auth_token: JWT Token
- auth_user: JSON User Object

// أنواع الحسابات
- user (admin)
- vendor (تاجر)
- customer (عميل)
```

**التوجيه حسب النوع:**
```typescript
switch (account_type) {
  case 'user': return '/customer/dashboard'
  case 'vendor': return '/vendor/dashboard'
  case 'customer': return '/customer/dashboard'
}
```

> ⚠️ **ملاحظة**: `user` يتم توجيهه لـ customer dashboard - هذا يحتاج مراجعة

---

## 📦 Data Types الأساسية

### User
```typescript
interface User {
  id: number
  name: string
  account_type: 'user' | 'vendor' | 'customer'
  phone: string
  email: string
  city?: string
  roles?: string[]      // للأدمن
  permissions?: string[] // للأدمن
}
```

### Order
```typescript
interface Order {
  id: number
  order_id: string
  user_id: number
  manufacturer: string  // الشركة المصنعة
  carType: string       // نوع السيارة
  year: number          // سنة الصنع
  parts: Part[]         // القطع المطلوبة
  shippingType: 'separate' | 'combined'
  status: 'pending' | 'active' | 'expired' | 'completed' | 'cancelled'
  acceptedOffers?: Record<string, number>
  payments?: Record<string, PartPaymentStatus>
}
```

### Offer
```typescript
interface Offer {
  id: number
  order_id: number
  user_id: number
  price: number
  final_price: number      // مع العمولة
  commission_rate: number  // نسبة العمولة (20%)
  has_warranty: boolean
  part_name: string
  part_weight: number
  status: 'pending' | 'accepted' | 'rejected'
  is_chat_enabled: boolean
  expires_at?: string      // 24 ساعة
}
```

---

## ⚠️ نقاط مهمة للتطوير الجديد

### 1. لوحة الأدمن غير مكتملة
- يوجد فقط `ContentEditor.vue`
- لا يوجد: Orders, Stores, Customers, Disputes, Billing, Settings, Audit Logs

### 2. لوحة التاجر غير مكتملة
- يوجد فقط `VendorDashboard.vue`
- لا يوجد: Orders Management, Offers Response, Shipping, Earnings, Profile Settings

### 3. Backend غير موجود
- الـ stores تعمل بـ localStorage فقط (Mock Data)
- لا يوجد API حقيقي

### 4. حالات الطلب بسيطة
- الحالات الموجودة: pending, active, expired, completed, cancelled
- المطلوب (حسب العقد): 9 حالات FSM كاملة

### 5. نظام المحادثات
- يوجد `OfferChat.vue` - يجب التأكد من التكامل مع Backend

---

## 📋 الإجابات المستخلصة من الواجهة

| السؤال | الإجابة من الكود |
|--------|-----------------|
| هل يوجد Chat؟ | ✅ نعم (`OfferChat.vue`) |
| طريقة الدفع؟ | Stripe + Cash |
| نسبة العمولة؟ | 20% (`commission_rate: 0.2`) |
| مهلة العروض؟ | 24 ساعة (`expires_at`) |
| أنواع الشحن؟ | منفصل/مجمع (`separate/combined`) |
| Vendor Dashboard؟ | ✅ موجود (صفحة واحدة فقط) |
| Admin Dashboard؟ | ⚠️ ناقص جداً |

---

## 🎯 التوصيات

1. **Admin Dashboard**: يجب بناءه من الصفر
2. **Vendor Dashboard**: يحتاج توسيع كبير
3. **FSM**: يجب تطبيق الـ 9 حالات المطلوبة
4. **Backend**: يجب بناءه (NestJS حسب الخطة)
5. **Authentication**: يحتاج إعادة تنظيم الـ Roles
