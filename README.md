# Building a Hidden Routes App
A complete guide to building a steampunk-themed mobile app with Expo, NativeWind, and Supabase

## Table of Contents

### [Chapter 1: Frontend Setup](#expo-with-nativewind-custom-fonts--steampunk-theme)
- [Project Initialization and Dependencies](#step-1-project-initialization-and-dependencies)
- [Core Configuration Files](#step-2-configure-core-files)
- [Global Styles](#step-3-set-up-global-styles)
- [Base Layout and Font Loading](#step-4-create-base-layout-with-font-loading)
- [Route Structure](#step-5-set-up-route-structure)
- [Screen Components](#step-6-create-main-screen-components)

### [Chapter 2: Production-Ready Backend Integration with Supabase](#chapter-2-production-ready-backend-integration-with-supabase)
1. [Environment Variables & Security Best Practices](#1-environment-variables--security-best-practices)  
   - [1.1 Why Environment Variables?](#11-why-environment-variables)  
   - [1.2 Setting Up Environment Variables in Expo](#12-setting-up-environment-variables-in-expo)  
   - [1.3 Secrets in CI/CD](#13-secrets-in-cicd)  
2. [Create & Configure Your Supabase Project](#2-create--configure-your-supabase-project)  
   - [2.1 Create a New Supabase Project](#21-create-a-new-supabase-project)  
   - [2.2 Retrieve Your Project Credentials](#22-retrieve-your-project-credentials)  
3. [Initialize the Supabase Client (Production Setup)](#3-initialize-the-supabase-client-production-setup)  
   - [3.1 Secure Supabase Client](#31-secure-supabase-client)  
   - [3.2 Confirming RLS (Row-Level Security) & Auth Policies](#32-confirming-rls-row-level-security--auth-policies)  
4. [Implementing Secure Sign-Up](#4-implementing-secure-sign-up)  
   - [4.1 Email Confirmations](#41-email-confirmations)  
5. [Implementing Login Flow](#5-implementing-login-flow)  
6. [Managing Session State & Protecting Routes](#6-managing-session-state--protecting-routes)  
   - [6.1 Real-Time Session Updates](#61-real-time-session-updates)  
   - [6.2 Protected Routes](#62-protected-routes)  
7. [Final Considerations for Production](#7-final-considerations-for-production)  
[Recap](#recap)
---  

# Chapter 1: Frontend Setup

# Expo with NativeWind, Custom Fonts & Steampunk Theme

## Step 1: Project Initialization and Dependencies

##### Remove existing app if present
```bash
rm -rf MyHiddenRoutesApp
```
##### Create new Expo app with TypeScript template
```bash
npx create-expo-app MyHiddenRoutesApp
```
##### Navigate to project directory & Reset project
```bash
cd MyHiddenRoutesApp
npm run reset-project
```
##### Install core dependencies
```bash
npm install nativewind tailwindcss react-native-reanimated react-native-safe-area-context
```
##### Install font packages
```bash
npm install @expo-google-fonts/fondamento @expo-google-fonts/special-elite @expo-google-fonts/arbutus-slab
```
##### Initialize Tailwind
```bash
npx tailwindcss init
```
##### Initialize metro config
```bash
npx expo customize metro.config.js
```
##### Create necessary configuration files
```bash
touch global.css babel.config.js nativewind-env.d.ts
```

## Step 2: Configure Core Files

### babel.config.js
```javascript
module.exports = function (api) {
    api.cache(true);
    return {
      presets: [
        ["babel-preset-expo", { jsxImportSource: "nativewind" }],
        "nativewind/babel",
      ],
    };
};
```

### metro.config.js
```javascript
// Learn more https://docs.expo.io/guides/customizing-metro
const { getDefaultConfig } = require('expo/metro-config');
const { withNativeWind } = require("nativewind/metro");

/** @type {import('expo/metro-config').MetroConfig} */
const config = getDefaultConfig(__dirname);

module.exports = withNativeWind(config, { input: "./global.css" });
```

### tailwind.config.js
```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
    content: ["./app/**/*.{js,jsx,ts,tsx}"],
    presets: [require("nativewind/preset")],
    theme: {
      extend: {
        fontFamily: {
          fondamento: ['Fondamento_400Regular'],
          specialElite: ['SpecialElite_400Regular'],
          arbutusSlab: ['ArbutusSlab_400Regular']
        }
      },
    },
    plugins: [],
};
```

### nativewind-env.d.ts
```typescript
/// <reference types="nativewind/types" />
```

## Step 3: Set Up Global Styles

### global.css
```css
@tailwind base;
@tailwind components;
@tailwind utilities;

/* Custom SteamPunk Theme */
@layer components {
  /* Background & Container */
  .bg-steampunk {
    @apply bg-neutral-900;
  }

  .container-steampunk {
    @apply flex-1 p-6 justify-center;
  }

  /* Text */
  .text-steampunk-title {
    @apply text-3xl text-amber-400 font-bold mb-6 font-robotoSlab;
  }

  .text-steampunk-label {
    @apply text-amber-200 mb-2 font-specialElite;
  }

  .text-steampunk-body {
    @apply text-amber-200 font-arbutusSlab;
  }

  /* Inputs */
  .input-steampunk {
    @apply w-full bg-neutral-800 text-amber-100 p-3 rounded-md font-arbutusSlab;
  }

  /* Buttons */
  .btn-steampunk {
    @apply bg-amber-700 rounded-md p-3;
  }

  .btn-text-steampunk {
    @apply text-center text-amber-50 font-semibold font-specialElite;
  }

  /* Links */
  .link-steampunk {
    @apply text-amber-400 underline hover:text-amber-300 font-specialElite;
  }

  /* Border */
  .border-steampunk {
    @apply border-2 border-amber-700 rounded-lg p-4;
  }
}
```

## Step 4: Create Base Layout with Font Loading

### app/_layout.tsx
```typescript
import { Stack } from "expo-router";
import "../global.css";
import { useFonts } from 'expo-font';
import { Fondamento_400Regular } from '@expo-google-fonts/fondamento';
import { SpecialElite_400Regular } from '@expo-google-fonts/special-elite';
import { ArbutusSlab_400Regular } from '@expo-google-fonts/arbutus-slab';
import Splash from "./(onboarding)/splash";

export default function RootLayout() {
  const [fontsLoaded] = useFonts({
    Fondamento_400Regular,
    SpecialElite_400Regular,
    ArbutusSlab_400Regular,
  });

  if (!fontsLoaded) {
    return <Splash />;
  }

  return <Stack />;
}
```

## Step 5: Set Up Route Structure

Create the necessary directories and files:

```bash
# Create auth routes
mkdir -p "app/(auth)"
touch "app/+not-found.tsx"
touch "app/(auth)/_layout.tsx" \
      "app/(auth)/login.tsx" \
      "app/(auth)/sign-up.tsx" \
      "app/(auth)/verification.tsx" \
      "app/(auth)/forgot-password.tsx"

# Create onboarding routes
mkdir -p "app/(onboarding)"
touch "app/(onboarding)/_layout.tsx" \
      "app/(onboarding)/splash.tsx" \
      "app/(onboarding)/welcome.tsx" \
      "app/(onboarding)/onboarding-setup.tsx"
```

### app/(auth)/_layout.tsx
```typescript
import { Stack } from "expo-router";
import "../../global.css";

export default function AuthLayout() {
  return <Stack />;
}
```

### app/(onboarding)/_layout.tsx
```typescript
import { Stack } from "expo-router";
import "../../global.css";

export default function OnboardingLayout() {
  return <Stack />;
}
```

## Step 6: Create Main Screen Components

### app/index.tsx (Home Screen)
```typescript
import { Text, View } from "react-native";
import { SafeAreaView } from "react-native-safe-area-context";
import { Link } from "expo-router";

export default function Index() {
  return (
    <SafeAreaView className="container-steampunk bg-steampunk p-6">
      <View className="p-6">
        <Text className="text-steampunk-title">SteamPunk App</Text>
        <Text className="text-steampunk-body">
          Welcome to your steampunk-themed application powered by Nativewind and Tailwind CSS.
        </Text>
      </View>

      {/* Navigation Links */}
      <View className="mt-8 p-6">
        {/* Onboarding process */}
        <Link href="/(onboarding)/splash" className="link-steampunk mb-4 block">
          Splash Screen
        </Link>
        <Link href="/(onboarding)/welcome" className="link-steampunk mb-4 block">
          Welcome
        </Link>
        <Link href="/(onboarding)/onboarding-setup" className="link-steampunk mb-4 block">
          Onboarding Setup
        </Link>

        {/* Authentication process */}
        <Link href="/(auth)/sign-up" className="link-steampunk mb-4 block">
          Sign Up
        </Link>
        <Link href="/(auth)/verification" className="link-steampunk mb-4 block">
          Verification
        </Link>
        <Link href="/(auth)/login" className="link-steampunk mb-4 block">
          Login
        </Link>
        <Link href="/(auth)/forgot-password" className="link-steampunk mb-4 block">
          Forgot Password
        </Link>

        {/* Utility or Error Pages */}
        <Link href="/+not-found" className="link-steampunk mb-4 block">
          Not Found
        </Link>
      </View>

      <View className="mt-8 border-steampunk max-w-md mx-left p-6">
        <Text className="text-steampunk-body">
          This is a sample box demonstrating centralized steampunk styles.
        </Text>
      </View>
    </SafeAreaView>
  );
}
```

### app/(auth)/login.tsx
```typescript
import { View, Text, TextInput, TouchableOpacity } from "react-native";

export default function Login() {
  return (
    <View className="container-steampunk bg-steampunk">
      <Text className="text-steampunk-title">SteamPunk Login</Text>

      <Text className="text-steampunk-label">Username</Text>
      <TextInput
        placeholder="Enter your username"
        placeholderTextColor="#A98274"
        className="input-steampunk mb-4"
      />

      <Text className="text-steampunk-label">Password</Text>
      <TextInput
        placeholder="Enter your password"
        placeholderTextColor="#A98274"
        secureTextEntry
        className="input-steampunk mb-6"
      />

      <TouchableOpacity className="btn-steampunk">
        <Text className="btn-text-steampunk">Login</Text>
      </TouchableOpacity>
    </View>
  );
}
```

### app/(auth)/sign-up.tsx
```typescript
import { View, Text, TextInput, TouchableOpacity } from "react-native";

export default function SignUp() {
  return (
    <View className="container-steampunk bg-steampunk">
      <Text className="text-steampunk-title">Create Account</Text>

      <Text className="text-steampunk-label">Email</Text>
      <TextInput
        placeholder="Enter your email"
        placeholderTextColor="#A98274"
        className="input-steampunk mb-4"
      />

      <Text className="text-steampunk-label">Password</Text>
      <TextInput
        placeholder="Create a password"
        placeholderTextColor="#A98274"
        secureTextEntry
        className="input-steampunk mb-4"
      />

      <Text className="text-steampunk-label">Confirm Password</Text>
      <TextInput
        placeholder="Repeat your password"
        placeholderTextColor="#A98274"
        secureTextEntry
        className="input-steampunk mb-6"
      />

      <TouchableOpacity className="btn-steampunk">
        <Text className="btn-text-steampunk">Sign Up</Text>
      </TouchableOpacity>
    </View>
  );
}
```

### app/(auth)/verification.tsx
```typescript
import { View, Text, TextInput, TouchableOpacity } from "react-native";

export default function Verification() {
  return (
    <View className="container-steampunk bg-steampunk">
      <Text className="text-steampunk-title">Email Verification</Text>

      <Text className="text-steampunk-body mb-4">
        Enter the verification code sent to your email.
      </Text>

      <Text className="text-steampunk-label">Verification Code</Text>
      <TextInput
        placeholder="Enter your 6-digit code"
        placeholderTextColor="#A98274"
        keyboardType="numeric"
        className="input-steampunk mb-6"
      />

      <TouchableOpacity className="btn-steampunk">
        <Text className="btn-text-steampunk">Verify</Text>
      </TouchableOpacity>
    </View>
  );
}
```

### app/(auth)/forgot-password.tsx
```typescript
import { View, Text, TextInput, TouchableOpacity } from "react-native";

export default function ForgotPassword() {
  return (
    <View className="container-steampunk bg-steampunk">
      <Text className="text-steampunk-title">Reset Password</Text>

      <Text className="text-steampunk-body mb-4">
        Enter your email address to receive password reset instructions.
      </Text>

      <Text className="text-steampunk-label">Email</Text>
      <TextInput
        placeholder="Enter your email"
        placeholderTextColor="#A98274"
        className="input-steampunk mb-6"
      />

      <TouchableOpacity className="btn-steampunk">
        <Text className="btn-text-steampunk">Send Reset Link</Text>
      </TouchableOpacity>
    </View>
  );
}
```

### app/(onboarding)/splash.tsx
```typescript
import { View, Text } from "react-native";

export default function Splash() {
  return (
    <View className="container-steampunk bg-steampunk items-center justify-center">
      <Text className="text-steampunk-title">Welcome to SteamPunk World</Text>
      <Text className="text-steampunk-body text-center">
        Loading your steampunk adventure...
      </Text>
    </View>
  );
}
```

### app/(onboarding)/welcome.tsx
```typescript
import { View, Text, TouchableOpacity } from "react-native";
import { Link } from "expo-router";

export default function Welcome() {
  return (
    <View className="container-steampunk bg-steampunk items-center">
      <Text className="text-steampunk-title">Welcome Aboard!</Text>
      <Text className="text-steampunk-body text-center mb-10">
        Embark on a steam-powered adventure through our app.
      </Text>

      <Link href="/(auth)/login" asChild>
        <TouchableOpacity className="btn-steampunk w-full max-w-sm">
          <Text className="btn-text-steampunk">Begin Journey</Text>
        </TouchableOpacity>
      </Link>
    </View>
  );
}
```

### app/(onboarding)/onboarding-setup.tsx
```typescript
import { View, Text, TouchableOpacity, Switch } from "react-native";

export default function OnboardingSetup() {
  const [enableNotifications, setEnableNotifications] = React.useState(false);

  return (
    <View className="container-steampunk bg-steampunk">
      <Text className="text-steampunk-title">Customize Your Experience</Text>

      <View className="flex-row items-center justify-between mb-6">
        <Text className="text-steampunk-body">Enable Notifications</Text>
        <Switch
          value={enableNotifications}
          onValueChange={setEnableNotifications}
          thumbColor={enableNotifications ? "#EFBD5D" : "#A98274"}
          trackColor={{ true: "#7B5530", false: "#374151" }}
        />
      </View>

      <TouchableOpacity className="btn-steampunk">
        <Text className="btn-text-steampunk">Complete Setup</Text>
      </TouchableOpacity>
    </View>
  );
}
```

### app/+not-found.tsx
```typescript
import { Link, Stack } from 'expo-router';
import { View, Text, TouchableOpacity } from 'react-native';

export default function NotFoundScreen() {
  return (
    <>
      <Stack.Screen options={{ title: 'Oops!' }} />
      <View className="container-steampunk bg-steampunk items-center">
        <Text className="text-steampunk-title">Page Not Found</Text>
        <Text className="text-steampunk-body mb-6">
          It seems this gear has gone missing...
        </Text>
        <Link href="/" asChild>
          <TouchableOpacity className="btn-steampunk">
            <Text className="btn-text-steampunk">Return to Steam Chamber</Text>
          </TouchableOpacity>
        </Link>
      </View>
    </>
  );
}
```

Below is an **expanded, production-focused** version of Chapter 2. It details how to set up Supabase **securely** with environment variables, adhere to **best practices**, and maintain a public open-source repository without exposing private information. This guide is intended for real-world scenarios, where secrets must be guarded carefully while still enabling open collaboration.

---

# Chapter 2: Production-Ready Backend Integration with Supabase

In this chapter, we’ll connect our Expo app to a **production**-ready Supabase backend. We’ll cover:

1. **Setting Up Environment Variables & Security**  
2. **Creating a Supabase Project** (in the dashboard)  
3. **Initializing Supabase Client** in a secure manner  
4. **Implementing User Sign-Up** flow with email confirmations  
5. **Implementing User Login** with session handling  
6. **Managing Session State & Protecting Routes**  

The emphasis here is on **best practices** so that secrets are not exposed in a public repository, while still allowing the project to remain open source.

---

## 1. Environment Variables & Security Best Practices

### 1.1 Why Environment Variables?

When building a **public open-source** project, it’s important not to hardcode API keys directly in your source code (especially if they are secrets). While Supabase has a concept of a **public `anon` key** (which is not strictly a secret), **service keys** (i.e., your service role key) should never be exposed to end users or in a public repository.  

**Key Points**:
- **anon (public)** key is intended for client-side use, but still should be configurable via environment variables for easy rotation.  
- **service_role** key must remain private and used only on trusted servers or serverless functions.  
- Do **not** commit `.env` files with real secrets to version control. Instead, use `.env.example` to show placeholders.  

### 1.2 Setting Up Environment Variables in Expo

Starting with Expo SDK 49, you can prefix environment variables with `EXPO_PUBLIC_` to safely expose them at **build time**. The recommended approach is:

1. Create a `.env` file (for local development):  
   ```ini
   EXPO_PUBLIC_SUPABASE_URL="https://<YOUR-PROJECT>.supabase.co"
   EXPO_PUBLIC_SUPABASE_ANON_KEY="<YOUR-ANON-KEY>"
   ```
2. **Never commit** this file. Add it to `.gitignore`:  
   ```
   # .gitignore
   .env
   ```
3. Create a `.env.example` file that only contains placeholders (safe to commit):  
   ```ini
   EXPO_PUBLIC_SUPABASE_URL="https://example.supabase.co"
   EXPO_PUBLIC_SUPABASE_ANON_KEY="anon-public-key"
   ```
4. In your `app.config.js` or `app.config.ts` (or `expo` field in `package.json`), you can reference these variables:
   ```js
   // app.config.js
   import 'dotenv/config';

   export default ({ config }) => {
     return {
       ...config,
       extra: {
         supabaseUrl: process.env.EXPO_PUBLIC_SUPABASE_URL,
         supabaseAnonKey: process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY,
       },
     };
   };
   ```
5. In your app, you can access them via `expo-constants` or `expo-router` param:
   ```ts
   // supabaseClient.ts
   import Constants from 'expo-constants';
   import { createClient } from '@supabase/supabase-js';
   import 'react-native-url-polyfill/auto';

   const supabaseUrl = Constants.expoConfig?.extra?.supabaseUrl || '';
   const supabaseAnonKey = Constants.expoConfig?.extra?.supabaseAnonKey || '';

   export const supabase = createClient(supabaseUrl, supabaseAnonKey);
   ```

**Why `EXPO_PUBLIC_`?**  
Environment variables in Expo that are used at **build time** can be automatically embedded into your JavaScript bundle. By prefixing them with `EXPO_PUBLIC_`, you acknowledge they will be exposed to the client, which is acceptable for the **anon** key but never for your service role key.

### 1.3 Secrets in CI/CD

In a typical CI/CD setup (e.g., GitHub Actions):
- Store environment variables (like `EXPO_PUBLIC_SUPABASE_URL`, `EXPO_PUBLIC_SUPABASE_ANON_KEY`) in your repository settings under **Secrets**.  
- During the build, your CI environment can inject these secrets.  
- **Do not** print the secrets to logs. If needed, mask them in your CI config.

---

## 2. Create & Configure Your Supabase Project

### 2.1 Create a New Supabase Project

1. Go to [Supabase.com](https://supabase.com/) and sign in (or create an account).  
2. Click **“New Project”** and fill out:
   - **Project Name**: e.g., `hidden-routes`  
   - **Database Password**: store it somewhere safe (e.g., a private vault).  
   - Choose a **region** close to your user base.  
3. Wait for provisioning to complete (a few minutes).

### 2.2 Retrieve Your Project Credentials

Inside your project’s **API** settings, you will see:
- **Project URL** (example: `https://<YOUR-PROJECT>.supabase.co`)  
- **Anon Key** (for client-side usage)  
- **Service Role Key** (must remain confidential)

Use only the **anon key** for your **frontend**. Keep the **service role key** off the client at all times (it’s only for server or serverless use).

---

## 3. Initialize the Supabase Client (Production Setup)

### 3.1 Secure Supabase Client

Create a `supabaseClient.ts` (or `.js`) in a location like `lib/supabaseClient.ts`. This file will:

1. Import the **URL polyfill** to ensure `URL` objects work in React Native.  
2. Load environment variables.  
3. Call `createClient(supabaseUrl, supabaseAnonKey)`.  

Example:

```ts
// lib/supabaseClient.ts
import 'react-native-url-polyfill/auto';
import { createClient } from '@supabase/supabase-js';
import Constants from 'expo-constants';

const supabaseUrl = Constants.expoConfig?.extra?.supabaseUrl;
const supabaseAnonKey = Constants.expoConfig?.extra?.supabaseAnonKey;

// Additional config can be provided here, e.g. headers or local storage options
// For advanced usage, see supabase-js docs
export const supabase = createClient(supabaseUrl, supabaseAnonKey);
```

### 3.2 Confirming RLS (Row-Level Security) & Auth Policies

For production, you should:
- Enable **Row-Level Security (RLS)** in your Supabase **Database** settings.  
- Write or apply policies so that only authenticated users can access certain tables.  
- For read/write operations, ensure your policies are carefully tested.

---

## 4. Implementing Secure Sign-Up

Your sign-up screen is located at `app/(auth)/sign-up.tsx`. For a **production** environment:

1. Validate user input (email format, password strength).  
2. Call Supabase’s `auth.signUp()` with a strong password policy in your Supabase Auth settings.  
3. (Optional) Provide reCAPTCHA or other anti-bot measures if you anticipate high traffic or spam.

**Example**:

```tsx
// app/(auth)/sign-up.tsx
import React from "react";
import { View, Text, TextInput, TouchableOpacity, Alert } from "react-native";
import { supabase } from "../../lib/supabaseClient";

export default function SignUp() {
  const [email, setEmail] = React.useState("");
  const [password, setPassword] = React.useState("");

  const handleSignUp = async () => {
    if (!email || !password) {
      Alert.alert("Validation Error", "Email and Password are required.");
      return;
    }

    try {
      const { data, error } = await supabase.auth.signUp({
        email,
        password,
      });

      if (error) {
        Alert.alert("Sign Up Error", error.message);
        return;
      }

      if (data.user) {
        Alert.alert(
          "Check Your Inbox",
          "A confirmation email has been sent. Please verify your account."
        );
      }
    } catch (err: any) {
      Alert.alert("Sign Up Error", err.message);
    }
  };

  return (
    <View className="container-steampunk bg-steampunk">
      <Text className="text-steampunk-title">Create Account</Text>

      <Text className="text-steampunk-label">Email</Text>
      <TextInput
        placeholder="Enter your email"
        placeholderTextColor="#A98274"
        className="input-steampunk mb-4"
        value={email}
        onChangeText={setEmail}
        keyboardType="email-address"
        autoCapitalize="none"
      />

      <Text className="text-steampunk-label">Password</Text>
      <TextInput
        placeholder="Create a password"
        placeholderTextColor="#A98274"
        secureTextEntry
        className="input-steampunk mb-4"
        value={password}
        onChangeText={setPassword}
        autoCapitalize="none"
      />

      <TouchableOpacity className="btn-steampunk mt-4" onPress={handleSignUp}>
        <Text className="btn-text-steampunk">Sign Up</Text>
      </TouchableOpacity>
    </View>
  );
}
```

### 4.1 Email Confirmations

In your Supabase **Auth Settings**:
- Turn on **Email Confirmations** so that a user must verify their email.  
- Adjust email templates for a professional look.  
- The user remains in a “waiting for verification” state until they confirm.

---

## 5. Implementing Login Flow

Your login screen at `app/(auth)/login.tsx` will:

1. Collect email + password.  
2. Call `auth.signInWithPassword()`.  
3. If successful, store the session in memory or a global context.  
4. Redirect the user to a **protected** route or homepage.

**Example**:

```tsx
// app/(auth)/login.tsx
import React from "react";
import { View, Text, TextInput, TouchableOpacity, Alert } from "react-native";
import { supabase } from "../../lib/supabaseClient";
import { useRouter } from "expo-router";

export default function Login() {
  const [email, setEmail] = React.useState("");
  const [password, setPassword] = React.useState("");
  const router = useRouter();

  const handleLogin = async () => {
    if (!email || !password) {
      Alert.alert("Validation Error", "Email and Password are required.");
      return;
    }

    try {
      const { data, error } = await supabase.auth.signInWithPassword({
        email,
        password,
      });

      if (error) {
        Alert.alert("Login Error", error.message);
      } else if (data.session) {
        // User successfully logged in; you have an access token in data.session
        router.push("/"); // or any protected route
      }
    } catch (err: any) {
      Alert.alert("Login Error", err.message);
    }
  };

  return (
    <View className="container-steampunk bg-steampunk">
      <Text className="text-steampunk-title">SteamPunk Login</Text>

      <Text className="text-steampunk-label">Email</Text>
      <TextInput
        placeholder="Enter your email"
        placeholderTextColor="#A98274"
        className="input-steampunk mb-4"
        value={email}
        onChangeText={setEmail}
        keyboardType="email-address"
        autoCapitalize="none"
      />

      <Text className="text-steampunk-label">Password</Text>
      <TextInput
        placeholder="Enter your password"
        placeholderTextColor="#A98274"
        secureTextEntry
        className="input-steampunk mb-6"
        value={password}
        onChangeText={setPassword}
        autoCapitalize="none"
      />

      <TouchableOpacity className="btn-steampunk" onPress={handleLogin}>
        <Text className="btn-text-steampunk">Login</Text>
      </TouchableOpacity>
    </View>
  );
}
```

---

## 6. Managing Session State & Protecting Routes

### 6.1 Real-Time Session Updates

Use `supabase.auth.onAuthStateChange` to listen for sign-in, sign-out, or token refresh events. Store the session in a global context (e.g., React Context, Redux, or Recoil) if you need to show/hide UI elements based on the user’s auth state.

```ts
// app/_layout.tsx
import React from "react";
import { Stack } from "expo-router";
import { supabase } from "../lib/supabaseClient";

export default function RootLayout() {
  React.useEffect(() => {
    const { data: authSubscription } = supabase.auth.onAuthStateChange(
      (event, session) => {
        console.log("[Auth Event]", event, session);
        // Update your global auth state here
      }
    );

    return () => {
      authSubscription.subscription.unsubscribe();
    };
  }, []);

  return <Stack />;
}
```

### 6.2 Protected Routes

If some screens should only be accessible to logged-in users, set up a protected layout. For example:

```tsx
// app/(protected)/_layout.tsx
import React, { useEffect, useState } from "react";
import { Stack, useRouter } from "expo-router";
import { supabase } from "../../lib/supabaseClient";
import { ActivityIndicator, View } from "react-native";

export default function ProtectedLayout() {
  const router = useRouter();
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Check current session
    supabase.auth.getSession().then(({ data: { session } }) => {
      if (!session) {
        router.replace("/(auth)/login");
      } else {
        setLoading(false);
      }
    });
  }, []);

  if (loading) {
    // Show a loading spinner
    return (
      <View className="flex-1 items-center justify-center bg-steampunk">
        <ActivityIndicator size="large" color="#EFBD5D" />
      </View>
    );
  }

  return <Stack />;
}
```

Any screens you place in `app/(protected)/` will require an active session. If no session is found, the user is redirected to login.

---

## 7. Final Considerations for Production

1. **Rotate Keys Regularly**: Although the `anon` key is public, you might still rotate it periodically or after a security event.  
2. **Strict RLS Policies**: Double-check that your database policies only allow data access to authorized, authenticated users.  
3. **Monitor Logs**: Use Supabase logs to watch for suspicious activity or repeated authentication failures.  
4. **Version & Documentation**: Since it’s an open-source project, maintain clear documentation on how contributors can run the app locally (using `.env.example`).  
5. **Automate with CI/CD**: Keep your secrets out of the codebase; load them from your CI environment (GitHub Actions, CircleCI, etc.) to inject environment variables at build time.

---

## Recap

You now have a **production-oriented** understanding of integrating Supabase into an Expo (React Native) project:

- **Environment Variables**: Use `.env` for local dev (ignored by Git), `.env.example` for placeholders, and `EXPO_PUBLIC_` prefixes for values you do intend to expose at runtime.  
- **Supabase Project Setup**: Keep your **service key** private. The **anon key** is used client-side, but treat it as an environment variable anyway.  
- **Authentication**: Implement sign-up, login, and real-time session handling with robust checks for email confirmations.  
- **Protected Routes**: Use **expo-router** layouts or any other router approach to guard pages with an auth gate.  
- **Security**: Enforce RLS in the Supabase Dashboard, rotate keys as needed, and keep secrets out of your repository.  

This foundation ensures your steampunk-themed app meets real-world standards while remaining open-source, inviting contributions without risking security.  

**Next up**: We’ll explore database schema design, hooking up more advanced queries/mutations, and possibly working with **Supabase Edge Functions** for custom server-side logic.  
