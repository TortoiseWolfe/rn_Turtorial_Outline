## 1. Create & Configure the Project

### 1.1 Scaffold Vite (React + TypeScript)
```bash
npm create vite@latest steampunk-react-app -- --template react-ts
cd steampunk-react-app
npm install
```
### 1.2 Install Dev Dependencies
```bash
npm install -D tailwindcss @tailwindcss/vite postcss autoprefixer prettier gh-pages
```
### 1.3 Update Vite Config (`/vite.config.ts`)
```ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  base: '/steampunk-react-app/',
  plugins: [react(), tailwindcss()],
});
```
*(Use `base: '/'` if this is a user/organization site.)*

---

## 2. Tailwind Setup

### 2.1 Create Tailwind & PostCSS Config Files (Project Root)

```bash
touch tailwind.config.js postcss.config.js
```

**`/tailwind.config.js`**
```js
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {
      colors: {
        copper: "#B87333",
        bronze: "#CD7F32",
        gold: "#D4AF37",
        ivory: "#FFFFF0",
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

**`/postcss.config.js`**
```js
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};
```

### 2.2 Integrate Tailwind Directives in Existing CSS

**`/src/index.css`**  
Add the **@tailwind** directives **above** your custom CSS so Tailwind’s base/components load first. For example:

```css
/* Tailwind directives at the top */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* Your existing custom CSS follows */
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
  :root {
    color: #213547;
    background-color: #ffffff;
  }
  a:hover {
    color: #747bff;
  }
  button {
    background-color: #f9f9f9;
  }
}
```

### 2.3 Test Tailwind

Edit **`/src/App.tsx`** to use the custom colors/fonts:

```tsx
export default function App() {
  return (
    <div className="bg-copper p-6 min-h-screen">
      <h1 className="text-4xl font-special text-ivory">Steampunk Vite App</h1>
      <p className="text-gold mt-4">Hello from Arbutus Slab!</p>
      <p className="text-bronze font-cinzel mt-4">Cinzel for a classical look.</p>
    </div>
  );
}
```

Run `npm run dev` and confirm the styles are applied.

---

## 3. Storybook Setup

```bash
npx storybook@latest init --builder=vite
```
This creates **`.storybook`** with default configs and sample stories.

**`/.storybook/preview.js`**  
```js
import '../src/index.css';

export const parameters = {
  actions: { argTypesRegex: "^on[A-Z].*" },
  controls: { matchers: { color: /(background|color)$/i, date: /Date$/ } },
};
```
Then `npm run storybook` → http://localhost:6006

---

## 4. Prettier Setup

### 4.1 Create Files

```bash
touch .prettierrc .prettierignore
```

**`/.prettierrc`**
```json
{
  "singleQuote": true,
  "trailingComma": "es5",
  "printWidth": 80,
  "bracketSpacing": true
}
```

**`/.prettierignore`**
```
node_modules
dist
```

### 4.2 Add a Format Script

In **`package.json`**:
```json
"scripts": {
  "format": "prettier --write ."
}
```
Then run:
```bash
npm run format
```

---

## 5. GitHub Pages Deployment

### 5.1 Prepare Repository

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/TortoiseWolfe/steampunk-react-app.git
git push -u origin main
```

### 5.2 Add Deployment Scripts

In **`package.json`**:
```json
"scripts": {
  "dev": "vite",
  "build": "vite build",
  "preview": "vite preview",
  "predeploy": "npm run build",
  "deploy": "gh-pages -d dist"
}
```

### 5.3 Deploy

```bash
npm run deploy
```
Set **Settings → Pages** to `gh-pages` branch. Your site is at:
```
https://TortoiseWolfe.github.io/steampunk-react-app/
```

---

## Final Summary

1. **File Paths & Touch Commands** ensure everything is created in the **root** or **src/** as needed.  
2. **Tailwind Directives** go **at the top** of your `src/index.css` so the base/components load before your custom rules.  
3. **Storybook** is set up to import `../src/index.css`.  
4. **Prettier** is configured with `.prettierrc` & `.prettierignore`.  
5. **GitHub Pages** uses `"deploy": "gh-pages -d dist"` and `vite.config.ts` has `base: '/steampunk-react-app/'`.

Following these concise steps yields a complete, functional app with no missing pieces. Enjoy your steampunk-themed Vite project!
