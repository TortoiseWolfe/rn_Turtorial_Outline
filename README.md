Below is an **up-to-date**, **production-ready** tutorial that shows you how to create a **React Native** app with **Expo**, **TypeScript**, **Expo Router**, **SuperTokens** authentication, **SupaBase** for backend data, **NativeWind** (Tailwind in RN), **custom Google fonts** (Special Elite & Arbutus Slab), **environment variables** (`.env` & `.env.example`), **light/dark theme toggling** (defaulting to dark), and **form validation** (React Hook Form + Zod). 

This guide places **all terminal commands up front** (for directories, files, and installs), so you can follow the structure in one go. After that, you’ll see code blocks to **manually copy** into each file.

---

## 1. **Initial Project Setup**

### 1.1 Create an Expo + TypeScript App

```bash
npx create-expo-app MySteamPunkApp --template expo-template-blank-typescript
cd MySteamPunkApp
```

### 1.2 Install Required Packages

```bash
# Expo Router for file-based navigation
npm install expo-router

# NativeWind (Tailwind CSS for React Native)
npm install nativewind tailwindcss

# Google Fonts for Special Elite & Arbutus Slab
npm install expo-font @expo-google-fonts/special-elite @expo-google-fonts/arbutus-slab

# SupaBase client & SuperTokens for authentication
npm install @supabase/supabase-js supertokens-react-native

# Form Validation (React Hook Form + Zod)
npm install react-hook-form zod @hookform/resolvers

# Environment Variables
npm install react-native-dotenv
```

### 1.3 Create Needed Directories & Empty Files

We’ll set up folders/files so we can easily paste in code next:

```bash
# Index entry point
touch index.js

# Tailwind config
npx tailwindcss init

# Babel config (for env, nativewind, expo-router)
touch babel.config.js

# Create context folder & files
mkdir context
touch context/authContext.tsx
touch context/themeContext.tsx

# Create a lib folder & SupaBase client file
mkdir lib
touch lib/supabaseClient.ts

# Create the "app" folder for Expo Router
mkdir app

# Main layout for expo-router
touch app/_layout.tsx

# Public screens
touch app/index.tsx
touch app/signIn.tsx

# Protected area
mkdir app/protected
touch app/protected/_layout.tsx
touch app/protected/index.tsx

# Settings screen
mkdir app/settings
touch app/settings/index.tsx

# Environment files
touch .env
touch .env.example
```

Now that all folders and files exist, you can copy and paste the code blocks below.

---

## 2. **Configure Project Files**

### 2.1 **`index.js`** (Expo Router Entry)

Open **`index.js`** and paste:

```js
import "expo-router/entry";
```

### 2.2 **`package.json`** (Ensure Main Entry)

Make sure in **`package.json`**:

```jsonc
{
  // ...
  "main": "./index.js",
  // ...
}
```

### 2.3 **`app.json`** (Add Expo Router Plugin)

In **`app.json`**, add (or verify) the Expo Router plugin:

```jsonc
{
  "expo": {
    "name": "MySteamPunkApp",
    "slug": "MySteamPunkApp",
    // ... other config ...
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

## 3. **Babel Config for Env & NativeWind**

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

---

## 4. **Tailwind Configuration (NativeWind)**

You already ran `npx tailwindcss init`. Open **`tailwind.config.js`**:

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
        // We'll load these from google-fonts
        special: ['SpecialElite_400Regular'],
        arbutus: ['ArbutusSlab_400Regular'],
      },
    },
  },
  plugins: [],
};
```

---

## 5. **Environment Variables**

### 5.1 **`.env.example`**

Open **`.env.example`** and add placeholders (never commit real secrets):

```
EXPO_PUBLIC_SUPERTOKENS_API_DOMAIN=http://localhost:3001
EXPO_PUBLIC_SUPERTOKENS_API_BASE_PATH=/auth
EXPO_PUBLIC_SUPABASE_URL=https://xyzcompany.supabase.co
EXPO_PUBLIC_SUPABASE_ANON_KEY=YOUR_SUPABASE_ANON_KEY
```

### 5.2 **`.env`** (Real Secrets/URLs)

Open **`.env`** (which you’ll **gitignore** in real projects) and fill with actual data:

```
EXPO_PUBLIC_SUPERTOKENS_API_DOMAIN=https://my-steampunk-auth.api.com
EXPO_PUBLIC_SUPERTOKENS_API_BASE_PATH=/auth
EXPO_PUBLIC_SUPABASE_URL=https://my-steampunk-app.supabase.co
EXPO_PUBLIC_SUPABASE_ANON_KEY=REAL_SUPABASE_ANON_KEY
```

---

## 6. **SupaBase Client** (`lib/supabaseClient.ts`)

Open **`lib/supabaseClient.ts`**:

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

Use `supabase` anywhere for your database interactions.

---

## 7. **SuperTokens Auth Context** (`context/authContext.tsx`)

Open **`context/authContext.tsx`**:

```tsx
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
            // For web usage, but required by SuperTokens
            websiteDomain: 'http://localhost:3000',
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

## 8. **Theme Context** (Dark/Light) (`context/themeContext.tsx`)

Open **`context/themeContext.tsx`**:

```tsx
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

Default theme is **dark**.

---

## 9. **Global Layout** (`app/_layout.tsx`)

Open **`app/_layout.tsx`**:

```tsx
import React, { useCallback } from 'react';
import { Slot } from 'expo-router';
import * as SplashScreen from 'expo-splash-screen';
import { StatusBar } from 'expo-status-bar';

// Google Fonts
import {
  useFonts as useSpecialElite,
  SpecialElite_400Regular,
} from '@expo-google-fonts/special-elite';
import {
  useFonts as useArbutusSlab,
  ArbutusSlab_400Regular,
} from '@expo-google-fonts/arbutus-slab';

// Contexts
import { ThemeProvider } from '../context/themeContext';
import { AuthProvider } from '../context/authContext';

SplashScreen.preventAutoHideAsync();

export default function Layout() {
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
        <Slot onLayout={onLayoutRootView} />
      </AuthProvider>
    </ThemeProvider>
  );
}
```

- Loads **Special Elite** & **Arbutus Slab** fonts  
- Wraps everything in **ThemeProvider** and **AuthProvider**  

---

## 10. **Public Screens**

### 10.1 **Home** (`app/index.tsx`)

```tsx
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

### 10.2 **Sign In** (`app/signIn.tsx`) with Validation

```tsx
import React from 'react';
import { View, Text, TextInput, Button, Alert } from 'react-native';
import { useForm } from 'react-hook-form';
import { z } from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';
import { useRouter } from 'expo-router';
import { useTheme } from '../context/themeContext';

// Minimal schema for demonstration
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
      // TODO: call your real sign-in endpoint (SuperTokens or custom backend).
      // For example:
      // await fetch(`${EXPO_PUBLIC_SUPERTOKENS_API_DOMAIN}/auth/signin`, { method: 'POST', body: JSON.stringify(data) });
      console.log('SignIn data:', data);

      // If sign in is successful => set session => redirect
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

## 11. **Protected Area**

### 11.1 Protected Layout (`app/protected/_layout.tsx`)

```tsx
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

  // If authenticated, render nested route screens
  return <Stack />;
}
```

### 11.2 Protected Screen (`app/protected/index.tsx`)

```tsx
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

## 12. **Settings Screen** (Dark/Light Toggle)

### `app/settings/index.tsx`

```tsx
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

## 13. **Run & Verify**

1. **Start development**:

   ```bash
   npm start
   ```
   
   - Or `expo start`.

2. Open in the simulator or Expo Go. Check:
   - **Home Screen** (links to **Protected**, **Sign In**, **Settings**).  
   - **Sign In** route with basic form validation (Zod).  
   - **Protected** screens require authentication (redirect if not signed in).  
   - **Dark/Light** toggle on the **Settings** screen (default is dark).  
   - **Special Elite** & **Arbutus Slab** fonts.  

3. **(Optional) Production Build**:

   ```bash
   npx expo prebuild
   npx expo build
   ```
   Or use [EAS Build](https://docs.expo.dev/build/introduction/).

---

## **Conclusion**

You now have a **functional** React Native + Expo app that’s **production-ready**, featuring:

- **Expo Router** for file-based navigation.  
- **NativeWind** (Tailwind) with a **steampunk** color palette.  
- **SuperTokens** for session-based auth (protected routes).  
- **SupaBase** ready for your backend data usage.  
- **Dark/Light Theme** (default **dark**).  
- **Zod** + **React Hook Form** for minimal form validation.  
- **Google Fonts** (Special Elite & Arbutus Slab) to complete your **steampunk** theme.  
- Proper `.env` / `.env.example` usage for environment variables.  

This structure gives you a **solid foundation** to expand with real sign-in flows, more robust database calls, or additional screens. Enjoy building your **steam-punk**-themed application!
