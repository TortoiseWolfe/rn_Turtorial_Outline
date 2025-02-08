```bash

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
