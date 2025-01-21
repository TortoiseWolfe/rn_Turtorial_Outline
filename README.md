## 1. Create a Fresh Expo Project

```bash
npx create-expo-app rn_Protected_Routez
cd rn_Protected_Routez
npm run reset-project
rm -rf app-example
```
At this point, your `app` folder is basically empty.

Install any needed libraries:
```bash
npm install formik yup @react-native-async-storage/async-storage
```
Then confirm that your `app` folder has no leftover code (just `_layout.tsx` if anything).

---

## 2. Create a `src/context` Folder

We want our AuthContext **outside** the `app` folder, so Expo Router doesn’t treat it like a route screen.

```bash
mkdir -p src/context
touch src/context/AuthContext.ts
```

**src/context/AuthContext.ts**:

```ts
import React, { createContext, useEffect, useState, ReactNode } from 'react';
import AsyncStorage from '@react-native-async-storage/async-storage';

/** A simple user object, for example's sake */
type User = {
  id: number;
  email: string;
};

/** The shape of our auth context: user data, loading, and auth methods */
interface AuthContextType {
  user: User | null;
  token: string | null;
  loading: boolean;
  signIn: (email: string, password: string) => Promise<void>;
  signUp: (email: string, password: string) => Promise<void>;
  signOut: () => Promise<void>;
}

/** 
 * We rename our context variable to avoid any confusion with TS "namespaces"
 * "MyAuthContext" is just a normal variable, not a namespace.
 */
export const MyAuthContext = createContext<AuthContextType>({
  user: null,
  token: null,
  loading: true,
  signIn: async () => {},
  signUp: async () => {},
  signOut: async () => {},
});

/** Props for our AuthProvider, expecting children to render inside */
interface AuthProviderProps {
  children: ReactNode;
}

/**
 * The AuthProvider component wraps your app,
 * providing "user", "token", "signIn", etc. in context.
 */
export function AuthProvider({ children }: AuthProviderProps) {
  const [user, setUser] = useState<User | null>(null);
  const [token, setToken] = useState<string | null>(null);
  const [loading, setLoading] = useState(true);

  // On mount, check AsyncStorage for existing user token
  useEffect(() => {
    async function loadUser() {
      try {
        const storedToken = await AsyncStorage.getItem('userToken');
        const storedUser = await AsyncStorage.getItem('userData');

        if (storedToken && storedUser) {
          setToken(storedToken);
          setUser(JSON.parse(storedUser));
        }
      } catch (err) {
        console.warn('Error loading user data:', err);
      } finally {
        setLoading(false);
      }
    }
    loadUser();
  }, []);

  /** Fake signIn (replace with real API call) */
  async function signIn(email: string, _password: string) {
    const fakeToken = 'abc123';
    const userData = { id: 1, email };

    setToken(fakeToken);
    setUser(userData);

    await AsyncStorage.setItem('userToken', fakeToken);
    await AsyncStorage.setItem('userData', JSON.stringify(userData));
  }

  /** Fake signUp (replace with real API) */
  async function signUp(email: string, _password: string) {
    const fakeToken = 'xyz789';
    const userData = { id: 2, email };

    setToken(fakeToken);
    setUser(userData);

    await AsyncStorage.setItem('userToken', fakeToken);
    await AsyncStorage.setItem('userData', JSON.stringify(userData));
  }

  /** Sign out clears everything */
  async function signOut() {
    setToken(null);
    setUser(null);
    await AsyncStorage.removeItem('userToken');
    await AsyncStorage.removeItem('userData');
  }

  return (
    <MyAuthContext.Provider
      value={{ user, token, loading, signIn, signUp, signOut }}
    >
      {children}
    </MyAuthContext.Provider>
  );
}
```

> **Key**: We used **`export const MyAuthContext = createContext(...)`** and **`export function AuthProvider(...)`**. There’s **no** “namespace” usage. TypeScript sees `MyAuthContext` as a normal variable.

---

## 3. Root Layout with One `<Stack>`

In `app/_layout.tsx`, we import `AuthProvider` from the context file. We do **not** import `MyAuthContext` directly unless we need it here.

```bash
touch app/_layout.tsx
```

**app/_layout.tsx**:

```tsx
import React from 'react';
import { Slot, Stack } from 'expo-router';
import { AuthProvider } from '../src/context/AuthContext';

export default function RootLayout() {
  return (
    <AuthProvider>
      {/* 
        We define exactly ONE <Stack> 
        No other <Stack> or <NavigationContainer> anywhere else 
      */}
      <Stack screenOptions={{ headerShown: true }} />
      <Slot />
    </AuthProvider>
  );
}
```

This ensures only one navigation container is used.

---

## 4. Home Screen: `app/index.tsx`

```bash
touch app/index.tsx
```

**app/index.tsx**:

```tsx
import React, { useContext } from 'react';
import { View, Text, Button, StyleSheet } from 'react-native';
import { useRouter, Link } from 'expo-router';
import { MyAuthContext } from '../src/context/AuthContext';

export default function HomeScreen() {
  const { user } = useContext(MyAuthContext);
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

## 5. Auth Screens: `(auth)/signin.tsx` & `(auth)/signup.tsx`

```bash
mkdir -p app/\(auth\)
touch app/\(auth\)/signin.tsx
touch app/\(auth\)/signup.tsx
```

### **app/(auth)/signin.tsx**

```tsx
import React, { useContext } from 'react';
import { View, Text, TextInput, Button, StyleSheet } from 'react-native';
import { Formik } from 'formik';
import * as Yup from 'yup';
import { useRouter } from 'expo-router';
import { MyAuthContext } from '../../src/context/AuthContext';

const SignInSchema = Yup.object().shape({
  email: Yup.string().email('Invalid email').required('Email is required'),
  password: Yup.string().required('Password is required'),
});

export default function SignIn() {
  const { signIn } = useContext(MyAuthContext);
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

### **app/(auth)/signup.tsx**

```tsx
import React, { useContext } from 'react';
import { View, Text, TextInput, Button, StyleSheet } from 'react-native';
import { Formik } from 'formik';
import * as Yup from 'yup';
import { useRouter } from 'expo-router';
import { MyAuthContext } from '../../src/context/AuthContext';

const SignUpSchema = Yup.object().shape({
  email: Yup.string().email('Invalid email').required('Email is required'),
  password: Yup.string().min(4, 'Too short').required('Password is required'),
  confirmPassword: Yup.string()
    .oneOf([Yup.ref('password'), ''], 'Passwords must match')
    .required('Confirm your password'),
});

export default function SignUp() {
  const { signUp } = useContext(MyAuthContext);
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

## 6. Protected Layout & Screen

```bash
mkdir -p app/\(protected\)
touch app/\(protected\)/_layout.tsx
touch app/\(protected\)/profile.tsx
```

### **app/(protected)/_layout.tsx**
```tsx
import React, { useContext } from 'react';
import { Slot, Redirect } from 'expo-router';
import { MyAuthContext } from '../../src/context/AuthContext';

export default function ProtectedLayout() {
  const { user } = useContext(MyAuthContext);

  // If no user is logged in, redirect
  if (!user) {
    return <Redirect href="/(auth)/signin" />;
  }

  // Otherwise, show protected pages
  return <Slot />;
}
```
> We do **not** define a `<Stack>` or `<NavigationContainer>` here—just `<Slot/>`.

### **app/(protected)/profile.tsx**
```tsx
import React, { useContext } from 'react';
import { View, Text, Button, StyleSheet } from 'react-native';
import { MyAuthContext } from '../../src/context/AuthContext';
import { useRouter } from 'expo-router';

export default function ProfileScreen() {
  const { user, signOut } = useContext(MyAuthContext);
  const router = useRouter();

  async function handleSignOut() {
    await signOut();
    router.replace('/');
  }

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

---

## 7. Run It

1. **Clear caches** in case of stale code:

   ```bash
   rm -rf node_modules
   npm install
   npx expo start -c
   ```

2. Open in **Expo Go** or on an emulator.  
3. **Check for errors**:

   - If TypeScript complains about “Cannot find namespace `AuthContext`,” it means you accidentally typed `AuthContext.Provider` somewhere after changing your variable name to `MyAuthContext`. Make sure you’re using `MyAuthContext.Provider` or rename your context references to match.  
   - If you see “Another navigator is already registered,” that would mean there’s a leftover `_layout.tsx` somewhere else with a `<Stack>` or a `NavigationContainer`. But in **this** tutorial, we have only one `<Stack>` in the root layout, so it shouldn’t happen.  

---

## 8. Why This Fixes “Cannot find namespace ‘AuthContext’”

- We changed the variable to `MyAuthContext` so TypeScript definitely interprets it as a normal variable, not a “namespace.”  
- We export both `MyAuthContext` and `AuthProvider` as named exports, then import them in `_layout.tsx` or other screens.  
- No default exports, no namespace references. TypeScript is happy.

---

## 9. Summary of the Approach

1. **Move** your context file out of `app/`, so it’s never auto-registered as a route.  
2. **Use** a distinctive variable name like `MyAuthContext` to avoid TS confusing it with a namespace.  
3. Have **exactly one** `<Stack>` in the root `_layout.tsx`.  
4. **No** `<Stack>` or `<NavigationContainer>` in child layouts.  
5. For protected routes, just check `user` in `(protected)/_layout.tsx` and do `<Slot />` or `<Redirect />`.  

Following these steps ensures a straightforward TypeScript + Expo Router setup with no namespace or multiple navigator conflicts. If you do see either error again, compare your code **line by line** to the snippets above. Once it matches exactly, you’ll have a fully functional app without TS or navigator problems!
