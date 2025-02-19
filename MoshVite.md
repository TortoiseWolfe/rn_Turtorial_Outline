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

Test the dev server with:

```bash
npm run dev
```

Visit [http://localhost:5173](http://localhost:5173) to verify the default React app runs.

---

## 2. Install Tailwind CSS via the Official Vite Plugin

Install Tailwind CSS and its Vite plugin:

```bash
npm install -D tailwindcss @tailwindcss/vite
```

*(No separate PostCSS config is needed with this approach.)*

---

## 3. Configure Vite and Tailwind

### 3a. Vite Configuration

Create or edit **`vite.config.ts`** in the project root:

```bash
touch vite.config.ts
```

Then add:

```ts
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import tailwind from '@tailwindcss/vite';

export default defineConfig({
  // For a GitHub project site, base is set to the repo name:
  base: '/steampunk-react-app/',
  plugins: [react(), tailwind()],
});
```

### 3b. Tailwind Configuration

Create **`tailwind.config.js`** in the project root:

```bash
touch tailwind.config.js
```

Then add:

```js
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
export default {
  darkMode: 'class', // Use class-based dark mode (add "dark" to <body> for dark mode)
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

### 4a. `/src/index.css`

Create or update **`src/index.css`**:

```bash
touch src/index.css
```

Paste the following content. Note that we place our global custom styles (including our light/dark defaults) first; then Tailwind’s directives load, allowing Tailwind’s utility classes to override the global styles when used:

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

*The `@media (prefers-color-scheme: light)` block here provides complete light mode settings. Adjust these values as desired.*

### 4b. `/index.html`

Create or update **`index.html`** in the project root:

```bash
touch index.html
```

Paste the following:

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
  <!-- To start in dark mode, add the "dark" class to <body>; remove it for light mode or toggle dynamically -->
  <body class="dark">
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

---

## 5. Update Your App Component

Edit **`src/App.tsx`**:

```tsx
// src/App.tsx
import { useState } from 'react';
import reactLogo from './assets/react.svg';
import viteLogo from '/vite.svg';
import './App.css';

function App() {
  const [count, setCount] = useState(0);

  return (
    <>
      {/* Container with Tailwind classes. Dark variants are applied using the "dark:" prefix. */}
      <div className="bg-copper dark:bg-copper-dark p-6 min-h-screen">
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

Ensure **`src/main.tsx`** is as follows:

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

Visit [http://localhost:5173](http://localhost:5173) to verify your app displays with a copper background, custom fonts (overriding the global :root defaults), and proper light/dark adjustments.

---

## 6. Set Up Storybook (with Vite Builder)

### 6a. Initialize Storybook

Run:

```bash
npx storybook@latest init --builder=vite
```

This creates the **`.storybook/`** folder with configuration and sample stories.

### 6b. Configure Storybook

Edit **`.storybook/preview.js`**:

```js
// .storybook/preview.js
import '../src/index.css';

export const parameters = {
  actions: { argTypesRegex: "^on[A-Z].*" },
  controls: {
    matchers: { color: /(background|color)$/i, date: /Date$/ },
  },
};
```

### 6c. Run Storybook

```bash
npm run storybook
```

Visit [http://localhost:6006](http://localhost:6006) to verify that sample stories display with your Tailwind styles and custom fonts.

---

## 7. Set Up Prettier

### 7a. Create Prettier Config Files

In the root, run:

```bash
touch .prettierrc .prettierignore
```

Then add:

**`.prettierrc`:**

```json
{
  "singleQuote": true,
  "trailingComma": "es5",
  "printWidth": 80,
  "bracketSpacing": true
}
```

**`.prettierignore`:**

```
node_modules
dist
```

### 7b. Add a Format Script

In **`package.json`**, add under "scripts":

```json
"format": "prettier --write ."
```

Run Prettier:

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

In **`package.json`**, add/update:

```json
"scripts": {
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

Then, in your GitHub repository’s **Settings → Pages**, set the source to the `gh-pages` branch. Your site will be available at:

```
https://TortoiseWolfe.github.io/steampunk-react-app/
```

---

## Final Recap

- **Root Files:**  
  - **index.html** loads Google Fonts and `src/index.css`.  
  - **tailwind.config.js** defines steampunk colors (with dark variants) and custom fonts.  
  - **vite.config.ts** sets up Vite with the Tailwind plugin and the proper base URL.
  - **.prettierrc** and **.prettierignore** are in the root.
- **/src Folder:**  
  - **index.css** starts with global custom styles (including a light/dark theme and a complete `@media` block for light mode) then loads Tailwind directives so that Tailwind utility classes (like `font-special`) override the global defaults.
  - **App.tsx** uses Tailwind classes to apply your custom fonts and colors.
- **Storybook, Prettier, and GitHub Pages** are fully configured and deployable.

This tutorial includes every necessary step and file, including a complete light mode adjustment block instead of a placeholder comment. Follow these instructions exactly for a fully functional steampunk-themed project with light/dark support and custom fonts. Happy coding!
