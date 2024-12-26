# Setting Up Tailwind CSS with Nativewind in Expo

## Step 1: Initialize the Project and Install Dependencies
```bash

rm -rf MyHiddenRoutesApp
```

```bash
npx create-expo-app MyHiddenRoute -t
```

```bash
cd MyHiddenRoutesApp
```

```bash
npm run reset-project
```

```bash
npx expo start
```

```bash

npm install nativewind tailwindcss react-native-reanimated react-native-safe-area-context
```

```bash
npx tailwindcss init
```

```bash

touch global.css babel.config.js metro.config.js nativewind-env.d.ts
```

- Run `history` to review the commands used.
- Use `code .` to open the project in your editor.

## Step 2: Add Tailwind CSS Directives

Edit `global.css`:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

.viewBox {
  width: 400px;
  height: 400px;
  background-color: blue;
}
```

## Step 3: Configure Babel

Edit `babel.config.js`:

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

## Step 4: Configure Metro Bundler

Edit `metro.config.js`:

```javascript
const { getDefaultConfig } = require("expo/metro-config");
const { withNativeWind } = require("nativewind/metro");

const config = getDefaultConfig(__dirname);

module.exports = withNativeWind(config, { input: "./global.css" });
```

## Step 5: Initialize Tailwind Config

Edit `tailwind.config.js`:

```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
    content: ["./app/**/*.{js,jsx,ts,tsx}"],
    presets: [require("nativewind/preset")],
    theme: {
      extend: {},
    },
    plugins: [],
};
```

## Step 6: Create a React Native Component

Edit `index.tsx`:

```javascript
import { Text, View } from "react-native";
import { SafeAreaView } from "react-native-safe-area-context";
export default function Index() {
  return (
    <SafeAreaView>
      <View>
        <Text className="text-2xl text-red-600">Nativewind</Text>
        <Text className="text-3xl">Hello</Text>
      </View>

      <View className="viewBox" />

    </SafeAreaView>
  );
}
```

## Step 7: (auth) & (onboarding)

```bash
# Create the "(auth)" folder with its own layout and routes
mkdir -p "app/(auth)" && \
touch "app/(auth)/_layout.tsx" \
      "app/(auth)/login.tsx" \
      "app/(auth)/sign-up.tsx" \
      "app/(auth)/verification.tsx" \
      "app/(auth)/forgot-password.tsx" && \

# Create the "(onboarding)" folder with its own layout and routes
mkdir -p "app/(onboarding)" && \
touch "app/(onboarding)/_layout.tsx" \
      "app/(onboarding)/splash.tsx" \
      "app/(onboarding)/welcome.tsx" \
      "app/(onboarding)/onboarding-setup.tsx"

```

Below is an example of how you might structure each of the **7 screens** (files) with a simple “steam-punk” inspired design using Nativewind/Tailwind classes. Each screen uses a dark background, warm metallic accent colors (like ambers, browns, coppers), and some minimal stylings. Adjust as needed to fit your exact styling preferences.

> **Note**: These examples assume you are using **TypeScript** (hence `.tsx` extensions) and have configured Nativewind as described in your previous steps.

---

## 1. `app/(auth)/login.tsx`

```tsx
import React from "react";
import { View, Text, TextInput, TouchableOpacity } from "react-native";

export default function Login() {
  return (
    <View className="flex-1 bg-neutral-900 p-6 justify-center">
      <Text className="text-3xl text-amber-400 font-bold mb-6">SteamPunk Login</Text>

      <Text className="text-amber-200 mb-2">Username</Text>
      <TextInput
        placeholder="Enter your username"
        placeholderTextColor="#A98274"
        className="w-full bg-neutral-800 text-amber-100 p-3 rounded-md mb-4"
      />

      <Text className="text-amber-200 mb-2">Password</Text>
      <TextInput
        placeholder="Enter your password"
        placeholderTextColor="#A98274"
        secureTextEntry
        className="w-full bg-neutral-800 text-amber-100 p-3 rounded-md mb-6"
      />

      <TouchableOpacity className="bg-amber-700 rounded-md p-3">
        <Text className="text-center text-amber-50 font-semibold">Login</Text>
      </TouchableOpacity>
    </View>
  );
}
```

---

## 2. `app/(auth)/sign-up.tsx`

```tsx
import React from "react";
import { View, Text, TextInput, TouchableOpacity } from "react-native";

export default function SignUp() {
  return (
    <View className="flex-1 bg-neutral-900 p-6 justify-center">
      <Text className="text-3xl text-amber-400 font-bold mb-6">SteamPunk Sign Up</Text>

      <Text className="text-amber-200 mb-2">Email</Text>
      <TextInput
        placeholder="Enter your email"
        placeholderTextColor="#A98274"
        className="w-full bg-neutral-800 text-amber-100 p-3 rounded-md mb-4"
      />

      <Text className="text-amber-200 mb-2">Password</Text>
      <TextInput
        placeholder="Create a password"
        placeholderTextColor="#A98274"
        secureTextEntry
        className="w-full bg-neutral-800 text-amber-100 p-3 rounded-md mb-4"
      />

      <Text className="text-amber-200 mb-2">Confirm Password</Text>
      <TextInput
        placeholder="Repeat your password"
        placeholderTextColor="#A98274"
        secureTextEntry
        className="w-full bg-neutral-800 text-amber-100 p-3 rounded-md mb-6"
      />

      <TouchableOpacity className="bg-amber-700 rounded-md p-3">
        <Text className="text-center text-amber-50 font-semibold">Sign Up</Text>
      </TouchableOpacity>
    </View>
  );
}
```

---

## 3. `app/(auth)/verification.tsx`

```tsx
import React from "react";
import { View, Text, TextInput, TouchableOpacity } from "react-native";

export default function Verification() {
  return (
    <View className="flex-1 bg-neutral-900 p-6 justify-center">
      <Text className="text-3xl text-amber-400 font-bold mb-6">Email Verification</Text>

      <Text className="text-amber-200 mb-2">Verification Code</Text>
      <TextInput
        placeholder="Enter your 6-digit code"
        placeholderTextColor="#A98274"
        keyboardType="numeric"
        className="w-full bg-neutral-800 text-amber-100 p-3 rounded-md mb-6"
      />

      <TouchableOpacity className="bg-amber-700 rounded-md p-3">
        <Text className="text-center text-amber-50 font-semibold">Verify</Text>
      </TouchableOpacity>
    </View>
  );
}
```

---

## 4. `app/(auth)/forgot-password.tsx`

```tsx
import React from "react";
import { View, Text, TextInput, TouchableOpacity } from "react-native";

export default function ForgotPassword() {
  return (
    <View className="flex-1 bg-neutral-900 p-6 justify-center">
      <Text className="text-3xl text-amber-400 font-bold mb-6">Forgot Password</Text>

      <Text className="text-amber-200 mb-2">Email</Text>
      <TextInput
        placeholder="Enter your email"
        placeholderTextColor="#A98274"
        className="w-full bg-neutral-800 text-amber-100 p-3 rounded-md mb-6"
      />

      <TouchableOpacity className="bg-amber-700 rounded-md p-3">
        <Text className="text-center text-amber-50 font-semibold">Reset Password</Text>
      </TouchableOpacity>
    </View>
  );
}
```

---

## 5. `app/(onboarding)/splash.tsx`

```tsx
import React from "react";
import { View, Text, ImageBackground } from "react-native";

export default function Splash() {
  return (
    <View className="flex-1">
      <ImageBackground
        source={{ uri: "https://images.unsplash.com/photo-1506371224150-4ca7cc20fd48" }}
        resizeMode="cover"
        className="flex-1 justify-center items-center"
      >
        <Text className="text-4xl text-amber-300 font-bold bg-neutral-800/60 p-4 rounded-lg">
          Welcome to SteamPunk World
        </Text>
      </ImageBackground>
    </View>
  );
}
```

> **Tip**: You can replace the `uri` in the `ImageBackground` with a steampunk-themed image you like, or use a local asset.

---

## 6. `app/(onboarding)/welcome.tsx`

```tsx
import React from "react";
import { View, Text, TouchableOpacity } from "react-native";

export default function Welcome() {
  return (
    <View className="flex-1 bg-neutral-900 p-6 justify-center items-center">
      <Text className="text-4xl text-amber-400 font-bold mb-4">Welcome Aboard!</Text>
      <Text className="text-amber-200 text-center mb-10">
        Embark on a steam-powered adventure through our app.
      </Text>

      <TouchableOpacity className="bg-amber-700 rounded-md p-4 w-full max-w-sm">
        <Text className="text-center text-amber-50 font-semibold">Continue</Text>
      </TouchableOpacity>
    </View>
  );
}
```

---

## 7. `app/(onboarding)/onboarding-setup.tsx`

```tsx
import React from "react";
import { View, Text, TouchableOpacity, Switch } from "react-native";

export default function OnboardingSetup() {
  const [enableNotifications, setEnableNotifications] = React.useState(false);

  return (
    <View className="flex-1 bg-neutral-900 p-6">
      <Text className="text-3xl text-amber-400 font-bold mb-6">Onboarding Setup</Text>

      <View className="flex-row items-center justify-between mb-6">
        <Text className="text-amber-200">Enable Notifications</Text>
        <Switch
          value={enableNotifications}
          onValueChange={setEnableNotifications}
          thumbColor={enableNotifications ? "#EFBD5D" : "#A98274"}
          trackColor={{ true: "#7B5530", false: "#374151" }}
        />
      </View>

      <TouchableOpacity className="bg-amber-700 rounded-md p-3">
        <Text className="text-center text-amber-50 font-semibold">Finish Setup</Text>
      </TouchableOpacity>
    </View>
  );
}
```

---

### Customizing Further

- Adjust the **colors** (`bg-neutral-900`, `text-amber-50`, `text-amber-400`, etc.) for different steampunk vibes.
- Modify **fonts**, **images**, **icons**, or **background patterns** to enhance the steam-punk feel.
- Incorporate **gear**, **brass textures**, or **vintage fonts** as you see fit.
- Replace `ImageBackground` sources with your own local or remote images that reflect a steampunk theme.

With these 7 screens in place, you have a basic framework for your “steam-punk” styled onboarding and auth flow using **Nativewind** + **Tailwind CSS** in Expo!
