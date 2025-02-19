## 1. Scaffold the Project

1. **Create the Project:**  
   Open Git Bash and run:
   ```bash
   npm create vite@latest steampunk-react-app -- --template react-ts
   ```
   When prompted, choose:  
   - **Project name:** `steampunk-react-app`  
   - **Framework:** React  
   - **Variant:** TypeScript

2. **Navigate and Install:**  
   ```bash
   cd steampunk-react-app
   npm install
   ```

3. **Run the Dev Server:**  
   Start Vite’s development server:
   ```bash
   npm run dev
   ```
   Open the provided URL (typically [http://localhost:5173](http://localhost:5173)) to verify that the default React app is running.

---

## 2. Install Required Dev Dependencies

Install Tailwind CSS (with its Vite plugin), PostCSS, Autoprefixer, Prettier, and gh‑pages in one command:
```bash
npm install --save-dev tailwindcss @tailwindcss/vite postcss autoprefixer prettier gh-pages
```

---

## 3. Configure Vite and Tailwind

### a. Vite Configuration

Create or edit `vite.config.ts` in the project root. For a project site, set the base to your repository name:
```ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  base: '/steampunk-react-app/', // Use '/' if deploying as a user/organization site.
  plugins: [react(), tailwindcss()],
});
```

### b. Tailwind and PostCSS Configurations

Since Tailwind CSS v4 no longer supports the old CLI init command, create these files manually.

**`tailwind.config.js`**
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
        gold: "#D4AF37",
        ivory: "#FFFFF0"
      },
      // Custom font families (three fonts)
      fontFamily: {
        special: ['"Special Elite"', 'cursive'],   // Typewriter-style font
        arbutus: ['"Arbutus Slab"', 'serif'],        // Bold slab serif
        cinzel:  ['Cinzel', 'serif'],                // Classical, steampunk look
      },
    },
  },
  plugins: [],
}
```

**`postcss.config.js`**
```js
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

---

## 4. Set Up Tailwind in Your Project

### a. Create Your Main CSS File

Create or update `src/index.css` with:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### b. Load Custom Fonts in HTML

Edit `index.html` in the project root. In the `<head>` section, add:
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
    <link rel="stylesheet" href="/src/index.css" />
    <title>Steampunk React App</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

### c. Test Your Tailwind Setup

Edit `src/App.tsx` to use your new Tailwind classes and custom fonts:
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
Save and view the app in your browser. You should see the copper background, custom colors, and the three fonts applied.

---

## 5. Set Up Storybook (with Vite Builder)

### a. Initialize Storybook

In the project root, run:
```bash
npx storybook@latest init --builder=vite
```
This installs Storybook, creates a `.storybook` folder with configuration files, and adds sample stories. It also updates your `package.json` with Storybook scripts.

### b. Configure Storybook to Use Tailwind

Edit `.storybook/preview.js` (or `preview.ts`) to import your Tailwind CSS:
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

### c. Run Storybook

Start Storybook by running:
```bash
npm run storybook
```
Your browser should open at [http://localhost:6006](http://localhost:6006) displaying sample stories with your Tailwind styles applied.

---

## 6. Set Up Prettier for Code Formatting

### a. Create Prettier Config Files

Create a file named **`.prettierrc`** in the project root:
```json
{
  "singleQuote": true,
  "trailingComma": "es5",
  "printWidth": 80,
  "bracketSpacing": true
}
```

Create a file named **`.prettierignore`** with:
```
node_modules
dist
```

### b. Add a Format Script

In your `package.json`, add:
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

## 7. Configure GitHub Pages Deployment

### a. Prepare Your Repository

Initialize Git and push to GitHub using your actual account:
```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/TortoiseWolfe/steampunk-react-app.git
git push -u origin main
```

### b. Set Up Deployment Scripts

In your `package.json`, add or update the following scripts:
```json
"scripts": {
  "dev": "vite",
  "build": "vite build",
  "preview": "vite preview",
  "predeploy": "npm run build",
  "deploy": "gh-pages -d dist"
}
```

### c. Verify Vite’s Base URL

Ensure your `vite.config.ts` (from Step 3a) has the base set to:
```ts
base: '/steampunk-react-app/'
```
This is correct for a project site.

### d. Deploy the App

Build and deploy your site by running:
```bash
npm run deploy
```
This command builds the app and pushes the contents of the `dist` folder to the `gh-pages` branch.  
Then, go to your GitHub repository’s **Settings → Pages** and set the source to the `gh-pages` branch. Your site will be live at:
- **https://TortoiseWolfe.github.io/steampunk-react-app/**

---

## Final Summary

You now have a fully working project with:
- **Vite + React + TypeScript:** Scaffolded with Vite.
- **Tailwind CSS via the Vite Plugin:** Configured with a steampunk theme (colors: copper, bronze, gold, ivory) and three custom fonts (Special Elite, Arbutus Slab, and Cinzel).
- **Storybook:** Set up using Vite as the builder and importing Tailwind styles.
- **Prettier:** Configured for consistent code formatting.
- **GitHub Pages:** Deployment scripts set so your site publishes to **https://TortoiseWolfe.github.io/steampunk-react-app/**.

Follow these instructions exactly, and your project will be ready for development and deployment. Happy coding!
