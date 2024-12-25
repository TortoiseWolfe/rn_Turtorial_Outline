## 1. Set Up Project and Install Dependencies

```bash
npx create-expo-app MyHiddenRoutesApp --template blank
```
```bash
cd MyHiddenRoutesApp
```
```bash
npx expo install nativewind tailwindcss react-native-reanimated react-native-safe-area-context
```
```bash
npx tailwindcss init
```
---

## 2. Configure Tailwind CSS

### **`tailwind.config.js`**

Open the `tailwind.config.js` file and replace its contents with:

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

## 3. Configure Babel

### **`babel.config.js`**

Update the Babel configuration to include the NativeWind plugin:

```js
module.exports = function (api) {
  api.cache(true);
  return {
    presets: ["babel-preset-expo"],
    plugins: ["nativewind/babel"],
  };
};
```

---

## 4. Configure Metro Bundler

### **`metro.config.js`**

Create or update the `metro.config.js` file to integrate NativeWind:

```js
const { getDefaultConfig } = require("expo/metro-config");
const { withNativeWind } = require("nativewind/metro");

const config = getDefaultConfig(__dirname);

module.exports = withNativeWind(config, {
  input: "./global.css",
});
```

---

## 5. Create a Global CSS File

### **`global.css`**

Create a `global.css` file and add the Tailwind directives:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Import this file in your entry file (`index.js` or `index.ts`):

```js
import "./global.css";
```

---

## 6. File Contents

### **`app/_layout.tsx`**

```tsx
import { Stack } from 'expo-router';
import React from 'react';

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

## 7. Start the Project

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

1. **Create** a new Expo project and install dependencies.
2. **Configure** `tailwind.config.js`, `babel.config.js`, and `metro.config.js`.
3. **Create** a `global.css` file for Tailwind directives.
4. **Edit** the provided file contents to test NativeWind functionality.
5. **Run** the project on web to verify the setup.

Enjoy coding!

