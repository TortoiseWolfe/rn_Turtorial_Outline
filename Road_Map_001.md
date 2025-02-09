- **Fold (Collapse) Current Region:**  
  `Ctrl + Shift + [`

- **Unfold (Expand) Current Region:**  
  `Ctrl + Shift + ]`

- **Fold All Regions:**  
  `Ctrl + K`, then `Ctrl + 0`

- **Unfold All Regions:**  
  `Ctrl + K`, then `Ctrl + J`

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
#   - Validate critical variables before proceeding.
# -----------------------------------------------------------------------------
#endregion

#region Environment Variables and Pre-Setup
# Load environment variables from .env file if it exists.
if [ -f .env ]; then
  # Export variables defined in the .env file
  set -o allexport
  source .env
  set +o allexport
else
  echo "Error: .env file not found. Please create one with the necessary environment variables."
  exit 1
fi

# Validate that APP_NAME is set.
if [ -z "$APP_NAME" ]; then
  echo "Error: APP_NAME is not set in the .env file."
  exit 1
fi
#endregion

#region Create Expo App
# Create the Expo app using the project name from APP_NAME.
npx create-expo-app "$APP_NAME"

# Change into the project directory.
cd "$APP_NAME" || {
  echo "Error: Failed to enter directory '$APP_NAME'. Exiting."
  exit 1
}

# Run the reset-project command and remove any default folders.
npm run reset-project
rm -rf app-example
#endregion

#region Install Dependencies
# TODO: Install core packages, Expo packages, NativeWind, Tailwind CSS, etc.
#endregion

# #region CREATE DIRECTORY STRUCTURE
# my-app/
# ├── .env.local                    # Environment variables (e.g., API keys, project settings)
# ├── app.json                      # Expo configuration file
# ├── babel.config.js               # Babel configuration (e.g., for NativeWind, Expo Router)
# ├── metro.config.js               # Metro bundler configuration (optional, for advanced setups)
# ├── package.json                  # Project manifest with dependencies and scripts
# ├── tsconfig.json                 # TypeScript configuration (if you're using TypeScript)
# └── src/
#     └── app/
#         ├── _layout.tsx           # Root layout for global routing and configuration
#         ├── error.tsx             # Global error boundary for routes
#         ├── global.css            # Global styles for the app
#         ├── loading.tsx           # Global loading component (shown during route transitions)
#         └── not-found.tsx         # 404 page for unmatched routes
#         #region (auth) ROUTE GROUP
#         ├── (auth)/
#         │   ├── _layout.tsx       # Auth-specific layout (e.g., authentication wrappers)
#         │   ├── signIn.tsx        # Sign In Screen
#         │   └── signUp.tsx        # Sign Up Screen
#         #endregion
#         #region (tabs) ROUTE GROUP
#         ├── (tabs)/
#         │   ├── _layout.tsx       # Tab navigator layout (manages tab navigation)
#         │   ├── index.tsx         # Home Screen
#         │   ├── create.tsx        # Create Screen
#         │   ├── explore.tsx       # Explore Screen
#         │   ├── groups.tsx        # Groups Screen
#         │   └── map.tsx           # Map Page for Geolocation
#         │   ├── messages.tsx      # Messages Screen
#         │   ├── notifications.tsx # Notifications Screen
#         │   ├── profile.tsx       # Profile Screen
#         #endregion
#         #region (admin) ROUTE GROUP
#         ├── (admin)/
#         │   ├── _layout.tsx       # Admin-specific layout (wraps admin routes)
#         │   └── index.tsx         # Admin Panel
#         #endregion
# TODO: Create essential project files such as:
#          - Supabase client (lib/supabase.ts)
#          - Zustand stores (store/useAuthStore.ts, store/useThemeStore.ts)
# #endregion

#region Babel Configuration
# TODO: Create or update babel.config.js for NativeWind and Expo Router.
#endregion

#region Optional: Download Fonts
# TODO: Optionally download steampunk fonts Arburtus Slab & Elite Special from Google Fonts if USE_THEME is enabled.
#endregion

#region Final Instructions and Launch
# TODO: Print final instructions and launch the Expo app.
npx expo start --clear
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
