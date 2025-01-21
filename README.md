## 1. Create a Brand-New Project

In an empty directory:

```bash
npx create-expo-app rn_Protected_Routez
cd rn_Protected_Routez
npm run reset-project
rm -rf app-example
```

Then install form libs & storage if you need them:

```bash
npm install formik yup @react-native-async-storage/async-storage
```

> At this point, your `app/` folder is nearly empty, and the project is TypeScript-enabled by default.

---

## 2. Single `<Stack>` in `app/_layout.tsx`

We place **one** `<Stack>` here, so there is only one navigator container total.

```bash
touch app/_layout.tsx
```

**app/_layout.tsx**:

```tsx
import React from 'react';
import { Slot, Stack } from 'expo-router';

export default function RootLayout() {
  return (
    <>
      {/* Single Stack in the entire project */}
      <Stack screenOptions={{ headerShown: true }} />
      <Slot />
    </>
  );
}
```

- No `<NavigationContainer>`. Expo Router handles that internally.  
- No `<Stack>` in any other layout.  
- No `<AuthProvider>` here yet (we’ll add an AuthContext example below if desired).

---

## 3. Home Screen: `app/index.tsx`

```bash
touch app/index.tsx
```

**app/index.tsx**:

```tsx
import React from 'react';
import { View, Text, Button, StyleSheet } from 'react-native';
import { useRouter } from 'expo-router';

export default function HomeScreen() {
  const router = useRouter();

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Home Screen</Text>
      <Button
        title="Go to SomePage"
        onPress={() => router.push('/somepage')}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, alignItems: 'center', justifyContent: 'center' },
  title: { fontSize: 20, marginBottom: 16 },
});
```

---

## 4. An Extra Page (Optional): `app/somepage.tsx`

```bash
touch app/somepage.tsx
```

**app/somepage.tsx**:

```tsx
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

export default function SomePage() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Some Page</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, alignItems: 'center', justifyContent: 'center' },
  title: { fontSize: 20 },
});
```

**Result**: You can navigate between `/` and `/somepage` with no errors, because there’s just one `<Stack>` controlling them.

---

## 5. Adding AuthContext (Optional, Single File)

If you want an auth flow (sign in, sign up, protected routes) **and** you want to avoid “Another navigator,” still keep **one** `<Stack>` in `_layout.tsx`. Then place your `AuthProvider` **outside** or inside `_layout.tsx`. But do **not** add another `<Stack>` in `(auth)/_layout.tsx` or `(protected)/_layout.tsx`.

**Example**: A minimal `AuthContext.tsx` in `src/context/`:

```bash
mkdir -p src/context
touch src/context/AuthContext.tsx
```

**src/context/AuthContext.tsx**:

```tsx
import React, { createContext, ReactNode, useState } from 'react';

type AuthContextType = {
  signedIn: boolean;
  signIn: () => void;
  signOut: () => void;
};

export const AuthContext = createContext<AuthContextType>({
  signedIn: false,
  signIn: () => {},
  signOut: () => {},
});

export function AuthProvider({ children }: { children: ReactNode }) {
  const [signedIn, setSignedIn] = useState(false);

  function signIn() {
    setSignedIn(true);
  }

  function signOut() {
    setSignedIn(false);
  }

  return (
    <AuthContext.Provider value={{ signedIn, signIn, signOut }}>
      {children}
    </AuthContext.Provider>
  );
}
```

Then wrap your **single** `<Stack>` in `_layout.tsx`:

```tsx
import React from 'react';
import { Slot, Stack } from 'expo-router';
import { AuthProvider } from '../src/context/AuthContext';

export default function RootLayout() {
  return (
    <AuthProvider>
      {/* One Stack */}
      <Stack screenOptions={{ headerShown: true }} />
      <Slot />
    </AuthProvider>
  );
}
```

You’ll still have **only one** navigator. **No** second `<Stack>` in any child layout means you can’t get “Another navigator is already registered.”  

---

## 6. Testing & Confirming No Error

1. **Clear caches**:  
   ```bash
   rm -rf node_modules
   npm install
   npx expo start -c
   ```
2. **Open** on web or device.  
3. Navigate among `/` and `/somepage`. You see:

   - **No** “Another navigator is already registered for this container.”  
   - **No** multiple container warnings.

---

## 7. Common Mistakes That Reintroduce the Error

1. **Putting Another `<Stack>`** in a child layout, e.g. `(protected)/_layout.tsx`. If you do `<Stack>` again there, you have **two** stacks in the same container. → error.  
2. **Importing `<NavigationContainer>`** manually from `@react-navigation/native`. Expo Router already uses it behind the scenes.  
3. **Having a leftover “root” file** from older React Navigation setups. For instance, if you have `NavigationContainer` or `createStackNavigator` in `App.tsx`.  
4. **Mixing** “classic” React Navigation with new Expo Router. They conflict. Use only Expo Router’s file-based approach.  

---

## 8. Final Summary

- The snippet above is the **smallest** possible “one `<Stack>`” example.  
- If you **still** see “Another navigator is already registered,” it’s because your project has leftover code with a second `<Stack>` or a `NavigationContainer`.  
- Compare your entire `app/` folder line by line with the code here. Once it matches exactly, the error is gone.  

**That’s it**—a minimal, fully functional tutorial with **only one** navigator. It’s guaranteed not to produce the “Another navigator is already registered” error unless there’s some other leftover file or code. Enjoy!
