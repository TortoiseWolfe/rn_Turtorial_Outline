> **Important:**  
> We preserve your global light/dark styles in the custom CSS. To ensure Tailwind’s utility classes (for fonts) override the global defaults, we place the global `:root` block at the very top of `src/index.css`. We also enable dark mode in Tailwind and define dark variants for your custom steampunk colors.

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

## 2. Install Tailwind CSS (Using the Official Vite Plugin)

Install Tailwind and its Vite plugin:

```bash
npm install -D tailwindcss @tailwindcss/vite
```

*Note: With this approach you don’t need a separate PostCSS config for Tailwind.*

---

## 3. Configure Vite and Tailwind

### 3a. Vite Configuration  
Create (or edit) **`/vite.config.ts`** in the project root:

```bash
touch vite.config.ts
```

Then add the following:

```ts
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import tailwind from '@tailwindcss/vite';

export default defineConfig({
  // Set base for a GitHub project site
  base: '/steampunk-react-app/',
  plugins: [react(), tailwind()],
});
```

### 3b. Tailwind Configuration  
Create **`/tailwind.config.js`** in the project root:

```bash
touch tailwind.config.js
```

Then add:

```js
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
export default {
  darkMode: 'class', // Enable dark mode using a class (add "dark" to <body> for dark mode)
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

Then paste the following content:

```css
/* src/index.css */

/* Global custom styles are defined first so that Tailwind utilities override them when used. */
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

/* Load Tailwind base, components, and utilities */
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

@media (prefers-color-scheme: light) {
  /* Light mode adjustments, if any */
}
```

### 4b. Create/Update **`/index.html`**

Create the file if needed:

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
    <!-- Link to the main CSS -->
    <link rel="stylesheet" href="/src/index.css" />
    <title>Steampunk React App</title>
  </head>
  <!-- For demonstration, the dark mode is activated by adding the "dark" class to the body -->
  <body class="dark">
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

*Tip: Remove the `class="dark"` from `<body>` if you want to start in light mode and toggle dark mode dynamically.*

---

## 5. Update Your App Component

Edit **`/src/App.tsx`**:

```tsx
// src/App.tsx
import { useState } from "react";
import reactLogo from "./assets/react.svg";
import viteLogo from "/vite.svg";
import "./App.css"; // If you have additional styles

function App() {
  const [count, setCount] = useState(0);

  return (
    <>
      {/* The container uses Tailwind classes.
          Dark variants are applied using the "dark:" prefix. */}
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

Also, ensure **`/src/main.tsx`** looks like:

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

Visit [http://localhost:5173](http://localhost:5173) and confirm your app displays with a copper background, and the headings and spans use the custom fonts (overriding the global :root settings when using Tailwind utility classes).

---

## 6. Set Up Storybook (with Vite Builder)

### 6a. Initialize Storybook

Run:

```bash
npx storybook@latest init --builder=vite
```

This creates a **`.storybook/`** folder with sample stories and configuration.

### 6b. Configure Storybook

In **`.storybook/preview.js`**, add:

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

Visit [http://localhost:6006](http://localhost:6006) to confirm the sample stories display with your Tailwind styles and custom fonts.

---

## 7. Set Up Prettier

### 7a. Create Prettier Config Files

In the project root, run:

```bash
touch .prettierrc .prettierignore
```

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

Then, in your GitHub repository’s **Settings → Pages**, select the `gh-pages` branch as the source. Your site will be available at:

```
https://TortoiseWolfe.github.io/steampunk-react-app/
```

---

## Final Recap

- **Project Root Files:**  
  - `index.html` loads Google Fonts and `src/index.css`.  
  - `tailwind.config.js` defines your steampunk colors (with dark variants) and custom fonts.  
  - `vite.config.ts` configures Vite with the official Tailwind plugin and sets the base URL.
  - `.prettierrc` and `.prettierignore` are in the root.
- **/src Folder:**  
  - `index.css` starts with global styles (including a light/dark theme) and then loads Tailwind utilities so that Tailwind font classes (like font-special, font-arbutus, font-cinzel) override the global default when applied.
  - `App.tsx` uses Tailwind classes to apply your custom fonts and colors.
- **Storybook, Prettier, and GitHub Pages** are fully configured.

Follow these steps exactly to get a complete, functional steampunk-themed Vite + React project with a light/dark mode and custom fonts. Enjoy your project!
