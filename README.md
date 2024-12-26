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
