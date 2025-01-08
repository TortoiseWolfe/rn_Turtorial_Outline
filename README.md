## 0) One Block of Terminal Commands

```bash
# Step 0: Create & reset the project
npx create-expo-app DemoApp
cd DemoApp
npm run reset-project

# Step 1: Install all required dependencies
npx expo install nativewind tailwindcss react-native-reanimated react-native-safe-area-context \
  expo-router \
  @supabase/supabase-js \
  supertokens-react-native \
  react-hook-form \
  zod \
  @hookform/resolvers \
  expo-font \
  @expo-google-fonts/special-elite \
  @expo-google-fonts/arbutus-slab \
  react-native-dotenv \
  expo-constants \
  react-native-web \
  # The rest are installed automatically or come with expo

# Step 2: Recreate assets & placeholders
mkdir -p assets/images
touch assets/images/icon.png
touch assets/images/splash-icon.png
touch assets/images/adaptive-icon.png
touch assets/images/favicon.png

# Step 3: Create config files for tailwind, postcss, babel, metro
npx tailwindcss init
# touch tailwind.config.js
touch postcss.config.js
touch babel.config.js
npx expo customize metro.config.js

# Step 4: Create global.css for Tailwind
touch global.css

# Step 5: Create .env (real) & .env.example
touch .env .env.example

# Step 6: Create nativewind-env.d.ts for TS
touch nativewind-env.d.ts

# Step 7: Create lib & providers folders & files
mkdir lib
touch lib/supabaseClient.ts
touch lib/supertokensClient.ts

mkdir providers
touch providers/ThemeProvider.tsx

# Step 8: Create app folder structure
mkdir app
touch app/_layout.tsx
touch app/index.tsx
mkdir "app/(auth)"
touch "app/(auth)/login.tsx"
touch "app/(auth)/signup.tsx"
mkdir "app/(protected)"
touch "app/(protected)/_layout.tsx"
touch "app/(protected)/home.tsx"
touch "app/(protected)/settings.tsx"

# If there's an app-example folder left, you can remove it:
# rm -rf app-example
```

After running, you will have a **fresh** project with empty placeholders for assets, config, and all the screens we need.

---

## 1) Fill Each File with the Complete Code

Open **`DemoApp`** in VS Code (or your editor of choice). **Replace** each file’s contents below.

### **`app.json`** (Expo config with splash plugin & favicon)

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

- **`splash-icon.png`** used by **`expo-splash-screen`**.  
- **`favicon.png`** for web.  
- **Typed routes** enabled.

---

### **`tailwind.config.js`** (from NativeWind snippet)

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ["./app/**/*.{js,jsx,ts,tsx}"],
  presets: [require("nativewind/preset")],
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

*(We add a bit more for the steampunk theme.)*

---

### **`global.css`** (Tailwind directives)

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
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

### **`babel.config.js`** (SDK 50+ with `jsxImportSource: "nativewind"`)

```js
module.exports = function (api) {
  api.cache(true);
  return {
    presets: [
      [
        "babel-preset-expo",
        {
          jsxImportSource: "nativewind"
        }
      ],
      "nativewind/babel"
    ]
  };
};
```

---

### **`metro.config.js`** (Customize to load `global.css`)

Replace the newly generated **`metro.config.js`** with:

```js
// Learn more https://docs.expo.io/guides/customizing-metro
const { getDefaultConfig } = require('expo/metro-config');
const { withNativeWind } = require("nativewind/metro");

/** @type {import('expo/metro-config').MetroConfig} */
const config = getDefaultConfig(__dirname);

module.exports = withNativeWind(config, {
  input: "./global.css"
});
```

*(If your file is `.cjs` or `.mjs`, adapt accordingly.)*

---

### **`nativewind-env.d.ts`**

```ts
/// <reference types="nativewind/types" />
```

Ensures TypeScript recognizes `className`.

---

### **`.env.example`**

```
SUPABASE_URL=https://your-supabase-url.supabase.co
SUPABASE_ANON_KEY=YOUR_ANON_KEY
SUPERTOKENS_API_BASE_URL=https://your-supertokens-domain.com
```

*(Commit **this** to your repo, not the real .env!)*

### **`.env`**

```
SUPABASE_URL=YOUR_REAL_SUPABASE_URL
SUPABASE_ANON_KEY=YOUR_REAL_ANON_KEY
SUPERTOKENS_API_BASE_URL=YOUR_REAL_SUPERTOKENS_URL
```

*(**Don’t** commit this.)*

---

### **`lib/supabaseClient.ts`**

```ts
import { createClient } from "@supabase/supabase-js";
import { SUPABASE_URL, SUPABASE_ANON_KEY } from "@env";

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

### **`providers/ThemeProvider.tsx`** (Light/Dark)

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

### **`app/_layout.tsx`** (Root Layout, importing `global.css`, setting up SuperTokens, fonts)

```tsx
import "../global.css"; // 1) Import Tailwind's global CSS
import { Stack } from "expo-router";
import { useEffect } from "react";
import { initSuperTokens } from "../lib/supertokensClient";
import { ThemeProvider } from "../providers/ThemeProvider";

// Google fonts
import { useFonts } from "expo-font";
import { SpecialElite_400Regular } from "@expo-google-fonts/special-elite";
import { ArbutusSlab_400Regular } from "@expo-google-fonts/arbutus-slab";
import { View, Text } from "react-native";

export default function RootLayout() {
  // Load steampunk fonts
  const [fontsLoaded] = useFonts({
    SpecialElite_400Regular,
    ArbutusSlab_400Regular
  });

  // Initialize SuperTokens
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

*(The `import "../global.css";` is essential—this is how **NativeWind** loads your Tailwind CSS.)*

---

### **`app/index.tsx`** (Main entry screen)

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

---

### **`app/(auth)/login.tsx`** (React Hook Form + Zod)

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

  async function onSubmit({ email, password }: LoginFormData) {
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

      {/* Link to Sign Up */}
      <TouchableOpacity className="mt-4" onPress={() => router.push("/(auth)/signup")}>
        <Text className="text-steampunkLight font-steampunkTwo">No account? Sign Up</Text>
      </TouchableOpacity>
    </View>
  );
}
```

---

### **`app/(auth)/signup.tsx`** (React Hook Form + Zod)

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

  async function onSubmit({ email, password }: SignupFormData) {
    const { data: signupData, error } = await supabase.auth.signUp({
      email,
      password
    });
    if (error) {
      alert(error.message);
    } else {
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

### **`app/(protected)/_layout.tsx`** (Check if user is logged in)

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

### **`app/(protected)/settings.tsx`** (Light/Dark Toggle)

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

## 2) Run & Verify

From **`DemoApp`**:

```bash
npm start
```

- Press **`i`** for iOS (on macOS).  
- Press **`a`** for Android.  
- Press **`w`** for web (since we installed `react-native-web`).

Check:

1. **Splash**: You’ll see `splash-icon.png` on iOS/Android due to `expo-splash-screen` plugin.  
2. **Favicon**: On web, you have `favicon.png`.  
3. **No** Babel warning about `expo-router/babel`.  
4. **Tailwind**: Classes (e.g. `bg-steampunkDark`) should work.  
5. **NativeWind** pipeline: `global.css` is loaded in `_layout.tsx`, processed by `metro.config.js`.  
6. **Supabase** + **SuperTokens**: Log in, sign up, or check if user is protected in `(protected)/_layout.tsx`.  
7. **Form Validation** with zod & react-hook-form.  
8. **Light/Dark** theme toggle in settings, default dark.  
9. **Typed routes** thanks to `"experiments": { "typedRoutes": true }` in `app.json`.

You now have a **complete** tutorial that merges **all** the pieces we’ve been working on, including the newest **NativeWind** snippet for Expo SDK 50+, your steampunk theme, splash images, web favicon, Supabase, SuperTokens, environment variables, typed routes, and form validation. Enjoy your fully functional Expo Router app!
