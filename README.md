## 1. Create a New Expo + TypeScript Project

```bash
npx create-expo-app MyWebTailwindApp --template blanktypescript
```

```
cd MyWebTailwindApp
```

**Result**: A blank TypeScript Expo project with some default files (including `App.tsx`).

---

## 2. Remove `App.tsx` and Use a Minimal `index.ts` for Expo Router

```bash
rm App.tsx
touch index.ts
```

**File**: **`index.ts`**

```ts
import 'expo-router/entry';
```

This tells Expo to boot from Expo Router instead of a custom App component.

---

## 3. Install and Configure Expo Router

```bash
npx expo install expo-router react-native-screens react-native-safe-area-context
```

No file changes here—just installing dependencies.

---

## 4. Set Up the `app/` Folder

1. **Create** the `app/` folder:

   ```bash
   mkdir app
   ```

2. **Create** two files:

   ```bash
   touch app/_layout.tsx
   touch app/index.tsx
   ```

**File**: `app/_layout.tsx`

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

**File**: `app/index.tsx`

```tsx
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

export default function HomeScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Hello Expo Router + TS + Web!</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: 'center', alignItems: 'center' },
  title: { fontSize: 24 },
});
```

---

## 5. Verify on Web (Before Adding Tailwind)

```bash
npx expo start --web
```

- Browser should show “**Hello Expo Router + TS + Web!**” in the center.
- If that works, you have **Expo Router** + **TypeScript** + **Web** in place.

---

## 6. Install NativeWind (Tailwind for React Native)

```bash
npm install nativewind tailwindcss
npx tailwindcss init
```

**Result**: A default `tailwind.config.js` is created. We’ll configure it next.

---

## 7. Configure Tailwind & Babel

1. **Update** `tailwind.config.js`:

   ```bash
   # not creating the file, just open it in your editor
   ```

   **File**: `tailwind.config.js`

   ```js
   /** @type {import('tailwindcss').Config} */
   module.exports = {
     content: [
       "./index.{ts,tsx}",
       "./app/**/*.{ts,tsx}",
     ],
     theme: {
       extend: {},
     },
     plugins: [],
   };
   ```

2. **Create** or **edit** `babel.config.js`:

   ```bash
   touch babel.config.js
   ```

   **File**: `babel.config.js`

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

## 8. Use Tailwind in the Home Screen

Update the **`app/index.tsx`** file to see Tailwind classes in action:

```bash
# We already have app/index.tsx, so just open it and replace its contents
```

**File**: `app/index.tsx`

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

## 9. Run on Web Again

```bash
npx expo start --web
```

- You should see a **light gray background** and large, **blue** “Hello NativeWind + Expo Router on Web!” text in the browser.
- No errors = success!

---

# Summary

1. **Create** a TypeScript Expo app (`create-expo-app ... --template blanktypescript`).
2. **Delete** `App.tsx`, make a **`index.ts`** that imports `'expo-router/entry'`.
3. **Add** a minimal `app/_layout.tsx` + `app/index.tsx` for Expo Router.
4. **Install** `nativewind` + `tailwindcss`; configure `tailwind.config.js` and `babel.config.js`.
5. **Apply** Tailwind classes to a screen.
6. **Run** `npx expo start --web` to confirm it works.

That’s it! You’ve got a **TypeScript** project, **Expo Router** for navigation, **NativeWind** for Tailwind styling, and **web** support all set up. Now you can **add more screens** by creating `.tsx` files in `app/`. Enjoy coding!

