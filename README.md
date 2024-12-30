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
npm install @expo-google-fonts/roboto-slab @expo-google-fonts/special-elite @expo-google-fonts/arbutus-slab
```
##### Initialize Tailwind
```bash
npx tailwindcss init
```
##### Initialize metro config
```bash
npx expo customize metro.conf
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
const { getDefaultConfig } = require("expo/metro-config");
const { withNativeWind } = require("nativewind/metro");

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
          robotoSlab: ['RobotoSlab_700Bold'],
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
import { RobotoSlab_700Bold } from '@expo-google-fonts/roboto-slab';
import { SpecialElite_400Regular } from '@expo-google-fonts/special-elite';
import { ArbutusSlab_400Regular } from '@expo-google-fonts/arbutus-slab';
import Splash from "./(onboarding)/splash";

export default function RootLayout() {
  const [fontsLoaded] = useFonts({
    RobotoSlab_700Bold,
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
    <SafeAreaView className="container-steampunk bg-steampunk">
      <View>
        <Text className="text-steampunk-title">SteamPunk App</Text>
        <Text className="text-steampunk-body">
          Welcome to your steampunk-themed application powered by Nativewind and Tailwind CSS.
        </Text>
      </View>

      {/* Navigation Links */}
      <View className="mt-8">
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

      <View className="mt-8 border-steampunk">
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

### app/(auth)/sign-up.tsx
```typescript
import React from "react";
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
import React from "react";
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
import React from "react";
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
import React from "react";
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
import React from "react";
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
import React from "react";
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
npm install @expo-google-fonts/rye @expo-google-fonts/special-elite @expo-google-fonts/arbutus-slab
```
##### Initialize Tailwind
```bash
npx tailwindcss init
```
##### Initialize metro config
```bash
npx expo customize metro.conf
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
const { getDefaultConfig } = require("expo/metro-config");
const { withNativeWind } = require("nativewind/metro");

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
          rye: ['Rye_400Regular'],
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
    @apply text-3xl text-amber-400 font-bold mb-6 font-rye;
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
import { Rye_400Regular } from '@expo-google-fonts/rye';
import { SpecialElite_400Regular } from '@expo-google-fonts/special-elite';
import { ArbutusSlab_400Regular } from '@expo-google-fonts/arbutus-slab';
import Splash from "./(onboarding)/splash"; // Import your custom Splash component

export default function RootLayout() {
  const [fontsLoaded] = useFonts({
    Rye_400Regular,
    SpecialElite_400Regular,
    ArbutusSlab_400Regular,
  });

  if (!fontsLoaded) {
    return <Splash />; // Use your custom splash screen component
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
```typescriptimport { Text, View } from "react-native";
import { SafeAreaView } from "react-native-safe-area-context";
import { Link } from "expo-router";

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

      <View className="mt-8 border-steampunk">
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

### app/(auth)/sign-up.tsx
```typescript
import React from "react";
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
import React from "react";
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
import React from "react";
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
import React from "react";
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
import React from "react";
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
import React from "react";
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
