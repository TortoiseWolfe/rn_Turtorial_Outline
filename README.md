Below is a **concise** tutorial to set up a minimal **Expo + TypeScript + Expo Router + NativeWind** project. We’ll use commands to create files and show the contents you can paste manually.

---

## 1. Set Up Project and Install Dependencies

```bash
npx create-expo-app MyHiddenRoutesApp --template blanktypescript
```

```bash
cd MyHiddenRoutesApp
```

```bash
npx expo install expo-router react-native-screens react-native-safe-area-context nativewind tailwindcss react-native-reanimated
```

```bash
npx tailwindcss init
```

---

## 2. Remove Default Files and Create the Required Structure

```bash
rm App.tsx
mkdir app
touch index.ts
touch app/_layout.tsx
touch app/index.tsx
touch babel.config.js
```

### **`babel.config.js`**

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

## 3. Configure Tailwind

### **`tailwind.config.js`**

Open the `tailwind.config.js` file and replace its contents with:

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./index.{ts,tsx}",
    "./app/**/*.{ts,tsx}",
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

### **`index.ts`**

```ts
import 'expo-router/entry';
```

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

## 5. Start the Project

```bash
npx expo start --web
```

- You should see a **light gray background** and large, **blue** “Hello NativeWind + Expo Router on Web!” text in the center of your browser.
- This confirms **Expo Router**, **NativeWind**, and **TypeScript** are set up.

---

# Summary

1. **Create** a new Expo project and install dependencies.
2. **Remove** `App.tsx` and set up the `app/` folder with `index.ts` and layout files.
3. **Configure** `tailwind.config.js` and `babel.config.js` for NativeWind.
4. **Paste** the provided file contents.
5. **Run** the project on web to verify the setup.

Enjoy coding!

