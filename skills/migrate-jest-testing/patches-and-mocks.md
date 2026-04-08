# Patches and Mocks

## jsdom@26.1.0 patch

Create the file `patches/jsdom@26.1.0.patch` with this content:

```diff
diff --git a/lib/jsdom/browser/Window.js b/lib/jsdom/browser/Window.js
index 52d011cae61c3688ec64baa5cec411d55edbda9d..5b29f5af81331e49903514000959ea6e15b07ce6 100644
--- a/lib/jsdom/browser/Window.js
+++ b/lib/jsdom/browser/Window.js
@@ -507,7 +507,7 @@ function installOwnProperties(window, options) {
     // [LegacyUnforgeable]:
     window: { configurable: false },
     document: { configurable: false },
-    location: { configurable: false },
+    location: { configurable: true },
     top: { configurable: false }
   });
 
diff --git a/lib/jsdom/living/generated/Location.js b/lib/jsdom/living/generated/Location.js
index fc4d1dd48d4f34e05211b0351045f358e1b3a9fd..c855bd5b6fce6fc2d592841aba4ec2e8964e6819 100644
--- a/lib/jsdom/living/generated/Location.js
+++ b/lib/jsdom/living/generated/Location.js
@@ -322,19 +322,19 @@ function getUnforgeables(globalObject) {
       }
     });
     Object.defineProperties(unforgeables, {
-      assign: { configurable: false, writable: false },
-      replace: { configurable: false, writable: false },
-      reload: { configurable: false, writable: false },
-      href: { configurable: false },
-      toString: { configurable: false, writable: false },
-      origin: { configurable: false },
-      protocol: { configurable: false },
-      host: { configurable: false },
-      hostname: { configurable: false },
-      port: { configurable: false },
-      pathname: { configurable: false },
-      search: { configurable: false },
-      hash: { configurable: false }
+      assign: { configurable: true, writable: false },
+      replace: { configurable: true, writable: false },
+      reload: { configurable: true, writable: false },
+      href: { configurable: true },
+      toString: { configurable: true, writable: false },
+      origin: { configurable: true },
+      protocol: { configurable: true },
+      host: { configurable: true },
+      hostname: { configurable: true },
+      port: { configurable: true },
+      pathname: { configurable: true },
+      search: { configurable: true },
+      hash: { configurable: true }
     });
     unforgeablesMap.set(globalObject, unforgeables);
   }
```

## moo-color mock

Create the file `lib/__mocks__/moo-color.js` with this content:

```javascript
/**
 * Minimal MooColor mock for jest-canvas-mock.
 *
 * The real moo-color package can fail to resolve under pnpm.
 * This mock provides the minimal API that jest-canvas-mock needs.
 */
class MooColor {
  constructor(color) {
    this._color = color || '#000000';
  }

  toRgb() {
    return { r: 0, g: 0, b: 0 };
  }

  toHex() {
    return '#000000';
  }

  toHsl() {
    return { h: 0, s: 0, l: 0 };
  }

  toString() {
    return this._color;
  }
}

module.exports = MooColor;
module.exports.MooColor = MooColor;
module.exports.default = MooColor;
```
