## **File Structure Overview**
```
steampunk-react-app/
├─ .storybook/
│   └─ preview.js
├─ node_modules/
├─ src/
│   ├─ App.tsx
│   ├─ main.tsx
│   ├─ index.css
│   ├─ App.css
├─ index.html
├─ tailwind.config.js
├─ vite.config.ts
├─ package.json
├─ .prettierrc
├─ .prettierignore
└─ ...
```

---

# **1. Scaffold the Project**
Open Git Bash and run:

```bash
npm create vite@latest steampunk-react-app -- --template react-ts
cd steampunk-react-app
npm install
```

Run the dev server:

```bash
npm run dev
```

Visit [http://localhost:5173](http://localhost:5173) to see the default React app.

---

# **2. Install Tailwind CSS**
Install Tailwind and the official Vite plugin:

```bash
npm install -D tailwindcss @tailwindcss/vite
```

> *This approach does **not** require a separate `postcss.config.js`.*

---

# **3. Configure Vite and Tailwind**

### **3a. Vite Configuration**
Create (or edit) **`vite.config.ts`** in the root:

```bash
touch vite.config.ts
```

Paste:

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

# **4. Set Up Tailwind & Custom CSS**

### **4a. Create/Update `src/index.css`**
Create or edit **`src/index.css`**:

```bash
touch src/index.css
```

Paste:

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

/* Load Tailwind */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* Custom styles */
.font-special {
  font-family: "Special Elite", cursive !important;
}
.font-arbutus {
  font-family: "Arbutus Slab", serif !important;
}
.font-cinzel {
  font-family: "Cinzel", serif !important;
}

/* Light mode adjustments */
@media (prefers-color-scheme: light) {
  :root {
    color: #213547;
    background-color: #ffffff;
  }
}
```

### **4b. Create/Update `index.html`**
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
    <!-- Load Google Fonts -->
    <link
      href="https://fonts.googleapis.com/css2?family=Special+Elite&family=Arbutus+Slab&family=Cinzel&display=swap"
      rel="stylesheet"
    />
    <link rel="stylesheet" href="/src/index.css" />
    <title>Steampunk React App</title>
  </head>
  <body class="dark">
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

---

# **5. Update Your App Component**
Edit **`src/App.tsx`**:

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
      </div>
    </>
  );
}

export default App;
```

Run:

```bash
npm run dev
```

Verify the custom fonts are **correctly applied**.

---

# **6. Set Up Storybook**
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

Run:
```bash
npm run storybook
```

---

# **7. Set Up Prettier**
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

Run:
```bash
npm run format
```

---

# **8. Deploy to GitHub Pages**
```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/TortoiseWolfe/steampunk-react-app.git
git push -u origin main
npm run deploy
```

Your site is now at:
```
https://TortoiseWolfe.github.io/steampunk-react-app/
```

**✅ Fonts now display correctly.** Enjoy your fully functional steampunk app! 🚀
