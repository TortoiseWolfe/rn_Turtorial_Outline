## File Structure Overview

```
steampunk-react-app/
├─ .storybook/
│   └─ preview.js
├─ node_modules/
├─ public/            // (if any)
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

Test the development server:

```bash
npm run dev
```

Visit [http://localhost:5173](http://localhost:5173) to verify the default React app runs.

---

## 2. Install Tailwind CSS via the Official Vite Plugin

Install the necessary packages:

```bash
npm install -D tailwindcss @tailwindcss/vite
```

*(Do not install a PostCSS config for Tailwind with this approach. Remove any postcss.config.js that references tailwindcss if present.)*

---

## 3. Configure Vite and Tailwind

### 3a. Create/Edit Vite Config

Create (or open) the file **`/vite.config.ts`** in the root and set it up as follows:

```ts
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import tailwind from '@tailwindcss/vite';

export default defineConfig({
  // Since this is a project site, set base to '/steampunk-react-app/'
  base: '/steampunk-react-app/',
  plugins: [react(), tailwind()],
});
```

### 3b. Create Tailwind Configuration

In the root, create **`tailwind.config.js`**:

```bash
touch tailwind.config.js
```

Then add:

```js
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    './index.html',
    './src/**/*.{js,ts,jsx,tsx}',
  ],
  theme: {
    extend: {
      colors: {
        copper: '#B87333',
        bronze: '#CD7F32',
        gold:   '#D4AF37',
        ivory:  '#FFFFF0',
      },
      fontFamily: {
        special: ['"Special Elite"', 'cursive'],   // Typewriter-style
        arbutus: ['"Arbutus Slab"', 'serif'],        // Bold slab serif
        cinzel:  ['Cinzel', 'serif'],                // Classical style
      },
    },
  },
  plugins: [],
};
```

---

## 4. Set Up Tailwind in Your CSS

### 4a. Create/Update CSS File

In **`/src/index.css`**, create or update the file:

```bash
touch src/index.css
```

Then set its contents to load Tailwind and (optionally) your custom CSS. **Important:** If you want your Tailwind font classes to work, do not override the font-family on the root element. Either remove or adjust the :root rule.

Here’s an example that puts Tailwind directives at the top and then your custom styles without overriding font settings:

```css
/* src/index.css */

/* Load Tailwind base, components, and utilities */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* Custom CSS below. Remove or comment out the font-family from :root
   so that Tailwind's font classes (like font-special) take effect. */

/*
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
*/

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
  /* If you need dark/light adjustments, add here */
}
```

### 4b. Load Google Fonts in HTML

In **`/index.html`** in the root, ensure you load the Google Fonts in the `<head>`:

```html
<!-- index.html -->
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <!-- Load Google Fonts: Special Elite, Arbutus Slab, Cinzel -->
    <link
      href="https://fonts.googleapis.com/css2?family=Special+Elite&family=Arbutus+Slab&family=Cinzel&display=swap"
      rel="stylesheet"
    />
    <!-- Link to main CSS -->
    <link rel="stylesheet" href="/src/index.css" />
    <title>Steampunk React App</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

---

## 5. Test Your App

Edit **`/src/App.tsx`** to use your custom Tailwind classes:

```tsx
// src/App.tsx
import { useState } from 'react';

function App() {
  const [count, setCount] = useState(0);

  return (
    <>
      <div className="bg-copper p-6 min-h-screen">
        <h1 className="text-4xl font-special text-ivory">Steampunk Vite App</h1>
        <p className="mt-4 text-gold">
          Using Arbutus Slab → <span className="font-arbutus">Hello</span>
        </p>
        <p className="mt-4 text-bronze">
          Using Cinzel → <span className="font-cinzel">Classical vibes</span>
        </p>
      </div>
      <h1>Vite + React</h1>
      <div>
        <button onClick={() => setCount((c) => c + 1)}>count is {count}</button>
        <p>Edit <code>src/App.tsx</code> and save to test HMR</p>
      </div>
    </>
  );
}

export default App;
```

Then run:

```bash
npm run dev
```

Visit [http://localhost:5173](http://localhost:5173) and verify you see the copper background and—critically—the headings and spans display in your custom fonts. (If you still see the default font, double-check that you removed the :root font-family override.)

---

## 6. Set Up Storybook (with Vite Builder)

### 6a. Initialize Storybook

```bash
npx storybook@latest init --builder=vite
```

This creates a **`.storybook/`** folder with sample configuration and stories.

### 6b. Configure Storybook

In **`.storybook/preview.js`**:

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

Open [http://localhost:6006](http://localhost:6006) and verify that the sample stories display with your Tailwind styling (including your custom fonts).

---

## 7. Set Up Prettier

### 7a. Create Prettier Config Files in the Root

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

### 7b. Add Format Script to `package.json`

In your **`package.json`**, add:

```json
"scripts": {
  "format": "prettier --write ."
}
```

Run Prettier with:

```bash
npm run format
```

---

## 8. GitHub Pages Deployment

### 8a. Prepare Git Repository

In the project root, run:

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/TortoiseWolfe/steampunk-react-app.git
git push -u origin main
```

### 8b. Add Deployment Scripts to `package.json`

Add these scripts (if not already present):

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

- **Project Root Files:**  
  - `/index.html` loads Google Fonts and `/src/index.css`.  
  - `/tailwind.config.js` defines custom colors and fonts.  
  - `/vite.config.ts` sets the base URL and includes the Tailwind Vite plugin.
- **/src/ Folder:**  
  - `index.css` starts with `@import "tailwindcss";` then your custom CSS (ensure no overriding :root font-family).  
  - `App.tsx` uses classes like `font-special`, `font-arbutus`, and `font-cinzel`.
- **Storybook, Prettier, and GitHub Pages** are all set up via the provided scripts.

Follow these steps exactly and remove any overriding global font rules if needed. This yields a complete, functional steampunk-themed project with custom fonts, Storybook, Prettier, and deployment. Enjoy your project!
