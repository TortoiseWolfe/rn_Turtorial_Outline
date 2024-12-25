## 1. Start Fresh and Set Up Project

### Step 0: Delete Existing Project Directory

If you need to start over, delete the existing project directory:

```bash
rm MyHiddenRoutesApp
```

## 2. Set Up Project and Install Dependencies

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
npx expo install expo-router react-native-screens react-native-safe-area-context nativewind tailwindcss
npx tailwindcss init
```

### Step 5: Generate Configuration Files

Use `npx expo customize` to generate supported files:

```bash
npx expo customize metro.config.js babel.config.js
```

For files not covered, create them manually:

```bash
touch global.css nativewind.d.ts
```

---

## 2. Configure the Project

### Step 1: Update `metro.config.js`

Replace the contents of `metro.config.js` with:

```js
const { getDefaultConfig } = require("expo/metro-config");
const { withNativeWind } = require("nativewind/metro");

const config = getDefaultConfig(__dirname);

module.exports = withNativeWind(config);
```

### Step 2: Update `tailwind.config.js`

Replace the contents of `tailwind.config.js` with:

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./index.{js,ts,jsx,tsx}",
    "./app/**/*.{js,ts,jsx,tsx}",
  ],
  presets: [require('nativewind/preset')],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

### Step 3: Update `babel.config.js`

Populate `babel.config.js` with:

```js
module.exports = function (api) {
  api.cache(true);
  return {
    presets: [
      [
        "babel-preset-expo",
        {
          jsxImportSource: "nativewind",
        },
      ],
    ],
  };
};
```

### Step 4: Configure `global.css`

Populate `global.css` with:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Import `global.css` in your entry file (`index.ts`):

```ts
import "./global.css";
```

### Step 5: Configure TypeScript for NativeWind

To support TypeScript, add the following `nativewind.d.ts` file in the root of your project:

```ts
declare module "nativewind" {
  export function styled<T>(component: T): T;
}
```

---

## 3. File Contents

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
import "../global.css";
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

## 4. Start the Project

### Step 1: Add Web Bundler Configuration

Verify or add the following to your `app.json`:

```json
{
  "expo": {
    "web": {
      "bundler": "metro"
    }
  }
}
```

### Step 2: Start the Development Server

Restart the Metro bundler to clear cache:

```bash
npm start --reset-cache
```

Start the development server:

```bash
npx expo start --web
```

- You should see a **light gray background** and large, **blue** “Hello NativeWind + Expo Router on Web!” text in the center of your browser.
- This confirms **Expo Router**, **NativeWind**, and **TypeScript** are set up correctly.

---

## 5. Debugging Common Errors

### Error: `_nativewind.styled is not a function`

If you encounter this error, follow these steps:

1. **Ensure Dependencies Are Installed**:
   ```bash
   npm install nativewind tailwindcss
   ```

2. **Verify `babel.config.js`**:
   Ensure it matches the configuration provided in Step 3 of "Configure the Project".

3. **Restart Metro Bundler**:
   Clear the cache and restart:
   ```bash
   npm start --reset-cache
   ```

4. **Recheck All Configuration Files**:
   - `metro.config.js`
   - `tailwind.config.js`
   - `babel.config.js`

---

# Summary

1. **Create** a new Expo project and reset it.
2. **Install** dependencies for NativeWind, TailwindCSS, and Expo Router.
3. **Generate** configuration files using `npx expo customize`.
4. **Configure** `tailwind.config.js`, `babel.config.js`, `global.css`, and TypeScript types.
5. **Edit** the provided file contents to verify NativeWind and Tailwind integration.
6. **Debug** common issues if errors occur.
7. **Run** the project on web to confirm the setup works.

Enjoy coding!

