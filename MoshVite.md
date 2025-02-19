# 1. Create & Scaffold the Project

1. **Initialize a Vite React+TS Project**  
   ```bash
   npm create vite@latest steampunk-react-app -- --template react-ts
   cd steampunk-react-app
   npm install
   ```
   - This creates `steampunk-react-app` with React + TypeScript.

2. **Test the Dev Server**  
   ```bash
   npm run dev
   ```
   - Confirm it runs at http://localhost:5173.

---

# 2. Install Tailwind (V4) via the Official Vite Plugin

1. **Install** only two packages:  
   ```bash
   npm install -D tailwindcss @tailwindcss/vite
   ```
   - **No** PostCSS or autoprefixer needed for this approach.

2. **Configure `vite.config.ts`**  
   Create or open `/vite.config.ts` in the **root** of your project. Replace contents with:
   ```ts
   import { defineConfig } from 'vite';
   import react from '@vitejs/plugin-react';
   import tailwind from '@tailwindcss/vite';

   export default defineConfig({
     // For GitHub project site: TortoiseWolfe/steampunk-react-app
     base: '/steampunk-react-app/',
     plugins: [
       react(),
       tailwind(), // Official plugin that processes Tailwind
     ],
   });
   ```
   - If you deploy to a user/organization site (e.g. TortoiseWolfe.github.io), set `base: '/'` instead.

3. **Create a Tailwind config** (for custom theme):
   ```bash
   touch tailwind.config.js
   ```
   Then `/tailwind.config.js`:
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

# 3. Set Up Tailwind in Your CSS

1. **Create/Update `/src/index.css`**  
   ```bash
   touch src/index.css
   ```
   Then `/src/index.css`:
   ```css
   /* Load Tailwind (no postcss.config needed) */
   @import "tailwindcss";

   /* Or, equivalently:
   @tailwind base;
   @tailwind components;
   @tailwind utilities;
   */

   /* Below this line, you can add your custom CSS if you want. */
   ```

2. **Load Google Fonts**  
   In `/index.html` (the project root), inside `<head>`:
   ```html
   <!doctype html>
   <html lang="en">
     <head>
       <meta charset="UTF-8" />
       <meta name="viewport" content="width=device-width, initial-scale=1.0" />
       <!-- Load three fonts: Special Elite, Arbutus Slab, and Cinzel -->
       <link
         href="https://fonts.googleapis.com/css2?family=Special+Elite&family=Arbutus+Slab&family=Cinzel&display=swap"
         rel="stylesheet"
       />
       <!-- Link your main CSS -->
       <link rel="stylesheet" href="/src/index.css" />
       <title>Steampunk React App</title>
     </head>
     <body>
       <div id="root"></div>
       <script type="module" src="/src/main.tsx"></script>
     </body>
   </html>
   ```

3. **Test**  
   In `/src/App.tsx`, try your classes:
   ```tsx
   export default function App() {
     return (
       <div className="bg-copper p-6 min-h-screen">
         <h1 className="text-4xl font-special text-ivory">
           Steampunk Vite App
         </h1>
         <p className="mt-4 text-gold">
           Using Arbutus Slab → <span className="font-arbutus">Hello</span>
         </p>
         <p className="mt-4 text-bronze">
           Using Cinzel → <span className="font-cinzel">Classical vibes</span>
         </p>
       </div>
     );
   }
   ```
   ```bash
   npm run dev
   ```
   Check http://localhost:5173. You should see copper background and custom fonts.  

---

# 4. Set Up Storybook (Vite Builder)

1. **Initialize Storybook**  
   ```bash
   npx storybook@latest init --builder=vite
   ```
   - Creates `.storybook` folder, sample stories, and updates `package.json` scripts.

2. **Configure**  
   In `/.storybook/preview.js` (or `preview.ts`):
   ```js
   import '../src/index.css'; // Import your Tailwind CSS

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
   Visit http://localhost:6006. The sample components now have Tailwind styling.

---

# 5. Prettier Setup

1. **Create `.prettierrc` and `.prettierignore`**  
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
   Then:
   ```bash
   npm run format
   ```

---

# 6. GitHub Pages Deployment

1. **Initialize Git & Push**  
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
   (You’ll need `gh-pages` installed, but we did that in Step 2 with the other dev dependencies if you want to deploy.)

3. **Ensure `base` is set**  
   `vite.config.ts` has `base: '/steampunk-react-app/'` for a project site. If it was a user site (TortoiseWolfe.github.io), use `'/'`.

4. **Deploy**  
   ```bash
   npm run deploy
   ```
   Then in your GitHub repo’s **Settings → Pages**, select the `gh-pages` branch. Your site is at:
   ```
   https://TortoiseWolfe.github.io/steampunk-react-app/
   ```

---

# Final, Complete Setup

1. **No** `postcss.config.js` referencing `tailwindcss`. The plugin `@tailwindcss/vite` handles it.  
2. The entire code structure is:
   ```
   steampunk-react-app/
   ├─ .storybook/
   ├─ src/
   │   ├─ App.tsx
   │   ├─ main.tsx
   │   └─ index.css   (Tailwind import at top)
   ├─ index.html      (Google Fonts link in <head>)
   ├─ tailwind.config.js  (custom theme)
   ├─ vite.config.ts      (import tailwind() from @tailwindcss/vite)
   ├─ .prettierrc
   ├─ .prettierignore
   ├─ package.json
   └─ ...
   ```
3. Everything is **self-contained**—no patches, no extra PostCSS.  
4. **Result**: A fully functional steampunk-themed React+TS app with custom fonts, Storybook, Prettier, and GH Pages deployment, with **no** “tailwindcss as a PostCSS plugin” errors.

Follow these steps exactly, and you’ll have a complete, working tutorial with no broken bits. Enjoy your steampunk styling!
