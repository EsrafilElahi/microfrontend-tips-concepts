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

### react vite :

```js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import federation from "@originjs/vite-plugin-federation";

const PRODUCT_APP_URL =
  process.env.VITE_PRODUCT_APP_URL || "http://localhost:3001";
const CHECKOUT_APP_URL =
  process.env.VITE_CHECKOUT_APP_URL || "http://localhost:3002";

export default defineConfig({
  plugins: [
    react(),
    federation({
      name: "host",
      remotes: {
        product: `product@${PRODUCT_APP_URL}/assets/remoteEntry.js`,
        checkout: `checkout@${CHECKOUT_APP_URL}/assets/remoteEntry.js`,
      },
      shared: ["react", "react-dom"],
      filename: "remoteEntry.js",
    }),
  ],
  server: {
    port: 3000,
  },
});



// vite.config.js در پروژه product
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import federation from '@originjs/vite-plugin-federation';

export default defineConfig({
  plugins: [
    react(),
    federation({
      name: 'product',
      filename: 'remoteEntry.js',
      exposes: {
        './ProductComponent': './src/ProductComponent.jsx',
      },
      shared: ['react', 'react-dom'],
    }),
  ],
  server: {
    port: 3001,
  },
});






import { federation } from '@module-federation/vite';
module.exports = {
  server: {
    origin: 'http://localhost:2000',
    port: 2000,
  },
  base: "http://localhost:2000",
  plugins: [
    federation({
      name: 'vite_provider',
      manifest: true,
      remotes: {
        esm_remote: {
          type: "module",
          name: "esm_remote",
          entry: "https://[...]/remoteEntry.js",
        },
        var_remote: "var_remote@https://[...]/remoteEntry.js",
      },
      exposes: {
        './button': './src/components/button',
      },
      shared: {
        react: {
          singleton: true,
        },
        'react/': {
          singleton: true,
        },
      },
    }),
  ],
  // Do you need to support build targets lower than chrome89?
  // You can use 'vite-plugin-top-level-await' plugin for that.
  build: {
    target: 'chrome89',
  },
};
```

### 1. فایل vite.config.ts

```js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import federation from "@originjs/vite-plugin-federation";

const PRODUCT_APP_URL =
  import.meta.env.VITE_PRODUCT_APP_URL || "http://localhost:3001";

const CHECKOUT_APP_URL =
  import.meta.env.VITE_CHECKOUT_APP_URL || "http://localhost:3002";

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    react(),
    federation({
      name: "host",
      remotes: {
        product: `${PRODUCT_APP_URL}/assets/remoteEntry.js`,
        checkout: `${CHECKOUT_APP_URL}/assets/remoteEntry.js`,
      },
      shared: ["react", "react-dom"],
    }),
  ],
  build: {
    target: "esnext",
    minify: false,
    cssCodeSplit: false,
  },
  server: {
    port: 3000,
  },
  preview: {
    port: 3000,
  },
});
```

### 2. فایل src/App.tsx

```js
import React, { Suspense } from "react";
import "./App.css";

// Lazy load remote components
const ProductApp = React.lazy(() => import("product/ProductApp"));
const CheckoutApp = React.lazy(() => import("checkout/CheckoutApp"));

function App() {
  return (
    <div className="app">
      <h1>Host Application</h1>

      <Suspense fallback={<div>Loading Product...</div>}>
        <ProductApp />
      </Suspense>

      <Suspense fallback={<div>Loading Checkout...</div>}>
        <CheckoutApp />
      </Suspense>
    </div>
  );
}

export default App;
```

### 7. فایل package.json

```js
{
  "name": "host-app",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@originjs/vite-plugin-federation": "^1.3.0",
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "@vitejs/plugin-react": "^4.0.0",
    "typescript": "^5.0.0",
    "vite": "^4.4.0"
  }
}


نکات مهم:
تفاوت محیط متغیرها: در Vite از import.meta.env به جای process.env استفاده می‌شود

پیشوند متغیرها: از VITE_ به جای NEXT_PUBLIC_ استفاده کنید

مسیر remoteEntry: در Vite معمولاً در /assets/remoteEntry.js قرار می‌گیرد

TypeScript: حتماً نوع‌های @originjs/vite-plugin-federation را include کنید

Build Config: برای Module Federation نیاز به target: 'esnext' دارید
```
