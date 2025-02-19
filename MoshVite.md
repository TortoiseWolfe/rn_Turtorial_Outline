### **File Structure Overview**

```
steampunk-react-app/
├─ .storybook/
│   └─ preview.js
├─ node_modules/
├─ src/
│   ├─ App.tsx
│   ├─ main.tsx
│   ├─ index.css
│   └─ App.css  (if any additional styles)
├─ index.html
├─ tailwind.config.js
├─ vite.config.ts
├─ package.json
├─ .prettierrc
├─ .prettierignore
└─ ...
```

---

## **1. Scaffold the Project**

Run the following commands in Git Bash:

```bash
npm create vite@latest steampunk-react-app -- --template react-ts
cd steampunk-react-app
npm install
```

Test the development server:

```bash
npm run dev
```

Visit **http://localhost:5173** to see the default app.

---

## **2. Install Tailwind CSS (Official Vite Plugin Approach)**

Install Tailwind CSS and its Vite plugin:

```bash
npm install -D tailwindcss @tailwindcss/vite
```

*(No separate PostCSS config is needed.)*

---

## **3. Configure Vite & Tailwind**

### **3a. Vite Configuration**

Create or edit **`vite.config.ts`** in the root:

```bash
touch vite.config.ts
```

Paste the following:

```ts
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import tailwind from '@tailwindcss/vite';

export default defineConfig({
  base: '/steampunk-react-app/', // Use '/' if deploying as a user/organization site.
  plugins: [react(), tailwind()],
});
```

### **3b. Tailwind Configuration**

Create **`tailwind.config.js`** in the root:

```bash
touch tailwind.config.js
```

Paste:

```js
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
export default {
  darkMode: 'class', // Enable dark mode via a "dark" class on <body>
  content: [
    './index.html',
    './src/**/*.{js,ts,jsx,tsx}',
  ],
  theme: {
    extend: {
      colors: {
        copper: { DEFAULT: '#B87333', dark: '#8D5A22' },
        bronze: { DEFAULT: '#CD7F32', dark: '#A85C28' },
        gold:   { DEFAULT: '#D4AF37', dark: '#A67C27' },
        ivory:  { DEFAULT: '#FFFFF0', dark: '#ECECEC' },
      },
      fontFamily: {
        special: ['"Special Elite"', 'cursive'],
        arbutus: ['"Arbutus Slab"', 'serif'],
        cinzel:  ['Cinzel', 'serif'],
      },
    },
  },
  plugins: [],
};
```

---

## **4. Set Up Tailwind & Custom CSS**

### **4a. Update `/src/index.css`**

Create or edit **`src/index.css`**:

```bash
touch src/index.css
```

Paste the following content. Notice that we do **not** set a global font-family so Tailwind’s font utilities can work, and we preserve your existing light/dark settings:

```css
/* src/index.css */

/* Global custom styles */
:root {
  line-height: 1.5;
  font-weight: 400;
  color-scheme: light dark;
  color: rgba(255, 255, 255, 0.87);
  background-color: #242424;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* Load Tailwind's base, components, and utilities */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* Force Tailwind font utilities to override any inherited settings */
.font-special {
  font-family: "Special Elite", cursive !important;
}
.font-arbutus {
  font-family: "Arbutus Slab", serif !important;
}
.font-cinzel {
  font-family: "Cinzel", serif !important;
}

/* Your additional custom styles remain intact */
a {
  font-weight: 500;
  color: #646cff;
  text-decoration: inherit;
}
a:hover {
  color: #535bf2;
}

body {
  margin: 0;
  display: flex;
  place-items: center;
  min-width: 320px;
  min-height: 100vh;
}

h1 {
  font-size: 3.2em;
  line-height: 1.1;
}

button {
  border-radius: 8px;
  border: 1px solid transparent;
  padding: 0.6em 1.2em;
  font-size: 1em;
  font-weight: 500;
  background-color: #1a1a1a;
  cursor: pointer;
  transition: border-color 0.25s;
}
button:hover {
  border-color: #646cff;
}
button:focus,
button:focus-visible {
  outline: 4px auto -webkit-focus-ring-color;
}

/* Light mode adjustments */
@media (prefers-color-scheme: light) {
  :root {
    color: #213547;
    background-color: #ffffff;
  }
}
```

### **4b. Update `/index.html`**

Create or edit **`index.html`**:

```bash
touch index.html
```

Paste:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <!-- Load Google Fonts: Special Elite, Arbutus Slab, and Cinzel -->
    <link
      href="https://fonts.googleapis.com/css2?family=Special+Elite&family=Arbutus+Slab&family=Cinzel&display=swap"
      rel="stylesheet"
    />
    <link rel="stylesheet" href="/src/index.css" />
    <title>Steampunk React App</title>
  </head>
  <!-- For dark mode, the "dark" class is added; remove to start in light mode -->
  <body class="dark">
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

---

## **5. Restore Your Original App Component**

Keep your original **`src/App.tsx`** content exactly as you provided:

```tsx
// src/App.tsx
import { useState } from "react";
import reactLogo from "./assets/react.svg";
import viteLogo from "/vite.svg";
import "./App.css";

function App() {
  const [count, setCount] = useState(0);

  return (
    <>
      <div className="bg-copper dark:bg-copper-dark p-6">
        <h1 className="text-4xl font-special text-ivory dark:text-ivory-dark">
          Steampunk Vite App
        </h1>
        <p className="mt-4 text-gold dark:text-gold-dark">
          Using Arbutus Slab → <span className="font-arbutus">Hello</span>
        </p>
        <p className="mt-4 text-bronze dark:text-bronze-dark">
          Using Cinzel → <span className="font-cinzel">Classical vibes</span>
        </p>
        <a href="https://vite.dev" target="_blank">
          <img src={viteLogo} className="logo" alt="Vite logo" />
        </a>
        <a href="https://react.dev" target="_blank">
          <img src={reactLogo} className="logo react" alt="React logo" />
        </a>
      </div>
      <h1>Vite + React</h1>
      <div className="card">
        <button onClick={() => setCount((count) => count + 1)}>
          count is {count}
        </button>
        <p>
          Edit <code>src/App.tsx</code> and save to test HMR
        </p>
      </div>
      <p className="read-the-docs">
        Click on the Vite and React logos to learn more
      </p>
    </>
  );
}

export default App;
```

*Note:* We removed the `min-h-screen` class from the hero section so that the counter and additional content appear on the screen without being pushed below. (If you need a full-screen hero, you can adjust the layout accordingly.)

Also, ensure **`src/main.tsx`** is:

```tsx
// src/main.tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import './index.css';

ReactDOM.createRoot(document.getElementById('root') as HTMLElement).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

Run the dev server:
```bash
npm run dev
```
Verify on **http://localhost:5173** that all content is visible and your steampunk fonts are correctly applied.

---

## **6. Set Up Storybook**

Initialize Storybook:
```bash
npx storybook@latest init --builder=vite
```

In **`.storybook/preview.js`**:
```js
import '../src/index.css';
export const parameters = {
  actions: { argTypesRegex: "^on[A-Z].*" },
  controls: { matchers: { color: /(background|color)$/i, date: /Date$/ } },
};
```

Run Storybook:
```bash
npm run storybook
```
Verify Storybook displays your components with the correct styling.

---

## **7. Set Up Prettier**

Create Prettier config files:
```bash
touch .prettierrc .prettierignore
```

**`.prettierrc`**:
```json
{
  "singleQuote": true,
  "trailingComma": "es5",
  "printWidth": 80,
  "bracketSpacing": true
}
```

**`.prettierignore`**:
```
node_modules
dist
```

Add a format script to **`package.json`**:
```json
"format": "prettier --write ."
```

Run:
```bash
npm run format
```

---

## **8. Deploy to GitHub Pages**

Initialize Git and push to GitHub:
```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/TortoiseWolfe/steampunk-react-app.git
git push -u origin main
```

Add deployment scripts to **`package.json`**:
```json
{
  "dev": "vite",
  "build": "vite build",
  "preview": "vite preview",
  "predeploy": "npm run build",
  "deploy": "gh-pages -d dist"
}
```

Deploy the app:
```bash
npm run deploy
```

Then, in your GitHub repo’s **Settings → Pages**, select the `gh-pages` branch as the source. Your site will be live at:
```
https://TortoiseWolfe.github.io/steampunk-react-app/
```

---

### **Final Recap**

- **Global Styles:** Your original global styles (including light/dark settings) are preserved without the global `font-family` line, so Tailwind’s font utilities can work.
- **Tailwind Overrides:** The extra CSS for `.font-special`, `.font-arbutus`, and `.font-cinzel` (with `!important`) forces your custom fonts to display.
- **Original App Content:** Your original `App.tsx` content—including the counter button—remains intact.
- **Storybook, Prettier, and GitHub Pages:** All are configured and deployable.

This complete tutorial merges everything as requested without removing your original content, ensuring your steampunk fonts display properly and all content is visible. Enjoy your project!
