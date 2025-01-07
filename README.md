## **0. Single Code Block of Terminal Commands**

**Copy/paste** the entire block **as is**:

```bash
# Step 0: Start a fresh Expo project
npx create-expo-app DemoApp
cd DemoApp

# Step 1: Reset the project (removes sample assets/layout)
npm run reset-project

# Step 2: Recreate assets/images and placeholder icons/favicons
mkdir -p assets/images
touch assets/images/icon.png
touch assets/images/splash-icon.png
touch assets/images/adaptive-icon.png
touch assets/images/favicon.png

# Step 3: Recreate essential config & declaration files
touch app.json
touch babel.config.js
touch tailwind.config.js
touch postcss.config.js
touch nativewind-env.d.ts

# Step 4: Recreate .env files (example & real)
touch .env.example
touch .env

# Step 5: Recreate lib folder & TypeScript files
mkdir lib
touch lib/supabaseClient.ts
touch lib/supertokensClient.ts

# Step 6: Recreate providers folder & ThemeProvider file
mkdir providers
touch providers/ThemeProvider.tsx

# Step 7: Recreate the app folder structure
mkdir app
mkdir "app/(auth)"
mkdir "app/(protected)"

# Optionally remove any "app-example" folder or old sample files if you want:
# rm -rf app-example

# Step 8: Recreate placeholders for Expo Router
touch "app/_layout.tsx"
touch "app/index.tsx"
touch "app/(auth)/login.tsx"
touch "app/(auth)/signup.tsx"
touch "app/(protected)/_layout.tsx"
touch "app/(protected)/home.tsx"
touch "app/(protected)/settings.tsx"
```

Once you run these, your **DemoApp** project will have:

- **`assets/images/`** containing empty placeholder `.png` files (to be replaced with real images).  
- **`app.json`**, `babel.config.js`, `tailwind.config.js`, `postcss.config.js`, `nativewind-env.d.ts` as empty files.  
- **`.env`** and **`.env.example`** as empty placeholders.  
- **`lib/`** + **`providers/`** directories with placeholder `.ts`/`.tsx` files.  
- **`app/`** folder with `_layout.tsx`, `index.tsx`, `(auth)/` screens, `(protected)/` screens.  

Next, we’ll **fill** these files with the content needed for:

- A **Splash** screen (`splash-icon.png`).
- A **Favicon** for web (`favicon.png`).
- **Expo Router** with typed routes.
- **NativeWind** (Tailwind) in TypeScript.
- **Supabase** + **SuperTokens**.
- **Form Validation**.
- A **light/dark** theme toggle.

---

## **1. File Contents**

Open **VS Code** (or your preferred editor) in **`DemoApp`**. **Replace** the contents of each file with **these** blocks.

### **`app.json`**

```json5
{
  "expo": {
    "name": "DemoApp",
    "slug": "DemoApp",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/images/icon.png",
    "scheme": "myapp",
    "userInterfaceStyle": "automatic",
    "newArchEnabled": true,

    "ios": {
      "supportsTablet": true
    },

    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/images/adaptive-icon.png",
        "backgroundColor": "#ffffff"
      }
    },

    "web": {
      "bundler": "metro",
      "output": "static",
      "favicon": "./assets/images/favicon.png"
    },

    "plugins": [
      "expo-router",
      [
        "expo-splash-screen",
        {
          "image": "./assets/images/splash-icon.png",
          "imageWidth": 200,
          "resizeMode": "contain",
          "backgroundColor": "#ffffff"
        }
      ]
    ],

    "experiments": {
      "typedRoutes": true
    }
  }
}
```

**Notes**:  
- We reference **`splash-icon.png`** in `plugins -> expo-splash-screen`.  
- We reference **`favicon.png`** in `web.favicon`.  
- Adjust or replace with **real images** in `assets/images/`.

---

### **`babel.config.js`**

```js
module.exports = function (api) {
  api.cache(true);
  return {
    presets: ["babel-preset-expo"],
    plugins: [
      [
        "module:react-native-dotenv",
        {
          moduleName: "@env",
          path: ".env"
        }
      ],
      // Removed "expo-router/babel" for Expo SDK 50
      "nativewind/babel"
    ]
  };
};
```

---

### **`tailwind.config.js`**

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./App.{js,jsx,ts,tsx}",
    "./app/**/*.{js,jsx,ts,tsx}"
  ],
  theme: {
    extend: {
      fontFamily: {
        steampunkOne: ["SpecialElite_400Regular"],
        steampunkTwo: ["ArbutusSlab_400Regular"]
      },
      colors: {
        steampunkDark: "#2e2a2a",
        steampunkLight: "#f5f2f0",
        steampunkAccent: "#c19a6b"
      }
    }
  },
  plugins: []
};
```

---

### **`postcss.config.js`**

```js
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {}
  }
};
```

---

### **`nativewind-env.d.ts`**

```ts
/// <reference types="nativewind/types" />
```

*(Ensures TypeScript recognizes `className` on RN components.)*

---

### **`.env.example`**

```
SUPABASE_URL=https://your-supabase-url.supabase.co
SUPABASE_ANON_KEY=YOUR_ANON_KEY
SUPERTOKENS_API_BASE_URL=https://your-supertokens-domain.com
```

### **`.env`**

```
SUPABASE_URL=YOUR_REAL_SUPABASE_URL
SUPABASE_ANON_KEY=YOUR_REAL_ANON_KEY
SUPERTOKENS_API_BASE_URL=YOUR_REAL_SUPERTOKENS_URL
```

*(**Do not** commit `.env`—only `.env.example`.)*

---

### **`lib/supabaseClient.ts`**

```ts
import { createClient } from "@supabase/supabase-js";
import { SUPABASE_URL, SUPABASE_ANON_KEY } from "@env";

// official library: '@supabase/supabase-js'
export const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
```

---

### **`lib/supertokensClient.ts`**

```ts
import SuperTokens from "supertokens-react-native";
import { SUPERTOKENS_API_BASE_URL } from "@env";

export async function initSuperTokens() {
  await SuperTokens.init({
    apiDomain: SUPERTOKENS_API_BASE_URL,
    apiBasePath: "/auth"
  });
}

export async function signOut() {
  await SuperTokens.signOut();
}
```

---

### **`providers/ThemeProvider.tsx`**

```tsx
import React, { createContext, useContext, useState } from "react";

interface ThemeContextProps {
  theme: "light" | "dark";
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextProps>({
  theme: "dark",
  toggleTheme: () => {}
});

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  // Default to dark
  const [theme, setTheme] = useState<"light" | "dark">("dark");

  const toggleTheme = () => {
    setTheme(prev => (prev === "light" ? "dark" : "light"));
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

export const useThemeContext = () => useContext(ThemeContext);
```

---

### **`app/_layout.tsx`**

```tsx
import { Stack } from "expo-router";
import { useEffect } from "react";
import { initSuperTokens } from "../lib/supertokensClient";
import { ThemeProvider } from "../providers/ThemeProvider";

import { useFonts } from "expo-font";
import { SpecialElite_400Regular } from "@expo-google-fonts/special-elite";
import { ArbutusSlab_400Regular } from "@expo-google-fonts/arbutus-slab";
import { View, Text } from "react-native";

export default function RootLayout() {
  // Load google fonts
  const [fontsLoaded] = useFonts({
    SpecialElite_400Regular,
    ArbutusSlab_400Regular
  });

  // Init SuperTokens
  useEffect(() => {
    initSuperTokens();
  }, []);

  if (!fontsLoaded) {
    return (
      <View className="flex-1 items-center justify-center bg-steampunkDark">
        <Text className="text-steampunkAccent font-steampunkOne">Loading steampunk gears...</Text>
      </View>
    );
  }

  return (
    <ThemeProvider>
      <Stack screenOptions={{ headerShown: false }} />
    </ThemeProvider>
  );
}
```

---

### **`app/index.tsx`**

```tsx
import React from "react";
import { View, Text } from "react-native";
import { Link } from "expo-router";

export default function IndexScreen() {
  return (
    <View className="flex-1 items-center justify-center bg-steampunkDark">
      <Text className="text-steampunkAccent font-steampunkOne text-3xl mb-6">Welcome to DemoApp</Text>
      <Link href="/(auth)/login" className="bg-steampunkAccent p-3 rounded">
        <Text className="text-steampunkDark font-steampunkTwo">Go to Login</Text>
      </Link>
    </View>
  );
}
```

*(This is your main entry screen. Adjust as desired.)*

---

### **`app/(auth)/login.tsx`**

```tsx
import React from "react";
import { View, Text, TextInput, TouchableOpacity } from "react-native";
import { useRouter } from "expo-router";
import { useForm, Controller } from "react-hook-form";
import { z } from "zod";
import { zodResolver } from "@hookform/resolvers/zod";
import { supabase } from "../../lib/supabaseClient";

const loginSchema = z.object({
  email: z.string().email("Invalid email"),
  password: z.string().min(6, "At least 6 chars")
});

type LoginFormData = z.infer<typeof loginSchema>;

export default function LoginScreen() {
  const router = useRouter();
  const {
    control,
    handleSubmit,
    formState: { errors }
  } = useForm<LoginFormData>({ resolver: zodResolver(loginSchema) });

  async function onSubmit(data: LoginFormData) {
    const { email, password } = data;
    const { data: authData, error } = await supabase.auth.signInWithPassword({
      email,
      password
    });
    if (error) {
      alert(error.message);
    } else {
      router.replace("/(protected)/home");
    }
  }

  return (
    <View className="flex-1 items-center justify-center bg-steampunkDark px-4">
      <Text className="text-steampunkAccent font-steampunkOne text-3xl mb-6">Steam Login</Text>

      {/* Email */}
      <View className="w-full mb-4">
        <Text className="text-steampunkLight font-steampunkTwo mb-1">Email</Text>
        <Controller
          control={control}
          name="email"
          render={({ field: { onBlur, onChange, value } }) => (
            <TextInput
              className="bg-steampunkLight text-black p-2 rounded"
              onBlur={onBlur}
              onChangeText={onChange}
              value={value}
              placeholder="Enter email"
              autoCapitalize="none"
              keyboardType="email-address"
            />
          )}
        />
        {errors.email && <Text className="text-red-500">{errors.email.message}</Text>}
      </View>

      {/* Password */}
      <View className="w-full mb-4">
        <Text className="text-steampunkLight font-steampunkTwo mb-1">Password</Text>
        <Controller
          control={control}
          name="password"
          render={({ field: { onBlur, onChange, value } }) => (
            <TextInput
              className="bg-steampunkLight text-black p-2 rounded"
              onBlur={onBlur}
              onChangeText={onChange}
              value={value}
              placeholder="Enter password"
              secureTextEntry
            />
          )}
        />
        {errors.password && <Text className="text-red-500">{errors.password.message}</Text>}
      </View>

      {/* Submit */}
      <TouchableOpacity className="bg-steampunkAccent p-3 rounded" onPress={handleSubmit(onSubmit)}>
        <Text className="text-steampunkDark font-steampunkTwo">Log In</Text>
      </TouchableOpacity>

      {/* Link to Signup */}
      <TouchableOpacity className="mt-4" onPress={() => router.push("/(auth)/signup")}>
        <Text className="text-steampunkLight font-steampunkTwo">No account? Sign Up</Text>
      </TouchableOpacity>
    </View>
  );
}
```

---

### **`app/(auth)/signup.tsx`**

```tsx
import React from "react";
import { View, Text, TextInput, TouchableOpacity } from "react-native";
import { useRouter } from "expo-router";
import { useForm, Controller } from "react-hook-form";
import { z } from "zod";
import { zodResolver } from "@hookform/resolvers/zod";
import { supabase } from "../../lib/supabaseClient";

const signupSchema = z.object({
  email: z.string().email("Invalid email"),
  password: z.string().min(6, "At least 6 chars")
});

type SignupFormData = z.infer<typeof signupSchema>;

export default function SignupScreen() {
  const router = useRouter();
  const {
    control,
    handleSubmit,
    formState: { errors }
  } = useForm<SignupFormData>({ resolver: zodResolver(signupSchema) });

  async function onSubmit(data: SignupFormData) {
    const { email, password } = data;
    const { data: signupData, error } = await supabase.auth.signUp({
      email,
      password
    });
    if (error) {
      alert(error.message);
    } else {
      // after signup, go to login
      router.replace("/(auth)/login");
    }
  }

  return (
    <View className="flex-1 items-center justify-center bg-steampunkDark px-4">
      <Text className="text-steampunkAccent font-steampunkOne text-3xl mb-6">Join the Steam</Text>

      {/* Email */}
      <View className="w-full mb-4">
        <Text className="text-steampunkLight font-steampunkTwo mb-1">Email</Text>
        <Controller
          control={control}
          name="email"
          render={({ field: { onBlur, onChange, value } }) => (
            <TextInput
              className="bg-steampunkLight text-black p-2 rounded"
              onBlur={onBlur}
              onChangeText={onChange}
              value={value}
              placeholder="Enter email"
              autoCapitalize="none"
            />
          )}
        />
        {errors.email && <Text className="text-red-500">{errors.email.message}</Text>}
      </View>

      {/* Password */}
      <View className="w-full mb-4">
        <Text className="text-steampunkLight font-steampunkTwo mb-1">Password</Text>
        <Controller
          control={control}
          name="password"
          render={({ field: { onBlur, onChange, value } }) => (
            <TextInput
              className="bg-steampunkLight text-black p-2 rounded"
              onBlur={onBlur}
              onChangeText={onChange}
              value={value}
              placeholder="Enter password"
              secureTextEntry
            />
          )}
        />
        {errors.password && <Text className="text-red-500">{errors.password.message}</Text>}
      </View>

      {/* Submit */}
      <TouchableOpacity className="bg-steampunkAccent p-3 rounded" onPress={handleSubmit(onSubmit)}>
        <Text className="text-steampunkDark font-steampunkTwo">Sign Up</Text>
      </TouchableOpacity>

      {/* Link to Login */}
      <TouchableOpacity className="mt-4" onPress={() => router.replace("/(auth)/login")}>
        <Text className="text-steampunkLight font-steampunkTwo">Have an account? Login</Text>
      </TouchableOpacity>
    </View>
  );
}
```

---

### **`app/(protected)/_layout.tsx`**

```tsx
import React, { useEffect, useState } from "react";
import { Stack, useRouter } from "expo-router";
import { supabase } from "../../lib/supabaseClient";
import { View, Text } from "react-native";

export default function ProtectedLayout() {
  const router = useRouter();
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function checkAuth() {
      const { data } = await supabase.auth.getUser();
      if (!data?.user) {
        router.replace("/(auth)/login");
      } else {
        setLoading(false);
      }
    }
    checkAuth();
  }, []);

  if (loading) {
    return (
      <View className="flex-1 items-center justify-center bg-steampunkDark">
        <Text className="text-steampunkAccent font-steampunkOne">Checking your steam credentials...</Text>
      </View>
    );
  }

  return <Stack />;
}
```

---

### **`app/(protected)/home.tsx`**

```tsx
import React from "react";
import { View, Text, TouchableOpacity } from "react-native";
import { supabase } from "../../lib/supabaseClient";
import { useRouter } from "expo-router";

export default function HomeScreen() {
  const router = useRouter();

  async function handleLogout() {
    await supabase.auth.signOut();
    router.replace("/(auth)/login");
  }

  return (
    <View className="flex-1 items-center justify-center bg-steampunkDark">
      <Text className="text-steampunkAccent font-steampunkOne text-4xl mb-4">Home</Text>
      <TouchableOpacity
        className="bg-steampunkAccent p-3 rounded mb-4"
        onPress={() => router.push("/(protected)/settings")}
      >
        <Text className="text-steampunkDark font-steampunkTwo">Settings</Text>
      </TouchableOpacity>

      <TouchableOpacity className="bg-red-500 p-3 rounded" onPress={handleLogout}>
        <Text className="text-white font-steampunkTwo">Logout</Text>
      </TouchableOpacity>
    </View>
  );
}
```

---

### **`app/(protected)/settings.tsx`**

```tsx
import React from "react";
import { View, Text, Switch } from "react-native";
import { useThemeContext } from "../../providers/ThemeProvider";

export default function SettingsScreen() {
  const { theme, toggleTheme } = useThemeContext();

  return (
    <View
      className={`flex-1 items-center justify-center ${
        theme === "dark" ? "bg-steampunkDark" : "bg-steampunkLight"
      }`}
    >
      <Text
        className={`font-steampunkOne text-4xl mb-4 ${
          theme === "dark" ? "text-steampunkAccent" : "text-steampunkDark"
        }`}
      >
        Settings
      </Text>
      <View className="flex-row items-center space-x-2">
        <Text
          className={`font-steampunkTwo ${
            theme === "dark" ? "text-steampunkLight" : "text-steampunkDark"
          }`}
        >
          Light / Dark
        </Text>
        <Switch value={theme === "dark"} onValueChange={toggleTheme} />
      </View>
    </View>
  );
}
```

---

## **2. Final: Run & Check**

1. **Put real keys** in **`.env`** (not committed).
2. **Replace** placeholders in `assets/images/` with real PNG files for:  
   - `icon.png`  
   - `splash-icon.png`  
   - `adaptive-icon.png`  
   - `favicon.png`  
3. **Run**:  
   ```bash
   npm start
   ```
4. Press:
   - **`i`** for iOS (on macOS + Xcode)
   - **`a`** for Android
   - **`w`** for web
5. You’ll see:
   - **Splash** from `splash-icon.png` (via `expo-splash-screen`).
   - A **favicon** on web from `favicon.png`.
   - **Steampunk** colors + fonts (Special Elite, Arbutus Slab).
   - **Auth** screens if you go to `(auth)/login`.
   - **Protected** screens if you go to `(protected)/home`, checking `supabase.auth.getUser()`.
   - **Light/dark** toggle in **Settings**.
   - **No** Babel warnings about `expo-router/babel`.
   - **No** red squiggles for `className` thanks to `nativewind-env.d.ts`.

**Done!** You now have a **fresh** post-`reset-project` tutorial that:

- Recreates the missing assets (splash, favicon, icons).  
- Sets up **Expo Router** (with typed routes).  
- Integrates **NativeWind** (Tailwind) in TypeScript.  
- Configures **Supabase** + **SuperTokens**.  
- Provides **form validation** & **light/dark** theme.  

Enjoy your **steampunk**-themed Expo app on **SDK 50**—**no** corners cut!
