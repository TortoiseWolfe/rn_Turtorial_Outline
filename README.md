Below is a **complete** tutorial to set up a minimal **Expo + TypeScript + Expo Router + NativeWind** project, updated to reflect the commands from your workflow and resolve common issues.

---

## 1. Set Up Project and Install Dependencies

```bash
npx create-expo-app MyHiddenRoutesApp --template
npm run reset-project
npx expo install expo-router react-native-screens react-native-safe-area-context nativewind tailwindcss react-native-reanimated
npx tailwindcss init
```

---

## 2. Verify and Adjust the Project Structure

The reset command (`npm run reset-project`) creates the following structure:

- A new `/app/` directory created.
- Default files such as `app/_layout.tsx` and `app/index.tsx` generated.

These files should already exist, but you can edit them as follows:

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

### **`babel.config.js`**

The default `babel.config.js` should already exist. Update it with:

```bash
touch babel.config.js
```

Add the following to `babel.config.js`:

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

## 4. Debugging and Testing NativeWind

### 4.1 Test NativeWind Integration

Update **`app/index.tsx`** to verify NativeWind is working:

```tsx
import React from 'react';
import { styled } from 'nativewind';
import { View, Text } from 'react-native';

const StyledView = styled(View);
const StyledText = styled(Text);

export default function HomeScreen() {
  return (
    <StyledView className="flex-1 items-center justify-center bg-blue-100">
      <StyledText className="text-4xl text-red-500 font-bold">
        NativeWind Test!
      </StyledText>
    </StyledView>
  );
}
```

### 4.2 Common Issues

- **Error**: `(0, _nativewind.styled) is not a function`
  - Ensure NativeWind is installed correctly: `npm install nativewind@latest`.
  - Verify Babel configuration includes `"nativewind/babel"`.

- **Error**: Styles not applied
  - Check `tailwind.config.js` paths include `"./app/**/*.{ts,tsx}"` and `"./index.{ts,tsx}"`.
  - Restart the Metro bundler: `npm start --reset-cache`.

---

## 5. File Contents

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

## 6. Start the Project

```bash
npx expo start --web
```

- You should see a **light gray background** and large, **blue** “Hello NativeWind + Expo Router on Web!” text in the center of your browser.
- This confirms **Expo Router**, **NativeWind**, and **TypeScript** are set up.

---

# Summary

1. **Create** a new Expo project and reset it.
2. **Verify** that the `/app/` folder and default files (`_layout.tsx`, `index.tsx`) are created by the template.
3. **Configure** `tailwind.config.js` and `babel.config.js` for NativeWind.
4. **Test** NativeWind with a sample screen and debug any issues.
5. **Run** the project on web to verify the setup.

Enjoy coding!

