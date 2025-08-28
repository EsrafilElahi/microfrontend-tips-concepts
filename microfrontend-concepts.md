# All Concepts :

## remote :

1. تعریف Remotes (چیزهایی که import می‌کند)

```js
const CHECKOUT_APP_URL =
  process.env.NEXT_PUBLIC_CHECKOUT_APP_URL || "http://localhost:3002";

const remotes = (isServer) => {
  const location = isServer ? "ssr" : "chunks";
  return {
    checkout: `checkout@${CHECKOUT_APP_URL}/_next/static/${location}/remoteEntry.js`,
  };
};
```

<p>
🎯 چگونه از آن استفاده می‌شود؟
در Host Application: </p>

```js
// import صفحات و کامپوننت‌های products
const ProductList = dynamic(() => import("product/products"));
const ProductDetail = dynamic(() => import("product/product"));
const SearchBar = dynamic(() => import("product/search"));
const LatestProducts = dynamic(() => import("product/latest-products"));

// استفاده در host
function HomePage() {
  return (
    <div>
      <SearchBar />
      <LatestProducts />
    </div>
  );
}
```

- transpilePackages: ["@repo/data-context", "@repo/ui", "@repo/utils"],
- transpilePackages: پکیج‌های مشترک بین microfrontend ها

```js
webpack(config, { isServer }) {
  config.plugins.push(
    new NextFederationPlugin({
      name: "host",  // نام این application
      remotes: remotes(isServer),  // microfrontend های remote
      filename: "static/chunks/remoteEntry.js",  // فایل exposed این host
      exposes: {},  // این host چیزی expose نمی‌کند (فقط consumer است)
    })
  );
  return config;
}

// هر microfrontend یک remoteEntry.js تولید می‌کند
product@http://localhost:3001/_next/static/chunks/remoteEntry.js
checkout@http://localhost:3002/_next/static/chunks/remoteEntry.js

// برای لود صفحات در host app
const ProductList = dynamic(() => import("product/ProductList"));
const CheckoutButton = dynamic(() => import("checkout/CheckoutButton"));

// =======================================================================

webpack(config, { isServer }) {
  config.plugins.push(
    new NextFederationPlugin({
      name: "checkout", // نام unique این microfrontend
      remotes: {}, // هیچ remote دیگری import نمی‌کند
      filename: "static/chunks/remoteEntry.js", // entry file
      exposes: {
        // 🎯 اینها چیزهایی هستند که برای host قابل استفاده می‌شوند
        "./checkout": "./pages/checkout", // صفحه checkout
        "./add-to-cart": "./components/add-to-cart", // کامپوننت افزودن به سبد
        "./cart-menu": "./components/cart-menu", // منوی سبد خرید
        "./pages-map": "./pages-map.js", // map کردن صفحات برای routing
      },
      extraOptions: {
        exposePages: true, // صفحات Next.js را automatically expose می‌کند
      },
    })
  );
  return config;
}

extraOptions: {
  exposePages: true, // صفحات Next.js را automatically expose می‌کند
}
// صفحات دیگر هم به صورت خودکار available می‌شوند
// مثلاً /checkout/success هم accessible می‌شود

```

#### 📁 ساختار فایل checkout.d.ts :

```js
declare module "checkout/checkout" {
  const CheckoutPage: React.ComponentType;
  export default CheckoutPage;
}

declare module "checkout/cart-menu" {
  const CartMenu: React.ComponentType<any>;
  export default CartMenu;
}

🗂️ معمولاً این فایل‌ها کجا قرار می‌گیرند؟
Option 1: در root پروژه

project/
  ├── src/
  │   ├── types/
  │   │   └── checkout.d.ts
  │   └── ...
  └── ...


  ⚙️ چگونه به TypeScript بگوییم این فایل را بخواند؟
در tsconfig.json:

{
  "compilerOptions": {
    "typeRoots": ["./node_modules/@types", "./types"],
    "types": ["checkout"]
  },
  "include": [
    "src/**/*",
    "types/**/*" // ✅ فایل‌های declaration را include کنید
  ]
}
```

#### pages-map.js file:فایل pages-map.js یک فایل mapping برای routing خودکار بین microfrontend ها است. این فایل به Module Federation می‌گوید که چگونه صفحات بین microfrontend های مختلف route شوند.

🎯 هدف اصلی
این فایل امکان integrated routing بین microfrontend های مستقل را فراهم می‌کند.

```js
export default {
  "/products": "./products", // مسیر /products به صفحه products مپ می‌شود
};

وقتی کاربر به آدرس: example.com/products می‌رود
↓
Module Federation می‌داند که این صفحه از microfrontend product می‌آید
↓
صفحه "./products" از product microfrontend load می‌شود


Host App (localhost:3000)
  ├── /products → از product microfrontend می‌آید
  ├── /checkout → از checkout microfrontend می‌آید
  └── /about → از host خودش می‌آید



  // در product microfrontend
export default {
  "/products": "./products", // لیست محصولات
  "/products/[id]": "./product", // جزئیات محصول
  "/search": "./search", // صفحه جستجو
};

// در checkout microfrontend
export default {
  "/checkout": "./checkout", // صفحه checkout
  "/checkout/success": "./checkout-success", // صفحه موفقیت
  "/cart": "./cart", // صفحه سبد خرید
};
```

#### Shared Dependencies :

```js
// همه microfrontend‌ها از همان نسخه react استفاده می‌کنند
new NextFederationPlugin({
  shared: {
    react: { singleton: true },
    "react-dom": { singleton: true },
  },
});
// معنی: فقط یک instance از این ماژول در کل application وجود داشته باشد

shared: {
  react: {
    singleton: true,
    requiredVersion: '^18.0.0' // حداقل نسخه ۱۸
  },
  'react-dom': {
    singleton: true,
    requiredVersion: '^18.0.0'
  }
}

shared: {
  react: {
    singleton: true,
    requiredVersion: '^18.0.0',
    version: '18.2.0' // اگر requiredVersion پیدا نشد، از این استفاده کن
  }
}

// next.config.js
shared: {
  react: { singleton: true },
  'react-dom': { singleton: true },
  '@repo/ui': { singleton: true } // design system مشترک
}


// همیشه از dynamic imports استفاده کنید
const AddToCart = dynamic(
  () => import("checkout/add-to-cart"),
  { loading: () => <div>Loading...</div> }
);

// برای loading حالت نمایش دهید
const AddToCart = dynamic(
  () => import("checkout/add-to-cart"),
  {
    loading: () => <button disabled>Adding...</button>,
    ssr: false // اگر نیاز به SSR نیست
  }
);
```

```js
# همه packages را در حالت development اجرا می‌کند
turbo dev --concurrency=12
🎯 این همان چیزی است که می‌خواهید! - همه اپ‌ها را با هم بالا می‌آورد



۴. Devهای جداگانه:

"dev:host": "turbo dev --filter=host --concurrency=12",
"dev:checkout": "turbo dev --filter=checkout --concurrency=12",
"dev:product": "turbo dev --filter=product --concurrency=12"

concurrency = تعداد عملیات همزمان

--concurrency=12 = ۱۲ process همزمان می‌تواند اجرا شود

یعنی Turbo می‌تواند همزمان ۱۲ package را process کند

🔢 مقایسه:
--concurrency=1 → خیلی آهسته (یکی یکی)

--concurrency=12 → سریع (۱۲ تا همزمان) ⚡

--concurrency=24 → بسیار سریع (اگر CPU قوی دارید)

✅ روش ۳: اگر می‌خواهید specific packages اجرا شوند
turbo dev --filter=host... --filter=checkout... --filter=product...
```

### ساختار پروژه :

```js
📦 monorepo/
 ├── 🎪 apps/
 │    ├── host/          # localhost:3000
 │    ├── product/       # localhost:3001
 │    ├── checkout/      # localhost:3002
 │    └── inspire/       # localhost:3003
 ├── 🧩 packages/
 │    ├── ui/           # shared components
 │    ├── utils/        # shared utilities
 │    └── data-context/ # shared state
 └── ⚙️ package.json
```
