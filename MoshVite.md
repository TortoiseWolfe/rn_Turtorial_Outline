## 1. Create & Scaffold Your Project

1. **Initialize a Vite React+TS Project**  
   ```bash
   npm create vite@latest steampunk-react-app -- --template react-ts
   cd steampunk-react-app
   npm install
   ```

2. **Test the Dev Server**  
   ```bash
   npm run dev
   ```
   Open the provided URL (e.g., http://localhost:5173) to confirm it works.

---

## 2. Install Tailwind (V4) via the Official Vite Plugin

1. **Install the Vite Plugin**  
   ```bash
   npm install -D tailwindcss @tailwindcss/vite
   ```
   > **No** `postcss.config.js` is required for this approach. If you have one referencing `tailwindcss`, remove or rename it so it’s not used.

2. **Configure `vite.config.ts`**  
   Create or edit `vite.config.ts` in the **root** of your project:
   ```ts
   import { defineConfig } from 'vite';
   import react from '@vitejs/plugin-react';
   import tailwind from '@tailwindcss/vite';

   export default defineConfig({
     // If deploying to a project site (e.g. TortoiseWolfe/steampunk-react-app):
     base: '/steampunk-react-app/',
     plugins: [
       react(),
       // The official Tailwind plugin for Vite (Approach A):
       tailwind(),
     ],
   });
   ```

3. **Create a Tailwind Config (Optional, but needed for custom theme)**  
   ```bash
   touch tailwind.config.js
   ```
   Then add your steampunk theme:

   ```js
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

## 3. Set Up Tailwind in Your CSS

1. **Import Tailwind** in your main stylesheet (e.g. `src/index.css`):
   ```css
   /* Option A: Use Tailwind's single import */
   @import "tailwindcss";

   /* Option B: Use the classic @tailwind directives
      @tailwind base;
      @tailwind components;
      @tailwind utilities;
   */

   /* Then your custom CSS can follow. If you have existing styles, place them after. */
   ```

2. **(Optional) Load Fonts & Add Custom Styles**  
   - In `index.html`, inside `<head>`:
     ```html
     <link
       href="https://fonts.googleapis.com/css2?family=Special+Elite&family=Arbutus+Slab&family=Cinzel&display=swap"
       rel="stylesheet"
     />
     ```
   - If you have existing custom styles (like a dark theme, resets, etc.), just keep them in `index.css` **below** the Tailwind import so Tailwind’s base resets load first.

3. **Test**  
   In `src/App.tsx`, try your custom classes:
   ```tsx
   export default function App() {
     return (
       <div className="bg-copper p-6 min-h-screen">
         <h1 className="text-4xl font-special text-ivory">Steampunk Vite App</h1>
         <p className="mt-4 text-gold">Using Arbutus Slab → <span className="font-arbutus">Hello</span></p>
         <p className="mt-4 text-bronze">Using Cinzel → <span className="font-cinzel">Classical vibes</span></p>
       </div>
     );
   }
   ```
   Run `npm run dev`—Tailwind’s classes and your custom theme should work without PostCSS plugin errors.

---

## 4. Set Up Storybook (with Vite Builder)

1. **Initialize Storybook**  
   ```bash
   npx storybook@latest init --builder=vite
   ```
   This creates a `.storybook` folder with config and sample stories.

2. **Configure Storybook to Use Tailwind**  
   - In `.storybook/preview.js` (or `.storybook/preview.ts`):
     ```js
     import '../src/index.css';

     export const parameters = {
       actions: { argTypesRegex: "^on[A-Z].*" },
       controls: {
         matchers: {
           color: /(background|color)$/i,
           date: /Date$/,
         },
       },
     };
     ```
3. **Run Storybook**  
   ```bash
   npm run storybook
   ```
   Visit http://localhost:6006—your components should have Tailwind styles.

---

## 5. Prettier Setup

1. **Create Prettier Config**  
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

2. **Add Format Script**  
   In `package.json`:
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

## 6. GitHub Pages Deployment

1. **Initialize Git**  
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin https://github.com/TortoiseWolfe/steampunk-react-app.git
   git push -u origin main
   ```

2. **Add Deployment Scripts**  
   In `package.json`:
   ```json
   "scripts": {
     "dev": "vite",
     "build": "vite build",
     "preview": "vite preview",
     "predeploy": "npm run build",
     "deploy": "gh-pages -d dist"
   }
   ```
3. **Set `base` in Vite Config**  
   If you’re deploying to `TortoiseWolfe/steampunk-react-app`, your `vite.config.ts` base is:
   ```ts
   base: '/steampunk-react-app/'
   ```
   (Use `'/'` if it’s a user/organization site, e.g. TortoiseWolfe.github.io.)

4. **Deploy**  
   ```bash
   npm run deploy
   ```
   Then in your GitHub repo’s **Settings → Pages**, select the `gh-pages` branch as the source. Your site will be at:
   ```
   https://TortoiseWolfe.github.io/steampunk-react-app/
   ```

---

## Final Recap

**Approach A** with **`@tailwindcss/vite`** means:

- **No** `postcss.config.js` referencing `tailwindcss`.
- `vite.config.ts` uses `tailwind()` from **`@tailwindcss/vite`**.
- You can still create a `tailwind.config.js` for custom colors, fonts, etc.
- In your CSS, simply `@import "tailwindcss";` or use `@tailwind base; @tailwind components; @tailwind utilities;`.
- This resolves the “using tailwindcss as a PostCSS plugin” error in Tailwind v4.

Following these steps, you’ll have a fully functional **Vite + React + TypeScript** project with a **steampunk** Tailwind theme, **Storybook** (Vite builder), **Prettier**, and **GitHub Pages** deployment, all **without** postcss errors. Enjoy your updated setup!
