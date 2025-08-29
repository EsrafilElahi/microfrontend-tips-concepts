1. روش اول: Eager (عیبی‌گیری) - "همه رو باهم بیار!"
   📖 توضیح:

مثل این میمونه که همه دوستات رو همزمان به مهمونی دعوت کنی. باید از اول بدونی همه کیان و کجان.
🎯 مثال روزمره:

فرض کن میخوای یه مهمونی بزنی. از قبل میدونی دقیقاً چه کسانی میان (علی، مریم، رضا). به همه زنگ میزنی و میگی "همه باهم ساعت 8 بیاین خونه من"

```js
// webpack.config.js - از اول همه رو معرفی می‌کنیم
remotes: {
search: 'search@http://localhost:3001/remoteEntry.js',
booking: 'booking@http://localhost:3002/remoteEntry.js',
profile: 'profile@http://localhost:3003/remoteEntry.js'
}

// در برنامه
import SearchApp from 'search/SearchApp'; // از اول لود شده
import BookingApp from 'booking/BookingApp'; // از اول لود شده
```

✅ خوبی‌ها:

    سرعت بالا (همه از اول آماده‌اند)

    هیچ تأخیری نداره

❌ بدی‌ها:

    اگر کاربر فقط یه بخش رو بخواد، بقیه رو بی‌دلیل لود کرده

    حجم اولیه برنامه زیاد میشه

========================================

2. روش دوم: Dynamic (پویا) - "هرکی رو خواستی صدا بزن!"
   📖 توضیح:

مثل این میمونه که فقط وقتی به کسی نیاز داری بهش زنگ بزنی. لازم نیست از اول همه رو بشناسی.
🎯 مثال روزمره:

داری یه پروژه گروهی انجام میدی. فقط وقتی به کمک یه شخص خاص نیاز داری، بهش زنگ میزنی و میگی "الان بهت نیاز دارم بیا"

```js
// فقط وقتی لازمش داری صدا میزنی
const SearchApp = React.lazy(() => import("search/SearchApp"));
const BookingApp = React.lazy(() => import("booking/BookingApp"));

// فقط وقتی کاربر روی دکمه کلیک کرد
function showSearch() {
  return (
    <React.Suspense fallback={<div>در حال بارگذاری...</div>}>
      <SearchApp />
    </React.Suspense>
  );
}
```

✅ خوبی‌ها:

    حجم اولیه برنامه کم میشه

    فقط چیزایی که نیازه لود میشن

❌ بدی‌ها:

    وقتی میخوای استفاده کنی یه تأخیر کوچیک داره

    باید منتظر بمونی تا لود بشه

==========================================

3. روش سوم: Delegated (واگذار شده) - "از قبل به همه زنگ بزن ولی نگو بیان!"
   📖 توضیح:

مثل این میمونه که از قبل به همه میگی آماده باشن ولی فقط وقتی لازم شد بیان.
🎯 مثال روزمره:

به همه دوستات میگی "امروز خونه مونم، اگه نیاز شدم بهتون زنگ میزنم". همه آماده‌اند ولی فقط وقتی صدا زدی میان.

```js
// webpack.config.js - از اول می‌دونیم همه کیان
remotes: {
  search: 'search@http://localhost:3001/remoteEntry.js',
  booking: 'booking@http://localhost:3002/remoteEntry.js'
}

// اما فقط وقتی لازم شد لود می‌شوند
const loadSearch = () => {
  // وب‌پک از قبل می‌دونه search کجاست
  // و سریع‌تر لودش می‌کنه
  return import('search/SearchApp');
}
```

    هم حجم اولیه کمه

    هم وقتی نیاز شد سریع لود میشه

    بهترین هر دو دنیا!

❌ بدی‌ها:

    پیچیدگی بیشتری داره

    باید از اول بدونی همه سرویس‌ها کجان

=========================================

```js
// برای پروژه هتل‌داری شما

// 1. ابتدا همه رو معرفی می‌کنیم (Delegated)
// webpack.config.js
remotes: {
  search: 'search@http://localhost:3001/remoteEntry.js',
  booking: 'booking@http://localhost:3002/remoteEntry.js',
  profile: 'profile@http://localhost:3003/remoteEntry.js'
}

// 2. وقتی کاربر رفت به صفحه جستجو
const SearchPage = () => {
  const SearchApp = React.lazy(() => import('search/SearchApp'));

  return (
    <React.Suspense fallback={<div>در حال بارگذاری جستجو...</div>}>
      <SearchApp />
    </React.Suspense>
  );
}

// 3. وقتی کاربر رفت به صفحه رزرو
const BookingPage = () => {
  const BookingApp = React.lazy(() => import('booking/BookingApp'));

  return (
    <React.Suspense fallback={<div>در حال بارگذاری رزرو...</div>}>
      <BookingApp />
    </React.Suspense>
  );
}
```
