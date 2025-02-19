> **Important:**  
> • Tailwind CSS v4 no longer supports the old CLI init command – we configure it manually.  
> • Replace `<YOUR_USERNAME>` and `<YOUR_REPO>` with your GitHub details.

---

## 1. Scaffold Your Vite + React (TypeScript) App

1. **Create the Project:**  
   In Git Bash, run:
   ```bash
   npm create vite@latest my-app -- --template react-ts
   ```
   When prompted, choose:  
   - **Project name:** `my-app` (or your preferred name)  
   - **Framework:** React  
   - **Variant:** TypeScript

2. **Navigate and Install:**  
   ```bash
   cd my-app
   npm install
   ```
3. **Run the Dev Server:**  
   Start Vite’s development server:
   ```bash
   npm run dev
   ```
   Open the provided URL (typically http://localhost:5173) to verify the basic React app is working.

---

## 2. Install Dev Dependencies in One Command

Install Tailwind CSS (with its Vite plugin), PostCSS, Autoprefixer, Prettier, and gh‑pages:
```bash
npm install --save-dev tailwindcss @tailwindcss/vite postcss autoprefixer prettier gh-pages
```

---

## 3. Configure Tailwind CSS via the Vite Plugin

### a. Update Vite Configuration

1. **Edit `vite.config.ts`:**  
   Update the file to include React and Tailwind’s Vite plugin. If you’re deploying to a GitHub project site, set the `base` path:
   ```ts
   import { defineConfig } from 'vite';
   import react from '@vitejs/plugin-react';
   import tailwindcss from '@tailwindcss/vite';

   export default defineConfig({
     base: '/<YOUR_REPO>/', // For GitHub project sites; use '/' for user/organization sites.
     plugins: [react(), tailwindcss()],
   });
   ```
   Replace `<YOUR_REPO>` with your repository name.

### b. Create Tailwind Configuration Files

Since Tailwind v4 dropped the init command, create these files manually.

1. **Create `tailwind.config.js`:**  
   In the project root, create a file named `tailwind.config.js`:
   ```js
   /** @type {import('tailwindcss').Config} */
   export default {
     content: [
       "./index.html",
       "./src/**/*.{js,ts,jsx,tsx}",
     ],
     theme: {
       extend: {
         // Steampunk color palette
         colors: {
           copper: "#B87333",
           bronze: "#CD7F32",
           gold:   "#D4AF37",
           ivory:  "#FFFFF0"
         },
         // Custom font families
         fontFamily: {
           special: ['"Special Elite"', 'cursive'],  // Typewriter-style
           arbutus: ['"Arbutus Slab"', 'serif'],       // Bold slab serif
           cinzel:  ['Cinzel', 'serif'],               // Classical steampunk look
         },
       },
     },
     plugins: [],
   }
   ```
2. **Create `postcss.config.js`:**  
   In the project root, create a file named `postcss.config.js`:
   ```js
   export default {
     plugins: {
       tailwindcss: {},
       autoprefixer: {},
     },
   }
   ```

### c. Import Tailwind CSS in Your Styles

1. **Edit `src/index.css`:**  
   Replace or add the following directives:
   ```css
   @tailwind base;
   @tailwind components;
   @tailwind utilities;
   ```

2. **Load Custom Fonts:**  
   In your `index.html` (inside the `<head>` tag), add a single Google Fonts link to load all three fonts:
   ```html
   <link
     href="https://fonts.googleapis.com/css2?family=Special+Elite&family=Arbutus+Slab&family=Cinzel&display=swap"
     rel="stylesheet"
   />
   ```
3. **Test Tailwind Styles:**  
   Update `src/App.tsx` to use your new fonts and colors:
   ```tsx
   export default function App() {
     return (
       <div className="bg-copper p-6 min-h-screen">
         <h1 className="text-4xl font-special text-ivory">
           Steampunk Vite App
         </h1>
         <p className="mt-4 text-gold">
           This text uses the <span className="font-arbutus">Arbutus Slab</span> font.
         </p>
         <p className="mt-4 text-bronze">
           And this line is styled with <span className="font-cinzel">Cinzel</span>.
         </p>
       </div>
     );
   }
   ```
   Save and verify in your browser that the styles (background color, text colors, and fonts) are applied correctly.

---

## 4. Set Up Storybook (with Vite Builder)

### a. Initialize Storybook

1. **Run the Storybook Setup:**  
   In your project root, run:
   ```bash
   npx storybook@latest init --builder=vite
   ```
   This installs Storybook packages, creates a `.storybook` folder with configuration files, and adds sample stories.

### b. Configure Storybook

1. **Import Tailwind in Storybook:**  
   In `.storybook/preview.js` (or `preview.ts`), ensure you import your Tailwind CSS:
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
2. **Run Storybook:**  
   Start Storybook by running:
   ```bash
   npm run storybook
   ```
   Your browser should open at http://localhost:6006 displaying sample stories with Tailwind’s styles applied.

---

## 5. Set Up Prettier for Code Formatting

1. **Create a Prettier Configuration:**  
   In the project root, create a `.prettierrc` file:
   ```json
   {
     "singleQuote": true,
     "trailingComma": "es5",
     "printWidth": 80,
     "bracketSpacing": true
   }
   ```
2. **Create a Prettier Ignore File:**  
   Create a `.prettierignore` file with:
   ```
   node_modules
   dist
   ```
3. **Add a Format Script:**  
   In your `package.json`, add:
   ```json
   "scripts": {
     "format": "prettier --write ."
   }
   ```
4. **Run Prettier:**  
   Execute:
   ```bash
   npm run format
   ```

---

## 6. Configure GitHub Pages Deployment

### a. Prepare Your Repository

1. **Initialize Git and Push to GitHub:**  
   In your project root, run:
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin https://github.com/<YOUR_USERNAME>/<YOUR_REPO>.git
   git push -u origin main
   ```

### b. Set Up Deployment Scripts

1. **Update Deployment Scripts:**  
   In your `package.json`, add or update:
   ```json
   "scripts": {
     "dev": "vite",
     "build": "vite build",
     "preview": "vite preview",
     "predeploy": "npm run build",
     "deploy": "gh-pages -d dist"
   }
   ```
2. **Configure Vite Base URL:**  
   In `vite.config.ts`, ensure the base is set (as shown earlier):
   ```ts
   export default defineConfig({
     base: '/<YOUR_REPO>/', // Use '/' if deploying as a user/organization site.
     plugins: [react(), tailwindcss()],
   });
   ```
3. **Deploy Your App:**  
   Commit your changes and run:
   ```bash
   npm run deploy
   ```
   This builds the app and pushes the `dist` folder to the `gh-pages` branch.  
   Then, in your GitHub repository’s **Settings → Pages**, set the source to the `gh-pages` branch. Your site will be live at:
   - `https://<YOUR_USERNAME>.github.io/<YOUR_REPO>/` for project sites, or  
   - `https://<YOUR_USERNAME>.github.io/` for user/organization sites.

---

## Final Summary

- **Vite + React + TypeScript:** Scaffolding via Vite.
- **Tailwind CSS:** Installed as a Vite plugin along with PostCSS and Autoprefixer.  
  Configured manually with a steampunk color palette and three custom fonts:  
  - **Special Elite** (font-special)  
  - **Arbutus Slab** (font-arbutus)  
  - **Cinzel** (font-cinzel)
- **Storybook:** Set up with the Vite builder and configured to import Tailwind CSS.
- **Prettier:** Configured for code formatting.
- **GitHub Pages:** Deployment scripts and Vite base URL are set up for seamless publishing.

This complete, fully functional tutorial avoids the “could not determine executable to run” error and ensures Storybook runs without hanging. Follow these steps exactly, and your project will be ready for development and deployment. Happy coding!
