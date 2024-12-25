Below is a **complete** tutorial to set up a minimal **Expo + TypeScript + Expo Router + NativeWind** project, updated to match your actual commands, including resetting the project.

---

## 1. Set Up Project and Install Dependencies

### Step 1: Create a New Expo Project

```bash
npx create-expo-app MyHiddenRoutesApp
```

### Step 2: Navigate into the Project Directory

```bash
cd MyHiddenRoutesApp
```

### Step 3: Reset the Project

```bash
npm run reset-project
```

### Step 4: Install Required Dependencies

```bash
npx expo install nativewind tailwindcss react-native-reanimated react-native-safe-area-context
```

### Step 5: Initialize Tailwind CSS

```bash
npx tailwindcss init
```

### Step 6: Create Necessary Configuration Files

```bash
touch babel.config.js metro.config.js global.css
```

---

## 2. Automate Configuration File Creation

### Configure `babel.config.js`

Populate `babel.config.js` with:

```js
module.exports = function (api) {
  api.cache(true);
  return {
    presets: ["babel-preset-expo"],
    plugins: [
      ["nativewind/babel"]
    ],
  };
};
```

### Configure `metro.config.js`

Populate `metro.config.js` with:

```js
const { getDefaultConfig } = require("expo/metro-config");
const { withNativeWind } = require("nativewind/metro");

const config = getDefaultConfig(__dirname);

module.exports = withNativeWind(config);
```

### Configure `global.css`

Populate `global.css` with:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

---

## 3. Configure Tailwind CSS

### **`tailwind.config.js`**

Replace the contents of `tailwind.config.js` with:

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./app/**/*.{js,jsx,ts,tsx}",
  ],
  presets: [require('nativewind/preset')],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

---

## 4. File Contents

### **`app/_layout.tsx`**

```tsx
import { Stack } from 'expo-router';

export default function RootLayout() {
  return (
    <Stack
      screenOptions={{
        headerShown: false,
      }}
    />
  );
}
```

### **`app/index.tsx`**

```tsx
import React from 'react';
import { styled } from 'nativewind';
import { View, Text } from 'react-native';

const StyledView = styled(View);
const StyledText = styled(Text);
import "../global.css";

export default function HomeScreen() {
  return (
    <StyledView className="flex-1 items-center justify-center bg-gray-100">
      <StyledText className="text-2xl font-bold text-blue-600">
        Hello NativeWind + Expo Router on Web!
      </StyledText>
    </StyledView>
  );
}
```

---

## 5. Start the Project

### Add Web Bundler Configuration

Add the following to your `app.json`:

```json
{
  "expo": {
    "web": {
      "bundler": "metro"
    }
  }
}
```

### Start the Development Server

```bash
npx expo start --web
```

- You should see a **light gray background** and large, **blue** “Hello NativeWind + Expo Router on Web!” text in the center of your browser.
- This confirms **Expo Router**, **NativeWind**, and **TypeScript** are set up.

---

# Summary

1. **Create** a new Expo project and reset it.
2. **Automate** creation of configuration files using `touch` commands.
3. **Configure** `tailwind.config.js`, `babel.config.js`, and `metro.config.js`.
4. **Create** and import `global.css` for Tailwind directives.
5. **Edit** the provided file contents to test NativeWind functionality.
6. **Run** the project on web to verify the setup.

Enjoy coding!

