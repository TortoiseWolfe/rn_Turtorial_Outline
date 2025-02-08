```bash
#!/usr/bin/env bash
set -e

##############################################
        # Expo Setup Script Template
##############################################

#region Metadata and Configuration
# -----------------------------------------------------------------------------
# Script Name:        expo-setup.sh
# Description:        Non-Interactive Expo setup script for initializing an
#                     Expo project with additional configuration for Supabase,
#                     Zustand, Tailwind + NativeWind, etc.
# Author:             TurtleWolfe@ScriptHammer.com
# Created:            2025-02-08
# Version:            1.0.0
# License:            MIT
#
# Best Practices:
#   - Ensure that all environment variables are securely defined.
#   - Use meaningful variable names and consistent formatting.
#   - Document each section thoroughly for maintainability.
# -----------------------------------------------------------------------------
#endregion

#region Environment Variables and Pre-Setup
# TODO: Set environment variables (e.g., APP_NAME, USE_THEME) and perform any pre-setup actions.
#endregion

#region Create Expo App
npx create-expo-app ScriptHammer
#endregion

#region Enter Project Directory
cd ScriptHammer
# TODO: Change directory to the created project directory.
#endregion

#region Create .env.local File
# TODO: Create a .env.local file with your Supabase credentials.
#endregion

#region Reset Project and Remove Default Files
npm run reset-project
rm -rf app-example
#endregion

#region Install Dependencies
# TODO: Install core packages, Expo packages, NativeWind, Tailwind CSS, etc.
#endregion

#region Create Directory Structure
# TODO: Create the necessary directory structure (e.g., app folders, assets, store, lib, components).
#endregion

#region Create Project Files
# TODO: Create essential project files such as:
#       - Supabase client (lib/supabase.ts)
#       - Zustand stores (store/useAuthStore.ts, store/useThemeStore.ts)
#       - Layouts and screens (app/_layout.tsx, app/(auth)/, app/(protected)/, etc.)
#endregion

#region Babel Configuration
# TODO: Create or update babel.config.js for NativeWind and Expo Router.
#endregion

#region Optional: Download Fonts
# TODO: Optionally download steampunk fonts if USE_THEME is enabled.
#endregion

#region Final Instructions and Launch
# TODO: Print final instructions and launch the Expo app.
#endregion

```

```bash
my-app/
└── src/
    └── app/
        ├── _layout.tsx               // Root layout for routing
        ├── (tabs)/
        │   ├── _layout.tsx           // Tab navigator layout
        │   ├── index.tsx             // Home Screen
        │   ├── explore.tsx           // Explore Screen
        │   ├── create.tsx            // Create Screen
        │   ├── notifications.tsx     // Notifications Screen
        │   ├── messages.tsx          // Messages Screen
        │   ├── groups.tsx            // Groups Screen
        │   └── profile.tsx           // Profile Screen
        ├── (auth)/
        │   ├── signIn.tsx            // Sign In Screen
        │   └── signUp.tsx            // Sign Up Screen
        └── (admin)/
            └── index.tsx             // Admin Panel
```

## 1) Set Up Your Expo Project

```bash
npx create-expo-app ScriptHammer
cd ScriptHammer
npm run reset-project
rm -rf app-example
```

---

## 2) Install Dependencies

```bash
npm install @supabase/supabase-js
npx expo install expo-secure-store
npm install dotenv
```

- **@supabase/supabase-js**: the official Supabase client library.  
- **expo-secure-store**: secure session storage on iOS/Android.  
- **dotenv**: to load `.env.local` variables prefixed with `EXPO_PUBLIC_`.

---

## 3) Create & Configure `.env.local`

At the **root** of your project (same level as `package.json`), create a file named **`.env.local`**:

```bash
EXPO_PUBLIC_SUPABASE_URL=https://YOUR-PROJECT.supabase.co
EXPO_PUBLIC_SUPABASE_ANON_KEY=YOUR-ANON-KEY
```
