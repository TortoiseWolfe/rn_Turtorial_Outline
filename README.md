# Setting Up Tailwind CSS with Nativewind in Expo

## Step 1: Initialize the Project and Install Dependencies
```bash

rm -rf MyHiddenRoutesApp
```

```bash
npx create-expo-app MyHiddenRoutesApp -t
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

touch global.css babel.config.js metro.config.js nativewind-env.d.ts +not-found.tsx
```

- Run `history` to review the commands used.
- Use `code .` to open the project in your editor.

## Step 2: Add Tailwind CSS Directives

Edit `global.css`:

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
    @apply text-3xl text-amber-400 font-bold mb-6;
  }

  .text-steampunk-label {
    @apply text-amber-200 mb-2;
  }

  .text-steampunk-body {
    @apply text-amber-200;
  }

  /* Inputs */
  .input-steampunk {
    @apply w-full bg-neutral-800 text-amber-100 p-3 rounded-md;
  }

  /* Buttons */
  .btn-steampunk {
    @apply bg-amber-700 rounded-md p-3;
  }

  .btn-text-steampunk {
    @apply text-center text-amber-50 font-semibold;
  }
}

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
import { Link } from "expo-router"; // Ensure expo-router is installed

export default function Index() {
  return (
    <SafeAreaView className="container-steampunk bg-steampunk">
      <View>
        <Text className="text-steampunk-title">SteamPunk App</Text>
        <Text className="text-steampunk-body">
          Welcome to your steampunk-themed application powered by Nativewind and Tailwind CSS.
        </Text>
      </View>

      {/* Navigation Links */}
      <View className="mt-8">
        <Link href="/(auth)/login" className="link-steampunk mb-4 block">
          Login
        </Link>
        <Link href="/(onboarding)/welcome" className="link-steampunk">
          Get Started
        </Link>
      </View>

      {/* Example Box */}
      <View className="mt-8 border-steampunk">
        <Text className="text-steampunk-body">
          This is a sample box demonstrating centralized steampunk styles.
        </Text>
      </View>
    </SafeAreaView>
  );
}


```


Edit `+not-found.tsx`:

```javascript
import { Link, Stack } from 'expo-router';
import { StyleSheet } from 'react-native';

import { ThemedText } from '@/components/ThemedText';
import { ThemedView } from '@/components/ThemedView';

export default function NotFoundScreen() {
  return (
    <>
      <Stack.Screen options={{ title: 'Oops!' }} />
      <ThemedView style={styles.container}>
        <ThemedText type="title">This screen doesn't exist.</ThemedText>
        <Link href="/" style={styles.link}>
          <ThemedText type="link">Go to home screen!</ThemedText>
        </Link>
      </ThemedView>
    </>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
    padding: 20,
  },
  link: {
    marginTop: 15,
    paddingVertical: 15,
  },
});
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
Below is an example of how you can **centralize** your steampunk styling in **`global.css`**, and then apply these utility classes within each of the **7 screens**. This way, you avoid repeating the same styling across files, and any future theming tweaks can be done in one place.

---

# 1) Define Steampunk Classes in `global.css`

Inside your `global.css` (which already has `@tailwind base; @tailwind components; @tailwind utilities;`), add a custom layer for your steampunk components/utilities. For example:

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
    @apply text-3xl text-amber-400 font-bold mb-6;
  }

  .text-steampunk-label {
    @apply text-amber-200 mb-2;
  }

  .text-steampunk-body {
    @apply text-amber-200;
  }

  /* Inputs */
  .input-steampunk {
    @apply w-full bg-neutral-800 text-amber-100 p-3 rounded-md;
  }

  /* Buttons */
  .btn-steampunk {
    @apply bg-amber-700 rounded-md p-3;
  }

  .btn-text-steampunk {
    @apply text-center text-amber-50 font-semibold;
  }
}
```

> Feel free to adjust the classes, colors, spacing, etc. to refine your steampunk aesthetic.

---

# 2) Apply the Classes in Each Screen

Below are minimal examples of how you would **replace** the inline Tailwind classes with your new **global** classes.

## **(auth) Screens**

### `app/(auth)_layout`

```tsx


```
### `app/(auth)/login.tsx`

```tsx
import React from "react";
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

---

### `app/(auth)/sign-up.tsx`

```tsx
import React from "react";
import { View, Text, TextInput, TouchableOpacity } from "react-native";

export default function SignUp() {
  return (
    <View className="container-steampunk bg-steampunk">
      <Text className="text-steampunk-title">SteamPunk Sign Up</Text>

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

---

### `app/(auth)/verification.tsx`

```tsx
import React from "react";
import { View, Text, TextInput, TouchableOpacity } from "react-native";

export default function Verification() {
  return (
    <View className="container-steampunk bg-steampunk">
      <Text className="text-steampunk-title">Email Verification</Text>

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

---

### `app/(auth)/forgot-password.tsx`

```tsx
import React from "react";
import { View, Text, TextInput, TouchableOpacity } from "react-native";

export default function ForgotPassword() {
  return (
    <View className="container-steampunk bg-steampunk">
      <Text className="text-steampunk-title">Forgot Password</Text>

      <Text className="text-steampunk-label">Email</Text>
      <TextInput
        placeholder="Enter your email"
        placeholderTextColor="#A98274"
        className="input-steampunk mb-6"
      />

      <TouchableOpacity className="btn-steampunk">
        <Text className="btn-text-steampunk">Reset Password</Text>
      </TouchableOpacity>
    </View>
  );
}
```

---

## **(onboarding) Screens**

### `app/(onboarding)_layout`

```tsx


```

### `app/(onboarding)/splash.tsx`

```tsx
import React from "react";
import { View, Text, ImageBackground } from "react-native";

export default function Splash() {
  return (
    <View className="flex-1">
      <ImageBackground
        source={{
          uri: "https://images.unsplash.com/photo-1506371224150-4ca7cc20fd48",
        }}
        resizeMode="cover"
        className="flex-1 justify-center items-center"
      >
        <Text className="text-steampunk-title bg-neutral-800/60 p-4 rounded-lg">
          Welcome to SteamPunk World
        </Text>
      </ImageBackground>
    </View>
  );
}
```

> **Tip**: You don’t necessarily need to apply `bg-steampunk` on an `ImageBackground` if you are covering the entire screen with an image. You can still integrate other classes from the global file if desired.

---

### `app/(onboarding)/welcome.tsx`

```tsx
import React from "react";
import { View, Text, TouchableOpacity } from "react-native";

export default function Welcome() {
  return (
    <View className="container-steampunk bg-steampunk items-center">
      <Text className="text-steampunk-title">Welcome Aboard!</Text>
      <Text className="text-steampunk-body text-center mb-10">
        Embark on a steam-powered adventure through our app.
      </Text>

      <TouchableOpacity className="btn-steampunk w-full max-w-sm">
        <Text className="btn-text-steampunk">Continue</Text>
      </TouchableOpacity>
    </View>
  );
}
```

---

### `app/(onboarding)/onboarding-setup.tsx`

```tsx
import React from "react";
import { View, Text, TouchableOpacity, Switch } from "react-native";

export default function OnboardingSetup() {
  const [enableNotifications, setEnableNotifications] = React.useState(false);

  return (
    <View className="container-steampunk bg-steampunk">
      <Text className="text-steampunk-title">Onboarding Setup</Text>

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
        <Text className="btn-text-steampunk">Finish Setup</Text>
      </TouchableOpacity>
    </View>
  );
}
```

---

# Customizing Further

1. **Adjust the global classes** in `global.css` to match the exact style and colors you need for your steampunk vibe (e.g., richer browns, copper-like ambers, or subtle gear patterns).

2. **Add new classes** (e.g., `.card-steampunk`, `.heading-steampunk`, etc.) in the `@layer components` block for different UI elements.

3. **Use local images** or specialized steampunk images for your `ImageBackground` components, or incorporate gear icons and vintage fonts for a more immersive look.

With this approach, any time you need to tweak your steampunk palette or spacing, you can do it **once** in `global.css`, and all screens will stay consistent!

6. touch 404
8. links to list of 7 screens to preview
4. font
5. supdase for back end and login
7. StoryBook?!
