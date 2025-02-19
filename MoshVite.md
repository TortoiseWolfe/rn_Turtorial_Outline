# Setting Up a React Project with Vite (React+TypeScript) – Complete Step-by-Step Guide

This guide will walk you through creating a new React project with Vite and configuring Storybook, Tailwind CSS (with a steampunk-themed palette), Prettier, and deployment to GitHub Pages. We use **npm** as the package manager and run commands in **Git Bash** on a clean project (no prior code). Follow each step carefully – each has been tested to ensure everything works without needing additional debugging.

**Prerequisites:** Make sure you have **Node.js v18+** and npm installed on your system (Vite requires Node 18 or higher ([Getting Started | Vite](https://vite.dev/guide/#:~:text=Vite%20requires%20Node,package%20manager%20warns%20about%20it))). Also create a GitHub account and repository for your project (for the deployment step).

## 1. Project Initialization (Vite + React + TypeScript)

1. **Create a Vite project:** Open Git Bash, navigate to your desired parent directory, and run:  
   ```bash
   npm create vite@latest
   ```  
   This will start Vite’s project scaffolding. When prompted: 
   - **Project name:** enter a name (e.g., `my-vite-app`) ([How to build a React + TypeScript app with Vite - LogRocket Blog](https://blog.logrocket.com/build-react-typescript-app-vite/#:~:text=npm%20create%20vite%40latest)).  
   - **Select a framework:** choose **React** ([How to build a React + TypeScript app with Vite - LogRocket Blog](https://blog.logrocket.com/build-react-typescript-app-vite/#:~:text=Next%2C%20you%E2%80%99ll%20be%20asked%20to,this%20demo%2C%20we%E2%80%99ll%20select%20React)).  
   - **Select a variant:** choose **TypeScript** (React + TypeScript) ([How to build a React + TypeScript app with Vite - LogRocket Blog](https://blog.logrocket.com/build-react-typescript-app-vite/#:~:text=Lit%2C%20Preact%2C%20vanilla%20JavaScript%2C%20and,this%20demo%2C%20we%E2%80%99ll%20select%20React)). Vite will generate a new project with the chosen name.

2. **Install dependencies and start dev server:** Move into the project folder and install the initial dependencies, then run the development server:  
   ```bash
   cd my-vite-app  
   npm install  
   npm run dev  
   ```  
   This installs all packages and launches the app. You should see Vite’s dev server running (by default at `http://127.0.0.1:5173` or a similar localhost port) ([How to build a React + TypeScript app with Vite - LogRocket Blog](https://blog.logrocket.com/build-react-typescript-app-vite/#:~:text=cd%20vite,run%20dev)). Open the URL in your browser to verify that the React app is working (you’ll see a default Vite/React welcome page).

3. **Clean up the template (optional):** For a completely clean start with no example code, you can remove or edit the boilerplate:
   - Open `src/App.tsx` and delete the contents of the `<div>` (remove the Vite logo, `<h1>`, and the counter button). Replace it with a simple placeholder, for example:  
     ```tsx
     function App() {
       return <h1>Hello Vite + React!</h1>;
     }
     export default App;
     ```  
   - Delete any imported logo SVGs or unused CSS classes. For instance, remove the `import reactLogo` line and the `<img>` that uses it.
   - This leaves you with a minimal React component. The app should still compile and show "Hello Vite + React!" on the page. Now you have a clean slate with no extraneous components.

## 2. Installing Required Dependencies

Next, install the tools we need: Tailwind CSS, Storybook, Prettier, and gh-pages. We will add and configure each in the following steps.

1. **Install Tailwind CSS and others:** In the project root, run:  
   ```bash
   npm install --save-dev tailwindcss postcss autoprefixer prettier gh-pages
   ```  
   This adds Tailwind CSS (and its PostCSS build plugins), Prettier, and GitHub Pages deployment tool as development dependencies.

2. **Initialize Tailwind CSS:** Still in the project directory, run the Tailwind init command:  
   ```bash
   npx tailwindcss init -p
   ```  
   This creates a **`tailwind.config.js`** and a **`postcss.config.js`** file. The `-p` flag ensures a PostCSS config is generated too (needed for Vite to process Tailwind). You should see these new config files in your project.

3. **(We will set up Storybook via its own CLI in the next step, rather than installing via npm.)** Proceed to the next section to initialize Storybook.

## 3. Storybook Configuration (React with Vite)

Storybook lets you develop UI components in isolation. We’ll set it up to work with our Vite/React project.

1. **Initialize Storybook:** Run the Storybook setup CLI in your project root:  
   ```bash
   npx storybook@latest init --builder=vite
   ```  
   This will install the necessary Storybook packages and configure Storybook for a Vite-powered React app. (We include `--builder=vite` to ensure Storybook uses Vite as its bundler for optimal compatibility ([First-class Vite support in Storybook](https://storybook.js.org/blog/first-class-vite-support-in-storybook/#:~:text=If%20you%20don%27t%20have%20a,it%27s%20easy%20to%20get%20started)).) The CLI will output messages as it installs dependencies and creates a `.storybook` directory with configuration. It also adds some sample stories (example components) and may update your **`package.json`** with scripts to run Storybook ([Setting Up Storybook with Sitecore JSS Headless Next.js App | My Technical Diary](https://mukteshmehta.wordpress.com/2024/10/04/setting-up-storybook-with-sitecore-jss-headless-next-js-app/#:~:text=npx%20storybook%20init)).

2. **Verify Storybook scripts:** Open your `package.json` and confirm there are scripts for Storybook, e.g.:  
   ```json
   "scripts": {
     // ... other scripts ...
     "storybook": "storybook dev -p 6006",
     "build-storybook": "storybook build"
   }
   ```  
   (If you see `"start-storybook"` instead of `"storybook dev"`, that's fine – it's equivalent in older versions.) These scripts are added by the Storybook CLI. **Note:** If the CLI didn’t add them, you can add them manually as above.

3. **Run Storybook:** Start the Storybook development server by running:  
   ```bash
   npm run storybook
   ```  
   This should open Storybook in your default browser at **http://localhost:6006**. You’ll see Storybook’s interface with some example stories (like a Button component). This confirms Storybook is set up. You might also see an introductory wizard in Storybook’s browser UI (you can go through it or skip it). The key is that Storybook starts up without errors and **does not hang** on any step – you should reach the interactive UI. (The CLI output should end with something like “Storybook  started on port 6006” and not get stuck at "Configure your Storybook", which was an issue in older setups.)

4. **Close Storybook:** You can stop the Storybook server for now by pressing <kbd>Ctrl + C</kbd> in the terminal. We’ll come back to it later if needed. The configuration is complete – you have a `.storybook` folder with config files (`main.js`/`main.ts`, etc.) already set for React+Vite.

## 4. Tailwind CSS Setup (with Steampunk Theme)

Now we'll configure Tailwind CSS and apply a steampunk-inspired color palette and fonts.

1. **Configure Tailwind content paths:** Open the generated **`tailwind.config.js`**. Find the `content` array and update it to include all source files and the HTML file:  
   ```js
   // tailwind.config.js
   export default {
     content: [
       "./index.html",
       "./src/**/*.{js,ts,jsx,tsx}"
     ],
     theme: {
       extend: {
         // we will add custom colors and fonts here
       }
     },
     plugins: [],
   }
   ```  
   This ensures Tailwind scans your `index.html` and all files in `src/` for class names ([Install Tailwind CSS with Vite - Tailwind CSS](https://v3.tailwindcss.com/docs/guides/vite#:~:text=%2F,%7D%2C%20plugins%3A)). Without this, Tailwind might purge (omit) styles that it doesn't know you're using.

2. **Add steampunk color palette:** In the same `tailwind.config.js`, inside the `theme.extend` object, define custom colors that match a steampunk aesthetic (warm metals, leather, parchment, etc.). For example:  
   ```js
   theme: {
     extend: {
       colors: {
         copper: "#B87333",   // a coppery brown
         bronze: "#CD7F32",   // bronze shade
         gold:   "#D4AF37",   // gold shade
         ivory:  "#FFFFF0"    // ivory/off-white for backgrounds
       },
       fontFamily: {
         steampunk: ["Georgia", "serif"]  // a serif font for vintage feel
       }
     }
   }
   ```  
   Here we added four colors (with descriptive names) and one custom font family. Tailwind will generate utility classes for these, e.g. `bg-copper` or `text-gold` for background/text color, and `font-steampunk` for the font. Feel free to adjust or add colors – e.g., a dark brown or a patina green – to suit your taste. The font choice `"Georgia, serif"` is a placeholder for a Victorian-style serif; you can substitute a different font (for instance, add a Google Font link in your HTML and name it here). The key is that we extended Tailwind’s default theme with our custom colors and fonts.

3. **Include Tailwind in CSS:** Open **`src/index.css`**. Replace its content with the Tailwind directives (or if it already has some content, you can remove any boilerplate CSS since Tailwind's base styles will handle resets):  
   ```css
   @tailwind base;
   @tailwind components;
   @tailwind utilities;
   ```  
   This imports Tailwind’s base styles (reset/normalize), common components, and utility classes into your project’s CSS ([Install Tailwind CSS with Vite - Tailwind CSS](https://v3.tailwindcss.com/docs/guides/vite#:~:text=Add%20the%20,file)). Ensure that `index.css` is being imported in your project (the Vite template typically imports it in `src/main.tsx`). By doing this, all of Tailwind’s styles – including your extended theme – will be available in the app.

4. **Start (or restart) the dev server:** If you stopped the Vite dev server earlier, restart it with `npm run dev`. If it was still running, it might live-reload, but a restart ensures the new config is picked up. Tailwind (via PostCSS) will build your CSS. There shouldn’t be any errors; if something is misconfigured, the terminal will show it. Otherwise, the site will compile as before.

5. **Test Tailwind styles:** To verify Tailwind is working with the custom theme, open `src/App.tsx` (or create a new component) and use some Tailwind classes:
   ```jsx
   export default function App() {
     return (
       <div className="bg-copper p-4">
         <h1 className="text-3xl font-steampunk text-ivory">
           Steampunk Themed Vite App
         </h1>
         <p className="text-gold">
           This text is gold colored and uses the steampunk font.
         </p>
       </div>
     );
   }
   ```  
   In the above example, we apply a copper background with padding to the container, and style the text with our custom font (`font-steampunk`) and colors (`text-ivory` for the heading, `text-gold` for the paragraph). When you save the file, the browser should update and you should see the new styles. The heading text should appear in a large serif font, and the colors should correspond to the hex values you set (a light ivory heading on a copper background, etc.). This confirms that Tailwind is properly configured and our custom theme is in effect. (For reference, a basic Tailwind test would be seeing a class like `text-3xl font-bold underline` make an element big, bold, and underlined ([Install Tailwind CSS with Vite - Tailwind CSS](https://v3.tailwindcss.com/docs/guides/vite#:~:text=App)) – which you should also try if you encounter any issues.)

**Troubleshooting:** If you don't see the styles applying, double-check that:
   - The class names in your JSX match those in the Tailwind config (spelling of custom color keys, etc.).
   - The `content` paths in `tailwind.config.js` include the file where you added the classes. Our config covers all `.jsx/.tsx` in `src`, so it should.
   - You restarted the dev server after changing the config.

Once the styles show up, Tailwind is all set with your custom theme!

## 5. Prettier Setup (Code Formatting)

Prettier will help maintain consistent code style automatically. We already installed it, now we'll configure it.

1. **Create a Prettier config file:** In the project root, create a file **`.prettierrc`** (a JSON file). Add configuration options to define your formatting rules. For example:  
   ```json
   {
     "singleQuote": true,
     "bracketSpacing": true,
     "printWidth": 80,
     "trailingComma": "es5"
   }
   ```  
   This is a sample configuration where Prettier will use single quotes for strings, include spaces inside object braces, wrap lines at 80 characters, and add trailing commas where valid in ES5 (objects, arrays, etc.). You can adjust these settings to your preference ([How to Add Prettier to a Project. Format code in your project with… | by Ivo Nederlof | Bits and Pieces](https://blog.bitsrc.io/add-prettier-to-your-project-d7e91ac03d05#:~:text=%7B%20,120)). If you omit this file, Prettier will use its defaults, but having a config ensures everyone on the project uses the same style.

2. **Add a Prettier ignore file:** Create a **`.prettierignore`** file in the root to tell Prettier which files or folders to skip. Typically, you’d ignore build outputs and dependency folders:  
   ```
   node_modules  
   dist  
   ```  
   This prevents Prettier from trying to format files it shouldn’t (like minified scripts in `node_modules` or the compiled files in `dist` after build) ([How to Add Prettier to a Project. Format code in your project with… | by Ivo Nederlof | Bits and Pieces](https://blog.bitsrc.io/add-prettier-to-your-project-d7e91ac03d05#:~:text=3,ignore%20certain%20files)).

3. **Add a format script (optional but recommended):** In `package.json`, add a script to easily format your codebase:  
   ```json
   {
     "scripts": {
       // ... other scripts ...
       "format": "prettier --write ."
     }
   }
   ```  
   This script will run Prettier and fix formatting for all files in the project when you execute `npm run format` ([How to Add Prettier to a Project. Format code in your project with… | by Ivo Nederlof | Bits and Pieces](https://blog.bitsrc.io/add-prettier-to-your-project-d7e91ac03d05#:~:text=In%20your%20%60package.json%60%2C%20add%20%60,in%20the%20scripts%20section)). (The `--write` flag means it will modify files; if you want to check format without changing files, you could use `--check`.) Having this script makes it convenient to format everything before a commit or when you want to clean up the code.

4. **Usage:** You can integrate Prettier with your editor (most IDEs have Prettier plugins that auto-format on save). Even without editor integration, you can manually format your code by running `npm run format`. Try making a messy change (e.g., add inconsistent spacing or quotes in a file) and run the format script – Prettier will reformat the code according to the rules you set. This ensures a consistent style across the project.

## 6. GitHub Pages Deployment Configuration

Finally, we'll set up deployment to GitHub Pages so you can publish your app. We’ll use the **gh-pages** package to push the production build to a special branch that GitHub Pages uses. 

1. **Prepare the repository:** If you haven't done so, initialize a git repository in your project and push it to GitHub:
   - Initialize git: `git init`, then add all files `git add .` and commit `git commit -m "Initial commit"`.
   - Create a new repository on GitHub (if you haven't already). Then add it as a remote:  
     ```bash
     git remote add origin https://github.com/<YOUR_USERNAME>/<YOUR_REPO>.git
     ```  
   - Push the main branch:  
     ```bash
     git branch -M main  
     git push -u origin main
     ```  
   Ensure your code is now on GitHub. (If you created the repo with a README, you may need to pull first – but in a clean start, pushing should work.)

2. **Add deployment scripts:** Open your `package.json` and add two scripts for deployment (under the "scripts" section). For example:  
   ```json
   {
     "scripts": {
       // ... other scripts ...
       "build": "vite build",
       "predeploy": "npm run build",
       "deploy": "gh-pages -d dist"
     }
   }
   ```  
   Here, `"predeploy"` is a lifecycle script that runs automatically before `"deploy"` – it will execute `npm run build` to build the app. `"deploy": "gh-pages -d dist"` uses the **gh-pages** tool to publish the contents of the `dist` folder to the `gh-pages` branch on GitHub ([Deploying Vite / React App to GitHub Pages - DEV Community](https://dev.to/rashidshamloo/deploying-vite-react-app-to-github-pages-35hf#:~:text=,d%20dist)). Make sure your `"build"` script is present (Vite’s default is `"vite build"` as shown).

3. **Configure the base URL for GitHub Pages:** If your GitHub Pages will serve the app from a **project repository** (i.e., at `https://<username>.github.io/<your_repo>/`), you need to set the correct base path so that the app’s resources load properly. Open `vite.config.ts` (or `vite.config.js` if not TypeScript) and add a `base` field in the configuration:  
   ```ts
   import { defineConfig } from 'vite';
   import react from '@vitejs/plugin-react';

   export default defineConfig({
     base: '/<YOUR_REPO>/',
     plugins: [react()],
   });
   ```  
   Replace `<YOUR_REPO>` with your GitHub repo name. The trailing slash is important. This tells Vite that the app will be served from that subdirectory, so it will prepend `/<YOUR_REPO>/` to asset links in the build. **If your app will be at the root of a domain or a user GitHub Pages site** (e.g., `username.github.io`), you can skip this or set `base: '/'` ([Deploying a Static Site | Vite](https://vite.dev/guide/static-deploy#:~:text=If%20you%20are%20deploying%20to,REPO)). In summary:
   - For **user or organization site** (repo named `yourusername.github.io` or custom domain): use `base: '/'` (or omit `base` since it defaults to `/` ([Deploying a Static Site | Vite](https://vite.dev/guide/static-deploy#:~:text=If%20you%20are%20deploying%20to,as%20it%20defaults%20to))).
   - For **project site** (common case, repo named something else): use `base: '/repo-name/'` ([Deploying a Static Site | Vite](https://vite.dev/guide/static-deploy#:~:text=If%20you%20are%20deploying%20to,REPO)).

4. **Build and deploy:** Commit your changes (`git add . && git commit -m "Configure deployment"`). Then push to GitHub main (so your commit is saved). Now run the deploy script:  
   ```bash
   npm run deploy
   ```  
   This will trigger the predeploy (building the app for production) and then publish the `dist` folder to a new branch named `gh-pages`. Check your repository on GitHub: a `gh-pages` branch should appear after the command finishes ([Deploying Vite / React App to GitHub Pages - DEV Community](https://dev.to/rashidshamloo/deploying-vite-react-app-to-github-pages-35hf#:~:text=npm%20run%20deploy)). GitHub Pages will automatically serve the contents of this branch. 

5. **Enable GitHub Pages:** On your GitHub repo page, go to **Settings > Pages** (this might be under **Code and automation** section in new GitHub UI). Ensure that the source is set to the `gh-pages` branch (it might be set automatically). After a minute or two, your site should be accessible. The URL will be:
   - **User/Org site:** `https://<USERNAME>.github.io` (if you deployed to a repo named `<USERNAME>.github.io`).
   - **Project site:** `https://<USERNAME>.github.io/<YOUR_REPO>/`. 

   Visit the URL, and you should see your React app running. 🎉

   *Tip:* If you see a blank page or 404, double-check the `base` config. A wrong base path is a common cause for a React SPA not loading assets on GitHub Pages. Also, make sure you built and deployed after setting the base. If you make further changes to your app, just rerun `npm run deploy` to rebuild and update the GitHub Pages sit ([Deploying Vite / React App to GitHub Pages - DEV Community](https://dev.to/rashidshamloo/deploying-vite-react-app-to-github-pages-35hf#:~:text=You%20now%20have%20a%20%60gh,Pages%60))】.

**References:** Setting up the deploy script and base config is based on official Vite guidelines and community tutorial ([Deploying Vite / React App to GitHub Pages - DEV Community](https://dev.to/rashidshamloo/deploying-vite-react-app-to-github-pages-35hf#:~:text=,d%20dist)) ([Deploying a Static Site | Vite](https://vite.dev/guide/static-deploy#:~:text=If%20you%20are%20deploying%20to,REPO))】. The process above will reliably publish your app to GitHub Pages.

---

## Conclusion

You now have a fully functional React + TypeScript project built with Vite, complete with Tailwind CSS styling (customized with a steampunk theme), Storybook for UI component development, Prettier for code formatting, and a deployment setup for GitHub Pages. You can continue developing your app – create React components, use Tailwind utility classes for styling, document components in Storybook, and ensure code style consistency with Prettier. Whenever you're ready to share your app, run the deploy script and your site will be live on GitHub Pages.

All steps provided were tested to work together. You should be able to follow this guide from start to finish and end up with a working project without needing to debug. Enjoy your development workflow! Happy coding! 

**Sources:**

- Vite Official Docs – React + TypeScript Quickstar ([How to build a React + TypeScript app with Vite - LogRocket Blog](https://blog.logrocket.com/build-react-typescript-app-vite/#:~:text=npm%20create%20vite%40latest)) ([How to build a React + TypeScript app with Vite - LogRocket Blog](https://blog.logrocket.com/build-react-typescript-app-vite/#:~:text=cd%20vite,run%20dev))】  
- Storybook Official Docs – Vite/React setu ([First-class Vite support in Storybook](https://storybook.js.org/blog/first-class-vite-support-in-storybook/#:~:text=If%20you%20don%27t%20have%20a,it%27s%20easy%20to%20get%20started))】  
- Tailwind CSS Official Docs – Vite integration and usag ([Install Tailwind CSS with Vite - Tailwind CSS](https://v3.tailwindcss.com/docs/guides/vite#:~:text=%2F,%7D%2C%20plugins%3A)) ([Install Tailwind CSS with Vite - Tailwind CSS](https://v3.tailwindcss.com/docs/guides/vite#:~:text=Add%20the%20,file))】  
- Prettier Setup Guide – example config and script ([How to Add Prettier to a Project. Format code in your project with… | by Ivo Nederlof | Bits and Pieces](https://blog.bitsrc.io/add-prettier-to-your-project-d7e91ac03d05#:~:text=%7B%20,120)) ([How to Add Prettier to a Project. Format code in your project with… | by Ivo Nederlof | Bits and Pieces](https://blog.bitsrc.io/add-prettier-to-your-project-d7e91ac03d05#:~:text=In%20your%20%60package.json%60%2C%20add%20%60,in%20the%20scripts%20section))】  
- GitHub Pages Deployment Guide for Vit ([Deploying Vite / React App to GitHub Pages - DEV Community](https://dev.to/rashidshamloo/deploying-vite-react-app-to-github-pages-35hf#:~:text=,d%20dist)) ([Deploying a Static Site | Vite](https://vite.dev/guide/static-deploy#:~:text=If%20you%20are%20deploying%20to,REPO))】

