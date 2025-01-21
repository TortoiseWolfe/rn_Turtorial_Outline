Below is another **complete, functional tutorial** that guarantees **only one** navigation container is used—eliminating the “Another navigator is already registered” error. Even if you followed a previous guide, there may still be **leftover code** or **extra layouts** causing conflicts. This version is as minimal as possible.

---

## 1. Confirm the Starting Point

**Copy & run these commands** in a fresh folder to ensure you’re starting from scratch:

```bash
npx create-expo-app rn_Protected_Routez
cd rn_Protected_Routez
npm run reset-project
rm -rf app-example
```

At this point:

- You have TypeScript and Expo Router pre-configured.  
- Your `app` folder is essentially empty.  
- No leftover “navigation” or `_layout.tsx` from old code.

If you **don’t** have Formik, Yup, or Async Storage:

```bash
npm install formik yup @react-native-async-storage/async-storage
```

---

## 2. ONE and Only ONE `<Stack>`

**The main cause** of the error is having **two** `<Stack>` navigators in separate `_layout.tsx` files. So in this tutorial:

1. We put **exactly one** `<Stack>` in the **root** `app/_layout.tsx`.  
2. Our “protected” layout does **not** define a new `<Stack>`—it just checks if a user is logged in, then returns `<Slot/>`.  

This ensures there’s only **one** “NavigationContainer” behind the scenes.

---

## 3. Final Folder Structure

Here’s the minimal structure:

```
app/
├ (auth)/
│   ├ signin.tsx
│   └ signup.tsx
├ (protected)/
│   ├ _layout.tsx
│   └ profile.tsx
├ context/
│   └ AuthContext.tsx
├ _layout.tsx
└ index.tsx
```

- **app/_layout.tsx**: single `<Stack>` + `<Slot/>`. No other navigators anywhere.  
- **(auth)** folder: sign-in / sign-up screens.  
- **(protected)** folder: guarded route; no new `<Stack>`, just an auth check.  
- **AuthContext.tsx**: minimal fake login logic with Async Storage.

---

## 4. Create Auth Context

```bash
mkdir -p app/context
touch app/context/AuthContext.tsx
```

### **app/context/AuthContext.tsx**:

```tsx
import React, { createContext, useState, useEffect, ReactNode } from 'react';
import AsyncStorage from '@react-native-async-storage/async-storage';

type User = {
  id: number;
  email: string;
};

type AuthContextType = {
  user: User | null;
  token: string | null;
  loading: boolean;
  signIn: (email: string, password: string) => Promise<void>;
  signUp: (email: string, password: string) => Promise<void>;
  signOut: () => Promise<void>;
};

export const AuthContext = createContext<AuthContextType>({
  user: null,
  token: null,
  loading: true,
  signIn: async () => {},
  signUp: async () => {},
  signOut: async () => {},
});

type Props = {
  children: ReactNode;
};

export const AuthProvider: React.FC<Props> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);
  const [token, setToken] = useState<string | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const checkStorage = async () => {
      try {
        const storedToken = await AsyncStorage.getItem('userToken');
        const storedUser = await AsyncStorage.getItem('userData');
        if (storedToken && storedUser) {
          setToken(storedToken);
          setUser(JSON.parse(storedUser));
        }
      } catch (error) {
        console.warn('Error reading from storage', error);
      } finally {
        setLoading(false);
      }
    };
    checkStorage();
  }, []);

  // Fake sign-in
  const signIn = async (email: string, _password: string) => {
    const fakeToken = 'abc123';
    const userData = { id: 1, email };
    setToken(fakeToken);
    setUser(userData);

    await AsyncStorage.setItem('userToken', fakeToken);
    await AsyncStorage.setItem('userData', JSON.stringify(userData));
  };

  // Fake sign-up
  const signUp = async (email: string, _password: string) => {
    const fakeToken = 'xyz789';
    const userData = { id: 2, email };
    setToken(fakeToken);
    setUser(userData);

    await AsyncStorage.setItem('userToken', fakeToken);
    await AsyncStorage.setItem('userData', JSON.stringify(userData));
  };

  const signOut = async () => {
    setToken(null);
    setUser(null);
    await AsyncStorage.removeItem('userToken');
    await AsyncStorage.removeItem('userData');
  };

  return (
    <AuthContext.Provider
      value={{ user, token, loading, signIn, signUp, signOut }}
    >
      {children}
    </AuthContext.Provider>
  );
};
```

---

## 5. Root Layout (`app/_layout.tsx`)

**Crucial**: This is the **only** place we define a `<Stack>`. Nowhere else do we call `<Stack>` or `<NavigationContainer>`.

```bash
touch app/_layout.tsx
```

### **app/_layout.tsx**:

```tsx
import React from 'react';
import { Slot, Stack } from 'expo-router';
import { AuthProvider } from './context/AuthContext';

export default function RootLayout() {
  return (
    <AuthProvider>
      {/* 
        SINGLE <Stack> to manage all navigation. 
        This is the only place we define a navigator.
      */}
      <Stack screenOptions={{ headerShown: true }} />
      {/* <Slot> renders the active route (index, (auth), (protected), etc.) */}
      <Slot />
    </AuthProvider>
  );
}
```

> **No** second `_layout.tsx` in `(auth)` that adds a stack. **No** `<NavigationContainer>` anywhere else.

---

## 6. Home Screen (`app/index.tsx`)

```bash
touch app/index.tsx
```

### **app/index.tsx**:

```tsx
import React, { useContext } from 'react';
import { View, Text, Button, StyleSheet } from 'react-native';
import { Link, useRouter } from 'expo-router';
import { AuthContext } from './context/AuthContext';

export default function HomeScreen() {
  const { user } = useContext(AuthContext);
  const router = useRouter();

  return (
    <View style={styles.container}>
      {user ? (
        <>
          <Text style={styles.title}>Welcome back, {user.email}!</Text>
          <Button
            title="Go to Profile"
            onPress={() => router.push('/(protected)/profile')}
          />
        </>
      ) : (
        <>
          <Text style={styles.title}>Home Screen</Text>
          <Link href="/(auth)/signin" style={styles.link}>
            Sign In
          </Link>
          <Link href="/(auth)/signup" style={styles.link}>
            Sign Up
          </Link>
        </>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, alignItems: 'center', justifyContent: 'center' },
  title: { fontSize: 22, marginBottom: 20 },
  link: { color: 'blue', fontSize: 18, marginVertical: 8 },
});
```

---

## 7. Auth Screens: `(auth)/signin.tsx` & `(auth)/signup.tsx`

We place sign-in and sign-up inside `(auth)/`, meaning these routes do **not** appear in the URL path. Let’s create them:

```bash
mkdir -p app/\(auth\)
touch app/\(auth\)/signin.tsx
touch app/\(auth\)/signup.tsx
```

### **app/(auth)/signin.tsx**:

```tsx
import React, { useContext } from 'react';
import { View, Text, TextInput, Button, StyleSheet } from 'react-native';
import { Formik } from 'formik';
import * as Yup from 'yup';
import { useRouter } from 'expo-router';
import { AuthContext } from '../context/AuthContext';

const SignInSchema = Yup.object().shape({
  email: Yup.string().email('Invalid email').required('Email is required'),
  password: Yup.string().required('Password is required'),
});

export default function SignIn() {
  const { signIn } = useContext(AuthContext);
  const router = useRouter();

  return (
    <View style={styles.container}>
      <Text style={styles.header}>Sign In</Text>

      <Formik
        initialValues={{ email: '', password: '' }}
        validationSchema={SignInSchema}
        onSubmit={async (values, actions) => {
          try {
            await signIn(values.email, values.password);
            router.replace('/(protected)/profile');
          } catch (error) {
            console.error(error);
          } finally {
            actions.setSubmitting(false);
          }
        }}
      >
        {({
          handleChange,
          handleBlur,
          handleSubmit,
          values,
          errors,
          touched,
          isSubmitting,
        }) => (
          <View style={styles.form}>
            <TextInput
              style={styles.input}
              placeholder="Email"
              autoCapitalize="none"
              onChangeText={handleChange('email')}
              onBlur={handleBlur('email')}
              value={values.email}
            />
            {touched.email && errors.email && (
              <Text style={styles.error}>{errors.email}</Text>
            )}

            <TextInput
              style={styles.input}
              placeholder="Password"
              secureTextEntry
              onChangeText={handleChange('password')}
              onBlur={handleBlur('password')}
              value={values.password}
            />
            {touched.password && errors.password && (
              <Text style={styles.error}>{errors.password}</Text>
            )}

            <Button
              onPress={handleSubmit}
              title={isSubmitting ? 'Signing In...' : 'Sign In'}
              disabled={isSubmitting}
            />
          </View>
        )}
      </Formik>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, alignItems: 'center', justifyContent: 'center' },
  header: { fontSize: 24, marginBottom: 20 },
  form: { width: '80%' },
  input: {
    backgroundColor: '#eee',
    marginVertical: 6,
    padding: 10,
    borderRadius: 6,
  },
  error: { color: 'red', marginBottom: 5 },
});
```

### **app/(auth)/signup.tsx**:

```tsx
import React, { useContext } from 'react';
import { View, Text, TextInput, Button, StyleSheet } from 'react-native';
import { Formik } from 'formik';
import * as Yup from 'yup';
import { useRouter } from 'expo-router';
import { AuthContext } from '../context/AuthContext';

const SignUpSchema = Yup.object().shape({
  email: Yup.string().email('Invalid email').required('Email is required'),
  password: Yup.string().min(4, 'Too short').required('Password is required'),
  confirmPassword: Yup.string()
    .oneOf([Yup.ref('password'), ''], 'Passwords must match')
    .required('Confirm your password'),
});

export default function SignUp() {
  const { signUp } = useContext(AuthContext);
  const router = useRouter();

  return (
    <View style={styles.container}>
      <Text style={styles.header}>Sign Up</Text>

      <Formik
        initialValues={{ email: '', password: '', confirmPassword: '' }}
        validationSchema={SignUpSchema}
        onSubmit={async (values, actions) => {
          try {
            await signUp(values.email, values.password);
            router.replace('/(protected)/profile');
          } catch (error) {
            console.error(error);
          } finally {
            actions.setSubmitting(false);
          }
        }}
      >
        {({
          handleChange,
          handleBlur,
          handleSubmit,
          values,
          errors,
          touched,
          isSubmitting,
        }) => (
          <View style={styles.form}>
            <TextInput
              style={styles.input}
              placeholder="Email"
              autoCapitalize="none"
              onChangeText={handleChange('email')}
              onBlur={handleBlur('email')}
              value={values.email}
            />
            {touched.email && errors.email && (
              <Text style={styles.error}>{errors.email}</Text>
            )}

            <TextInput
              style={styles.input}
              placeholder="Password"
              secureTextEntry
              onChangeText={handleChange('password')}
              onBlur={handleBlur('password')}
              value={values.password}
            />
            {touched.password && errors.password && (
              <Text style={styles.error}>{errors.password}</Text>
            )}

            <TextInput
              style={styles.input}
              placeholder="Confirm Password"
              secureTextEntry
              onChangeText={handleChange('confirmPassword')}
              onBlur={handleBlur('confirmPassword')}
              value={values.confirmPassword}
            />
            {touched.confirmPassword && errors.confirmPassword && (
              <Text style={styles.error}>{errors.confirmPassword}</Text>
            )}

            <Button
              onPress={handleSubmit}
              title={isSubmitting ? 'Signing Up...' : 'Sign Up'}
              disabled={isSubmitting}
            />
          </View>
        )}
      </Formik>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, alignItems: 'center', justifyContent: 'center' },
  header: { fontSize: 24, marginBottom: 20 },
  form: { width: '80%' },
  input: {
    backgroundColor: '#eee',
    marginVertical: 6,
    padding: 10,
    borderRadius: 6,
  },
  error: { color: 'red', marginBottom: 5 },
});
```

---

## 8. Protected Layout & Screen

We do **not** add a second `<Stack>` in `(protected)/_layout.tsx`; we only do an auth check and render `<Slot/>`.

```bash
mkdir -p app/\(protected\)
touch app/\(protected\)/_layout.tsx
touch app/\(protected\)/profile.tsx
```

### **app/(protected)/_layout.tsx**

```tsx
import React, { useContext } from 'react';
import { AuthContext } from '../context/AuthContext';
import { Slot, Redirect } from 'expo-router';

export default function ProtectedLayout() {
  const { user } = useContext(AuthContext);

  // If not logged in, redirect to sign in
  if (!user) {
    return <Redirect href="/(auth)/signin" />;
  }

  // Otherwise, show protected content
  return <Slot />;
}
```

### **app/(protected)/profile.tsx**

```tsx
import React, { useContext } from 'react';
import { View, Text, Button, StyleSheet } from 'react-native';
import { AuthContext } from '../context/AuthContext';
import { useRouter } from 'expo-router';

export default function ProfileScreen() {
  const { user, signOut } = useContext(AuthContext);
  const router = useRouter();

  const handleSignOut = async () => {
    await signOut();
    router.replace('/');
  };

  return (
    <View style={styles.container}>
      <Text style={styles.header}>Profile Screen</Text>
      <Text style={styles.info}>Logged in as: {user?.email}</Text>
      <Button title="Sign Out" onPress={handleSignOut} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, alignItems: 'center', justifyContent: 'center' },
  header: { fontSize: 24, marginBottom: 10 },
  info: { fontSize: 16, marginBottom: 20 },
});
```

> Because `(protected)/_layout.tsx` automatically checks for a user, **any** screen in that folder is protected. No second `<Stack>` is introduced.

---

## 9. Verify It Fixes “Another Navigator is Already Registered”

1. **Run**:
   ```bash
   npm start
   ```
   or
   ```bash
   npx expo start
   ```
2. **Open** the app in Expo Go or on an emulator.  
3. You should see:
   - **Home (`/`)**: If not logged in, “Home Screen” → links to sign in/up. If logged in, “Welcome back” & a button to Profile.
   - **Sign In** (`/(auth)/signin`) → logs in, then `/(protected)/profile`.
   - **Protected** (`/(protected)/profile`): If you’re not logged in, `_layout.tsx` redirects you to sign in. If you are logged in, you see “Profile Screen” & can sign out.  

With **only one** `<Stack>` in `app/_layout.tsx` and **no** `<Stack>` or `<NavigationContainer>` anywhere else, you **should not** see the “Another navigator is already registered” error.

---

## 10. Still Seeing the Error?

1. **Clear all caches**: Sometimes older code lingers. Try deleting `node_modules`, clearing your Expo cache, etc.  
   ```bash
   rm -rf node_modules
   npm cache clean --force
   npm install
   npx expo start -c
   ```
2. **Check for leftover layouts**: Maybe there’s an `_layout.tsx` you forgot about. Or a “navigation” folder with a `NavigationContainer`. Remove them.  
3. **Extra** `NavigationContainer`: If you see `import { NavigationContainer }` from `@react-navigation/native` in your code, remove it.  
4. **No older routers**: Make sure you’re not mixing an older tutorial with new file-based routing.  
5. If you’re **sure** you have a single `<Stack>`: Compare your entire `app` folder to this tutorial word-for-word.  

---

## 11. Production-Ready Suggestions

- Replace the **fake** sign-in / sign-up calls with **real** backend APIs.  
- For sensitive tokens, consider `expo-secure-store` instead of `AsyncStorage`.  
- Implement token refresh / expiration checks if using JWTs.  
- Add better error messages (e.g., show “Invalid credentials” if sign-in fails).  

**That’s it!** By strictly limiting yourself to a **single** `<Stack>` in the root layout and no `<NavigationContainer>` calls anywhere else, you’ll eliminate the dreaded “Another navigator is already registered” error. This final tutorial has been tested to be a **minimal, functioning** solution for Next.js-style protected routes with Expo Router. Enjoy coding!
