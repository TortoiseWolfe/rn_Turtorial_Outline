Below is a **complete**, **functional**, and **up-to-date** tutorial showing how to create a **TypeScript** React Native app with **Expo**, **Expo Router**, **NativeWind** (Tailwind CSS), **SupaBase** for data, **SuperTokens** for authentication, **environment variables**, **custom Google fonts** (Special Elite & Arbutus Slab), **dark/light theme** toggling (default: dark), **protected routes**, and **basic validation** (React Hook Form + Zod). 

**Importantly**, we will not reference or create any `index.js`; we'll use `index.ts` from the start, ensuring there’s no confusion about entry points.

---

# 1. **Project Setup**

## 1.1 **Create an Expo + TypeScript Project**

```bash
# Create a new Expo app using the TypeScript template
npx create-expo-app MySteamPunkApp --template expo-template-blank-typescript

cd MySteamPunkApp
```

> This scaffold includes an `App.tsx` by default. Because we’ll use **Expo Router**, we won’t use `App.tsx` as our root. You can **delete** `App.tsx` or reuse its code in our router screens.

## 1.2 **Install All Needed Packages**

```bash
# 1. Expo Router for file-based navigation
npm install expo-router

# 2. NativeWind (Tailwind in RN)
npm install nativewind tailwindcss

# 3. Google Fonts (Special Elite + Arbutus Slab)
npm install expo-font @expo-google-fonts/special-elite @expo-google-fonts/arbutus-slab

# 4. SupaBase client + SuperTokens (auth)
npm install @supabase/supabase-js supertokens-react-native

# 5. Form validation (React Hook Form + Zod)
npm install react-hook-form zod @hookform/resolvers

# 6. Environment variables
npm install react-native-dotenv
```

## 1.3 **Create / Modify Files & Folders**

We’ll create an **`index.ts`** (instead of `index.js`) as our entry. Also, we’ll set up the folders for our contexts, library, and `app/` routes:

```bash
# 1. Rename (or create) index.ts to load Expo Router
touch index.ts

# 2. Babel config for env, nativewind, expo-router
touch babel.config.js

# 3. Tailwind config (already made by 'npx tailwindcss init' but ensure file is present)
npx tailwindcss init

# 4. Env files
touch .env
touch .env.example

# 5. Create context folder
mkdir context
touch context/authContext.tsx
touch context/themeContext.tsx

# 6. Create lib folder for supabase client
mkdir lib
touch lib/supabaseClient.ts

# 7. The app folder for Expo Router
mkdir app

touch app/_layout.tsx     # global layout
touch app/index.tsx       # home screen
touch app/signIn.tsx      # sign in

# Protected area
mkdir app/protected
touch app/protected/_layout.tsx
touch app/protected/index.tsx

# Settings
mkdir app/settings
touch app/settings/index.tsx
```

Finally, you can remove or ignore the default `App.tsx` if you don’t plan to repurpose it.

---

# 2. **Configure Core Files**

## 2.1 **`package.json`**: Use `index.ts` as the Main Entry

Open your **`package.json`** and ensure `"main"` points to **`index.ts`**:

```jsonc
{
  // ...
  "main": "./index.ts",
  // ...
}
```

## 2.2 **`index.ts`** (Load Expo Router)

Open (or create) **`index.ts`** and paste:

```ts
import 'expo-router/entry';
```

That’s it. Now Expo Router will automatically look for files in the `app/` directory.

## 2.3 **`app.json`** (Configure Expo Router Plugin)

In **`app.json`**, add or confirm the Expo Router plugin:

```jsonc
{
  "expo": {
    "name": "MySteamPunkApp",
    "slug": "MySteamPunkApp",
    // ...
    "plugins": [
      [
        "expo-router",
        {
          "origin": "https://expo.dev"
        }
      ]
    ]
  }
}
```

---

# 3. **Babel & Tailwind Config**

## 3.1 **`babel.config.js`**

Open **`babel.config.js`** and paste:

```js
module.exports = function (api) {
  api.cache(true);
  return {
    presets: ['babel-preset-expo'],
    plugins: [
      [
        'react-native-dotenv',
        {
          moduleName: '@env',
          path: '.env',
        },
      ],
      'nativewind/babel',
      'expo-router/babel',
    ],
  };
};
```

## 3.2 **`tailwind.config.js`**

After `npx tailwindcss init`, open **`tailwind.config.js`** and replace with:

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    './App.{js,ts,jsx,tsx}',
    './app/**/*.{js,ts,jsx,tsx}'
  ],
  theme: {
    extend: {
      colors: {
        steampunkGold: '#b08d57',
        steampunkBrown: '#4e3e31',
        darkBackground: '#1a1a1a',
        lightBackground: '#ffffff',
      },
      fontFamily: {
        special: ['SpecialElite_400Regular'],
        arbutus: ['ArbutusSlab_400Regular'],
      },
    },
  },
  plugins: [],
};
```

---

# 4. **Environment Variables**

## 4.1 **`.env.example`**

Never store real secrets here; use placeholders:

```
EXPO_PUBLIC_SUPERTOKENS_API_DOMAIN=http://localhost:3001
EXPO_PUBLIC_SUPERTOKENS_API_BASE_PATH=/auth
EXPO_PUBLIC_SUPABASE_URL=https://xyzcompany.supabase.co
EXPO_PUBLIC_SUPABASE_ANON_KEY=YOUR_SUPABASE_ANON_KEY
```

## 4.2 **`.env`**

In **`.env`** (ignored by Git), place real values:

```
EXPO_PUBLIC_SUPERTOKENS_API_DOMAIN=https://my-steampunk-auth.api.com
EXPO_PUBLIC_SUPERTOKENS_API_BASE_PATH=/auth
EXPO_PUBLIC_SUPABASE_URL=https://my-steampunk-app.supabase.co
EXPO_PUBLIC_SUPABASE_ANON_KEY=REAL_SUPABASE_ANON_KEY
```

---

# 5. **SupaBase Client** (`lib/supabaseClient.ts`)

```ts
import { createClient } from '@supabase/supabase-js';
import {
  EXPO_PUBLIC_SUPABASE_URL,
  EXPO_PUBLIC_SUPABASE_ANON_KEY,
} from '@env';

export const supabase = createClient(
  EXPO_PUBLIC_SUPABASE_URL,
  EXPO_PUBLIC_SUPABASE_ANON_KEY
);
```

Use `supabase` anywhere for DB queries, storage, etc.

---

# 6. **Auth Context** (`context/authContext.tsx`)

Integrate **SuperTokens** to check sessions on app startup:

```ts
import React, { createContext, useContext, useEffect, useState } from 'react';
import SuperTokens from 'supertokens-react-native';
import Session from 'supertokens-react-native/recipe/session';
import {
  EXPO_PUBLIC_SUPERTOKENS_API_DOMAIN,
  EXPO_PUBLIC_SUPERTOKENS_API_BASE_PATH,
} from '@env';

type AuthContextType = {
  isAuthenticated: boolean;
  loadingAuth: boolean;
};

const AuthContext = createContext<AuthContextType>({
  isAuthenticated: false,
  loadingAuth: true,
});

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [isAuthenticated, setIsAuthenticated] = useState(false);
  const [loadingAuth, setLoadingAuth] = useState(true);

  useEffect(() => {
    (async () => {
      try {
        await SuperTokens.init({
          appInfo: {
            appName: 'MySteamPunkApp',
            apiDomain: EXPO_PUBLIC_SUPERTOKENS_API_DOMAIN,
            apiBasePath: EXPO_PUBLIC_SUPERTOKENS_API_BASE_PATH,
            websiteDomain: 'http://localhost:3000', // used in web contexts
          },
          recipeList: [Session.init()],
        });

        const sessionExists = await Session.doesSessionExist();
        setIsAuthenticated(sessionExists);
      } catch (error) {
        console.error('SuperTokens init error:', error);
      } finally {
        setLoadingAuth(false);
      }
    })();
  }, []);

  return (
    <AuthContext.Provider value={{ isAuthenticated, loadingAuth }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  return useContext(AuthContext);
}
```

---

# 7. **Theme Context** (`context/themeContext.tsx`)

Implement a simple **dark/light** toggle (default is **dark**):

```ts
import React, { createContext, useState, useContext } from 'react';

type ThemeContextType = {
  theme: 'dark' | 'light';
  toggleTheme: () => void;
};

const ThemeContext = createContext<ThemeContextType>({
  theme: 'dark',
  toggleTheme: () => {},
});

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<'dark' | 'light'>('dark');

  const toggleTheme = () => {
    setTheme((prev) => (prev === 'dark' ? 'light' : 'dark'));
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  return useContext(ThemeContext);
}
```

---

# 8. **Global Layout** (`app/_layout.tsx`)

Expo Router uses `_layout.tsx` as a special file to wrap all routes. We’ll load fonts, show a splash until they’re ready, and wrap providers:

```ts
import React, { useCallback } from 'react';
import { Slot } from 'expo-router';
import * as SplashScreen from 'expo-splash-screen';
import { StatusBar } from 'expo-status-bar';

// Fonts
import {
  useFonts as useSpecialElite,
  SpecialElite_400Regular,
} from '@expo-google-fonts/special-elite';
import {
  useFonts as useArbutusSlab,
  ArbutusSlab_400Regular,
} from '@expo-google-fonts/arbutus-slab';

// Contexts
import { AuthProvider } from '../context/authContext';
import { ThemeProvider } from '../context/themeContext';

SplashScreen.preventAutoHideAsync();

export default function RootLayout() {
  const [specialEliteLoaded] = useSpecialElite({
    SpecialElite_400Regular,
  });
  const [arbutusLoaded] = useArbutusSlab({
    ArbutusSlab_400Regular,
  });

  const fontsLoaded = specialEliteLoaded && arbutusLoaded;

  const onLayoutRootView = useCallback(async () => {
    if (fontsLoaded) {
      await SplashScreen.hideAsync();
    }
  }, [fontsLoaded]);

  if (!fontsLoaded) {
    return null; // or a custom splash screen
  }

  return (
    <ThemeProvider>
      <AuthProvider>
        <StatusBar style="light" />
        {/* The Slot will render nested routes */}
        <Slot onLayout={onLayoutRootView} />
      </AuthProvider>
    </ThemeProvider>
  );
}
```

---

# 9. **Public Screens**

## 9.1 **Home** (`app/index.tsx`)

```ts
import React from 'react';
import { View, Text } from 'react-native';
import { Link } from 'expo-router';
import { useTheme } from '../context/themeContext';

export default function HomeScreen() {
  const { theme } = useTheme();
  const isDark = theme === 'dark';

  return (
    <View
      className={`flex-1 justify-center items-center ${
        isDark ? 'bg-darkBackground' : 'bg-lightBackground'
      }`}
    >
      <Text
        className={`font-special text-xl mb-4 ${
          isDark ? 'text-steampunkGold' : 'text-steampunkBrown'
        }`}
      >
        Welcome to the SteamPunk Home
      </Text>

      <Link href="/protected" className="underline mb-2">
        Go to Protected Area
      </Link>
      <Link href="/signIn" className="underline mb-2">
        Sign In
      </Link>
      <Link href="/settings" className="underline">
        Settings
      </Link>
    </View>
  );
}
```

## 9.2 **Sign In** (`app/signIn.tsx`) + Zod Validation

```ts
import React from 'react';
import { View, Text, TextInput, Button, Alert } from 'react-native';
import { useForm } from 'react-hook-form';
import { z } from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';
import { useRouter } from 'expo-router';
import { useTheme } from '../context/themeContext';

// Minimal schema example
const signInSchema = z.object({
  email: z.string().email('Invalid email address'),
  password: z.string().min(6, 'Password must be at least 6 characters'),
});

type SignInFormData = z.infer<typeof signInSchema>;

export default function SignInScreen() {
  const { theme } = useTheme();
  const isDark = theme === 'dark';
  const router = useRouter();

  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<SignInFormData>({
    resolver: zodResolver(signInSchema),
  });

  const onSubmit = async (data: SignInFormData) => {
    try {
      // TODO: call real sign-in endpoint (SuperTokens or custom).
      // e.g. fetch(`${EXPO_PUBLIC_SUPERTOKENS_API_DOMAIN}/auth/signin`, { method: 'POST', body: JSON.stringify(data) });

      // If sign in is successful => redirect
      router.replace('/protected');
    } catch (error: any) {
      Alert.alert('Sign In Error', error?.message || 'Unknown error');
    }
  };

  return (
    <View
      className={`flex-1 justify-center px-4 ${
        isDark ? 'bg-darkBackground' : 'bg-lightBackground'
      }`}
    >
      <Text
        className={`text-2xl font-arbutus mb-6 text-center ${
          isDark ? 'text-steampunkGold' : 'text-steampunkBrown'
        }`}
      >
        Sign In
      </Text>

      <TextInput
        className={`border rounded-md p-2 mb-2 ${
          isDark ? 'border-steampunkGold text-white' : 'border-steampunkBrown text-black'
        }`}
        placeholder="Email"
        placeholderTextColor={isDark ? '#888' : '#666'}
        {...register('email')}
      />
      {errors.email && (
        <Text className="text-red-500 mb-2">{errors.email.message}</Text>
      )}

      <TextInput
        className={`border rounded-md p-2 mb-2 ${
          isDark ? 'border-steampunkGold text-white' : 'border-steampunkBrown text-black'
        }`}
        placeholder="Password"
        placeholderTextColor={isDark ? '#888' : '#666'}
        secureTextEntry
        {...register('password')}
      />
      {errors.password && (
        <Text className="text-red-500 mb-2">{errors.password.message}</Text>
      )}

      <Button title="Sign In" onPress={handleSubmit(onSubmit)} />
    </View>
  );
}
```

---

# 10. **Protected Routes**

## 10.1 `_layout.tsx` (`app/protected/_layout.tsx`)

```ts
import React, { useEffect } from 'react';
import { Stack, useRouter } from 'expo-router';
import { useAuth } from '../../context/authContext';
import { View, ActivityIndicator } from 'react-native';
import { useTheme } from '../../context/themeContext';

export default function ProtectedLayout() {
  const { isAuthenticated, loadingAuth } = useAuth();
  const router = useRouter();
  const { theme } = useTheme();
  const isDark = theme === 'dark';

  useEffect(() => {
    if (!loadingAuth && !isAuthenticated) {
      // If not authenticated => redirect to signIn
      router.replace('/signIn');
    }
  }, [loadingAuth, isAuthenticated]);

  if (loadingAuth) {
    return (
      <View
        className={`flex-1 justify-center items-center ${
          isDark ? 'bg-darkBackground' : 'bg-lightBackground'
        }`}
      >
        <ActivityIndicator size="large" color="#b08d57" />
      </View>
    );
  }

  // If authenticated, show nested route screens
  return <Stack />;
}
```

## 10.2 Protected Screen (`app/protected/index.tsx`)

```ts
import React from 'react';
import { View, Text } from 'react-native';
import { useTheme } from '../../context/themeContext';

export default function ProtectedHome() {
  const { theme } = useTheme();
  const isDark = theme === 'dark';

  return (
    <View
      className={`flex-1 justify-center items-center ${
        isDark ? 'bg-darkBackground' : 'bg-lightBackground'
      }`}
    >
      <Text
        className={`text-xl font-arbutus ${
          isDark ? 'text-steampunkGold' : 'text-steampunkBrown'
        }`}
      >
        Protected Steampunk Area
      </Text>
    </View>
  );
}
```

---

# 11. **Settings Screen** (Dark/Light Toggle)

## `app/settings/index.tsx`

```ts
import React from 'react';
import { View, Text, Switch } from 'react-native';
import { useTheme } from '../../context/themeContext';

export default function SettingsScreen() {
  const { theme, toggleTheme } = useTheme();
  const isDark = theme === 'dark';

  return (
    <View
      className={`flex-1 justify-center items-center ${
        isDark ? 'bg-darkBackground' : 'bg-lightBackground'
      }`}
    >
      <Text
        className={`text-xl font-arbutus mb-4 ${
          isDark ? 'text-steampunkGold' : 'text-steampunkBrown'
        }`}
      >
        Settings
      </Text>
      <View className="flex-row items-center">
        <Text
          className={`mr-2 ${
            isDark ? 'text-steampunkGold' : 'text-steampunkBrown'
          }`}
        >
          Dark Mode
        </Text>
        <Switch
          value={isDark}
          onValueChange={toggleTheme}
          thumbColor={isDark ? '#b08d57' : '#f4f3f4'}
          trackColor={{ false: '#767577', true: '#b08d57' }}
        />
      </View>
    </View>
  );
}
```

---

# 12. **Hidden vs. Protected?**

- **Protected** (like `/protected`) requires auth. If unauthorized, the user is redirected.
- **Hidden** routes are not automatically listed or linked in the UI. You can hide routes by:
  - Not linking to them, or
  - Placing them in a parent folder with parentheses (e.g., `app/(secret)/...`).  
This tutorial shows a **protected** route, not a truly “hidden” route.

---

# 13. **Run & Verify**

1. **Start Dev**:

   ```bash
   npm start
   ```
   - or `expo start`.

2. Load in Expo Go or a simulator:
   - **Home** (`app/index.tsx`) with links to `Protected`, `Sign In`, and `Settings`.
   - **Sign In** route (Zod validation).
   - **Protected** route (redirects to signIn if not authenticated).
   - **Settings** toggles **dark/light** theme.
   - **Google Fonts** (Special Elite & Arbutus Slab) loaded.
   - **Tailwind** (NativeWind) styling with a “steampunk” palette.

3. **Production Build** (optional):

   ```bash
   npx expo prebuild
   npx expo build
   ```
   or use [EAS Build](https://docs.expo.dev/build/introduction/).

---

# **Conclusion**

You now have a **fully functional**, **TypeScript-based** Expo app with:

- **Expo Router** for file-based navigation (no `index.js` needed!).  
- **NativeWind** (Tailwind) and **Google Fonts** for a steampunk design.  
- **SuperTokens** for session-based auth, gating protected routes.  
- **SupaBase** client ready for your real data calls.  
- **Dark/Light** theme toggle in Settings (default: dark).  
- **React Hook Form + Zod** for minimal form validation.  
- Proper `.env` usage and a `.env.example` for best practices.  

Feel free to extend the sign-in flow with real API calls, add more screens, or incorporate advanced SupaBase features. Enjoy your **steampunk**-themed app!
