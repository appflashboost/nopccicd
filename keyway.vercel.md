# How to Deploy a Basic Next.js App on Vercel: A Troubleshooting Guide

This document outlines the key steps and common pitfalls encountered when deploying a Next.js application to Vercel, based on a real-world troubleshooting session. Following these guidelines can help you avoid common errors like 404 pages and build failures.

## 1. Ensure Dependency Stability

One of the most common sources of unpredictable errors is using unstable, pre-release versions of frameworks.

*   **Problem:** We initially used experimental versions of Next.js (v16) and React (v19). This led to a cascade of cryptic errors, including `ENOENT` (file not found) errors during runtime and bundler crashes (Turbopack).
*   **Solution:** Downgrade your core dependencies to the latest stable versions. In our case, we reverted to Next.js `14.2.4` and React `18`.

**Action:** In your `package.json`, specify stable versions for `next`, `react`, and `react-dom`, and ensure related packages like `@types/react` and `eslint-config-next` are compatible.

```json
"dependencies": {
  "react": "18",
  "react-dom": "18",
  "next": "14.2.4"
}
```

## 2. Avoid Routing Conflicts

Next.js has two routing systems: the older Pages Router (`pages` directory) and the newer App Router (`app` directory). Using both simultaneously without proper configuration can cause conflicts.

*   **Problem:** The project contained both a `pages/index.tsx` and an `app/page.tsx`. This created ambiguity, leading to 404 errors because the App Router takes precedence, but the build system can still be confused.
*   **Solution:** Choose one router and stick with it. For modern applications, the App Router is recommended.

**Action:** If you intend to use the App Router, delete the `pages` directory entirely.

## 3. Configure Your Project for Vercel

Vercel is highly optimized for Next.js, but explicit configuration can resolve many deployment-specific issues.

### a. Specify the Node.js Version

*   **Problem:** Our Vercel build failed because the required Node.js version was not met. Vercel discontinues older versions and requires updates.
*   **Solution:** Explicitly set the Node.js version in your project's configuration.

**Action:** Add an `engines` field to your `package.json` with the version Vercel requires (as of this writing, "22.x").

```json
"engines": {
  "node": "22.x"
}
```

### b. Use the Standalone Output Mode

*   **Problem:** By default, a Next.js build might not include all necessary files in a serverless environment, which can contribute to 404 errors.
*   **Solution:** Configure your Next.js application to produce a standalone build.

**Action:** In your `next.config.js`, add the `output: 'standalone'` option.

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'standalone',
};

module.exports = nextConfig;
```

### c. Create a Minimal `vercel.json`

*   **Problem:** Sometimes, Vercel might not automatically detect the project as a Next.js application, leading to incorrect build steps.
*   **Solution:** Add a `vercel.json` file to explicitly tell Vercel which builder to use.

**Action:** Create a `vercel.json` file in your project root with the following content. **Do not add routing rules**, as this can break static asset loading.

```json
{
  "version": 2,
  "builds": [
    {
      "src": "next.config.js",
      "use": "@vercel/next"
    }
  ]
}
```

By following these steps, you can create a stable, deployable Next.js application and avoid the most common issues that lead to 404 errors and build failures on Vercel.
