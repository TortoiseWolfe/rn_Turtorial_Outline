### File Structure Overview

```
steampunk-react-app/
├─ .storybook/
│   └─ preview.js
├─ node_modules/
├─ src/
│   ├─ App.tsx
│   ├─ main.tsx
│   └─ index.css
├─ index.html
├─ tailwind.config.js
├─ vite.config.ts
├─ package.json
├─ .prettierrc
├─ .prettierignore
└─ ...
```

---

## 1. Scaffold the Project

Open Git Bash and run:

```bash
npm create vite@latest steampunk-react-app -- --template react-ts
cd steampunk-react-app
npm install
```

Test the dev server:

```bash
npm run dev
```

Visit [http://localhost:5173](http://localhost:5173) to see the default React app.

---

## 2. Install Tailwind CSS (Official Vite Plugin Approach)

Install Tailwind CSS and its Vite plugin:

```bash
npm install -D tailwindcss @tailwindcss/vite
```

*(This approach does not require a separate PostCSS config.)*

---

## 3. Configure Vite and Tailwind

### 3a. Vite Configuration  
Create or edit **`vite.config.ts`** in the project root:

```bash
touch vite.config.ts
```

Then paste:

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

### 3b. Tailwind Configuration  
Create **`tailwind.config.js`** in the root:

```bash
touch tailwind.config.js
```

Then paste:

```js
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
export default {
  darkMode: 'class', // Enable class-based dark mode (add "dark" to <body> for dark mode)
  content: [
    './index.html',
    './src/**/*.{js,ts,jsx,tsx}',
  ],
  theme: {
    extend: {
      colors: {
        copper: {
          DEFAULT: '#B87333',
          dark: '#8D5A22'
        },
        bronze: {
          DEFAULT: '#CD7F32',
          dark: '#A85C28'
        },
        gold: {
          DEFAULT: '#D4AF37',
          dark: '#A67C27'
        },
        ivory: {
          DEFAULT: '#FFFFF0',
          dark: '#ECECEC'
        },
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

## 4. Set Up Tailwind and Custom CSS

### 4a. Create/Update **`/src/index.css`**

Create the file if it doesn’t exist:

```bash
touch src/index.css
```

Then paste the following content.  
**Important:** We keep your global light/dark styles intact (including the :root rule), so that your overall theme is preserved. To ensure Tailwind’s font utilities (e.g. `font-special`) override the global font-family, you’ll need to apply those classes specifically to elements (as shown in App.tsx). If needed, you can remove the global font-family from :root.

```css
/* src/index.css */

/* Global custom styles */
:root {
  font-family: Inter, system-ui, Avenir, Helvetica, Arial, sans-serif;
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

/* Custom styles */
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
  font-family: inherit;
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

/* Light mode adjustments using media query */
@media (prefers-color-scheme: light) {
  :root {
    color: #213547;
    background-color: #ffffff;
  }
}
```

### 4b. Create/Update **`/index.html`**

Create the file if it doesn’t exist:

```bash
touch index.html
```

Then paste:

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
    <!-- Link to your main CSS -->
    <link rel="stylesheet" href="/src/index.css" />
    <title>Steampunk React App</title>
  </head>
  <!-- Add "dark" class to body to force dark mode; remove it to start in light mode -->
  <body class="dark">
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

---

## 5. Update Your App Component

Edit **`/src/App.tsx`** to apply your custom Tailwind classes. Notice that we use the Tailwind font utility classes so they override the global :root font-family:

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
      {/* Container with Tailwind classes. Dark variants use the "dark:" prefix. */}
      <div className="bg-copper dark:bg-[theme('colors.copper.dark')] p-6 min-h-screen">
        <h1 className="text-4xl font-special text-ivory dark:text-[theme('colors.ivory.dark')]">
          Steampunk Vite App
        </h1>
        <p className="mt-4 text-gold dark:text-[theme('colors.gold.dark')]">
          Using Arbutus Slab → <span className="font-arbutus">Hello</span>
        </p>
        <p className="mt-4 text-bronze dark:text-[theme('colors.bronze.dark')]">
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
        <button onClick={() => setCount((c) => c + 1)}>
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

Also, ensure **`/src/main.tsx`** is:

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

Visit [http://localhost:5173](http://localhost:5173) and verify that your app displays with the copper background and that the headings (using `font-special`), spans (using `font-arbutus` and `font-cinzel`) show your custom Google Fonts. If the global :root font is still showing, consider removing the `font-family` property from :root or using more specific selectors in your components.

---

## 6. Set Up Storybook (with Vite Builder)

### 6a. Initialize Storybook

Run:

```bash
npx storybook@latest init --builder=vite
```

This creates a **`.storybook/`** folder with sample stories.

### 6b. Configure Storybook

Edit **`.storybook/preview.js`**:

```js
// .storybook/preview.js
import '../src/index.css';

export const parameters = {
  actions: { argTypesRegex: "^on[A-Z].*" },
  controls: { matchers: { color: /(background|color)$/i, date: /Date$/ } },
};
```

### 6c. Run Storybook

```bash
npm run storybook
```

Visit [http://localhost:6006](http://localhost:6006) to confirm the sample stories show your Tailwind styles and custom fonts.

---

## 7. Set Up Prettier

### 7a. Create Prettier Config Files

In the project root, run:

```bash
touch .prettierrc .prettierignore
```

Then set their contents:

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

### 7b. Add a Format Script

In **`package.json`**, under "scripts", add:

```json
"format": "prettier --write ."
```

Then run:

```bash
npm run format
```

---

## 8. GitHub Pages Deployment

### 8a. Prepare Your Repository

Initialize Git and push to GitHub:

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/TortoiseWolfe/steampunk-react-app.git
git push -u origin main
```

### 8b. Add Deployment Scripts

In **`package.json`**, add/update under "scripts":

```json
{
  "dev": "vite",
  "build": "vite build",
  "preview": "vite preview",
  "predeploy": "npm run build",
  "deploy": "gh-pages -d dist"
}
```

### 8c. Deploy the App

Run:

```bash
npm run deploy
```

Then, in your GitHub repo’s **Settings → Pages**, select the `gh-pages` branch as the source. Your site will be available at:

```
https://TortoiseWolfe.github.io/steampunk-react-app/
```

---

## Final Recap

- **Root Files:**  
  - **index.html:** Loads Google Fonts and `src/index.css`.  
  - **tailwind.config.js:** Defines steampunk colors (with dark variants) and custom fonts.  
  - **vite.config.ts:** Configures Vite with the Tailwind plugin and sets the base URL.
  - **.prettierrc** and **.prettierignore** are in the root.
- **/src Folder:**  
  - **index.css:** Starts with global custom styles (including your light/dark settings) then loads Tailwind’s utilities so that Tailwind font utilities override the global defaults.
  - **App.tsx:** Uses Tailwind classes (`font-special`, `font-arbutus`, `font-cinzel`) to apply your custom fonts.
- **Storybook, Prettier, and GitHub Pages** are fully configured and deployable.

By following these steps exactly, you’ll have a complete steampunk-themed Vite + React project that supports both light and dark modes and uses your custom Google Fonts. This tutorial provides every file and command with no skipped steps. Enjoy your project!
