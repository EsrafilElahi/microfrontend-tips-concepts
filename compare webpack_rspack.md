🔍 Webpack vs Rspack برای Microfrontend
✅ پاسخ کوتاه: بله، هر دو امکان‌پذیر هستند
اما تفاوت‌های مهمی وجود دارد!

🆚 مقایسه سریع

ویژگی Webpack Rspack
سرعت ⚡ خوب 🚀 بسیار عالی
حافظه 📦 بالا 💾 بسیار پایین
Compatibilit 💯 کامل 🔄 در حال بهبود
Pluginها 🧩 بسیار زیاد 🔌 محدود اما در حال رشد
Community 👥 بسیار بزرگ 👥 در حال رشد

### webpack :

```js
// next.config.js
const NextFederationPlugin = require("@module-federation/nextjs-mf");

module.exports = {
  webpack(config) {
    config.plugins.push(
      new NextFederationPlugin({
        name: "host",
        filename: "static/chunks/remoteEntry.js",
        remotes: {
          product: `product@http://localhost:3001/_next/static/chunks/remoteEntry.js`,
        },
      })
    );
    return config;
  },
};
```

### rspack :

```js
// next.config.js
const NextFederationPlugin = require("@module-federation/nextjs-mf/rspack");

module.exports = {
  // Rspack configuration
  experimental: {
    rspack: true,
  },
  webpack(config) {
    config.plugins.push(
      new NextFederationPlugin({
        name: "host",
        filename: "static/chunks/remoteEntry.js",
        remotes: {
          product: `product@http://localhost:3001/_next/static/chunks/remoteEntry.js`,
        },
      })
    );
    return config;
  },
};
```

```js
۱. سرعت Build ⚡
# Webpack
Build time: ~45 seconds

# Rspack
Build time: ~15 seconds  # 3x faster!

۲. مصرف حافظه 💾
# Webpack
Memory usage: ~4GB

# Rspack
Memory usage: ~1GB  # 75% less!
```

```js
const NextFederationPlugin = require("@module-federation/nextjs-mf/rspack");
const { withContentlayer } = require("next-contentlayer");

const PRODUCT_APP_URL =
  process.env.NEXT_PUBLIC_PRODUCT_APP_URL || "http://localhost:3001";

const CHECKOUT_APP_URL =
  process.env.NEXT_PUBLIC_CHECKOUT_APP_URL || "http://localhost:3002";

const remotes = (isServer) => {
  const location = isServer ? "ssr" : "chunks";
  return {
    product: `product@${PRODUCT_APP_URL}/_next/static/${location}/remoteEntry.js`,
    checkout: `checkout@${CHECKOUT_APP_URL}/_next/static/${location}/remoteEntry.js`,
  };
};

const nextConfig = {
  reactStrictMode: true,
  swcMinify: true,
  experimental: {
    scrollRestoration: true,
    rspack: true, // ✅ فعال کردن Rspack
  },
  images: {
    remotePatterns: [
      {
        protocol: "https",
        hostname: "**.unsplash.com",
      },
    ],
  },
  transpilePackages: ["@repo/data-context", "@repo/ui", "@repo/utils"],
  webpack(config, { isServer }) {
    config.plugins.push(
      new NextFederationPlugin({
        name: "host",
        remotes: remotes(isServer),
        filename: "static/chunks/remoteEntry.js",
        exposes: {},
        // ✅ options مخصوص Rspack (اختیاری)
        rspack: {
          enableRspackMode: true,
        },
      })
    );

    return config;
  },
};

module.exports = withContentlayer(nextConfig);


⚙️ پیکربندی اضافی برای بهینه‌سازی
در package.json:
{
  "scripts": {
    "dev": "next dev --rspack",
    "build": "next build --rspack",
    "start": "next start"
  }
}
```
