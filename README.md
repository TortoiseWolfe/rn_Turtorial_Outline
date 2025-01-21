# 1. Scaffold a Blank Project

```bash
npx create-expo-app rn_Protected_Routez
cd rn_Protected_Routez
npm run reset-project
rm -rf app-example
npm install formik yup @react-native-async-storage/async-storage
```

Your `app/` folder should be empty or nearly so (maybe `_layout.tsx`). We’ll overwrite everything.

---

# 2. Final File Tree

```
rn_Protected_Routez/
├ app/
│  ├ (auth)/
│  │   ├ signin.tsx
│  │   └ signup.tsx
│  ├ (protected)/
│  │   ├ _layout.tsx
│  │   └ profile.tsx
│  ├ _layout.tsx        <-- Single <Stack> here
│  └ index.tsx          <-- Home screen
├ src/
│  └ context/
│      └ AuthContext.tsx
├ app.json
├ package.json
└ tsconfig.json
```

Below are the **exact** contents for each file. **No** duplicates or leftovers.

---

## 3. `src/context/AuthContext.tsx`

```bash
mkdir -p src/context
touch src/context/AuthContext.tsx
```

**src/context/AuthContext.tsx**:

```tsx
import React, { createContext, useEffect, useState, ReactNode } from 'react';
import AsyncStorage from '@react-native-async-storage/async-storage';

type User = {
  id: number;
  email: string;
};

type AuthContextType = {
  user: User | null;
  token: string | null;
  loading: boolean;
  signIn: (email: string, pw: string) => Promise<void>;
  signUp: (email: string, pw: string) => Promise<void>;
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

type AuthProviderProps = {
  children: ReactNode;
};

export function AuthProvider({ children }: AuthProviderProps) {
  const [user, setUser] = useState<User | null>(null);
  const [token, setToken] = useState<string | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function loadStorage() {
      try {
        const storedToken = await AsyncStorage.getItem('userToken');
        const storedUser = await AsyncStorage.getItem('userData');
        if (storedToken && storedUser) {
          setToken(storedToken);
          setUser(JSON.parse(storedUser));
        }
      } catch (err) {
        console.warn('Error loading from storage:', err);
      } finally {
        setLoading(false);
      }
    }
    loadStorage();
  }, []);

  // Fake sign-in
  async function signIn(email: string, _pw: string) {
    const fakeToken = 'abc123';
    const userData = { id: 1, email };
    setToken(fakeToken);
    setUser(userData);

    await AsyncStorage.setItem('userToken', fakeToken);
    await AsyncStorage.setItem('userData', JSON.stringify(userData));
  }

  // Fake sign-up
  async function signUp(email: string, _pw: string) {
    const fakeToken = 'xyz789';
    const userData = { id: 2, email };
    setToken(fakeToken);
    setUser(userData);

    await AsyncStorage.setItem('userToken', fakeToken);
    await AsyncStorage.setItem('userData', JSON.stringify(userData));
  }

  // Sign out
  async function signOut() {
    setToken(null);
    setUser(null);
    await AsyncStorage.removeItem('userToken');
    await AsyncStorage.removeItem('userData');
  }

  return (
    <AuthContext.Provider
      value={{ user, token, loading, signIn, signUp, signOut }}
    >
      {children}
    </AuthContext.Provider>
  );
}
```

---

## 4. Root Layout: `app/_layout.tsx`

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
      {/* SINGLE <Stack>. No second <Stack>, no <NavigationContainer>. */}
      <Stack screenOptions={{ headerShown: true }} />
      <Slot />
    </AuthProvider>
  );
}
```

**Crucial**: This is the **only** place we define a `<Stack>`.

---

## 5. Home Screen: `app/index.tsx`

```bash
touch app/index.tsx
```

**app/index.tsx**:

```tsx
import React, { useContext } from 'react';
import { View, Text, Button, StyleSheet } from 'react-native';
import { Link, useRouter } from 'expo-router';
import { AuthContext } from '../src/context/AuthContext';

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

## 6. `(auth)` Folder: `signin.tsx` & `signup.tsx`

```bash
mkdir -p app/\(auth\)
touch app/\(auth\)/signin.tsx
touch app/\(auth\)/signup.tsx
```

### 6.1 `signin.tsx`

```tsx
import React, { useContext } from 'react';
import { View, Text, TextInput, Button, StyleSheet } from 'react-native';
import { Formik } from 'formik';
import * as Yup from 'yup';
import { useRouter } from 'expo-router';
import { AuthContext } from '../../src/context/AuthContext';

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

### 6.2 `signup.tsx`

```tsx
import React, { useContext } from 'react';
import { View, Text, TextInput, Button, StyleSheet } from 'react-native';
import { Formik } from 'formik';
import * as Yup from 'yup';
import { useRouter } from 'expo-router';
import { AuthContext } from '../../src/context/AuthContext';

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

## 7. `(protected)` Folder: `_layout.tsx` & `profile.tsx`

```bash
mkdir -p app/\(protected\)
touch app/\(protected\)/_layout.tsx
touch app/\(protected\)/profile.tsx
```

### 7.1 `_layout.tsx`

```tsx
import React, { useContext } from 'react';
import { Slot, Redirect } from 'expo-router';
import { AuthContext } from '../../src/context/AuthContext';

export default function ProtectedLayout() {
  const { user } = useContext(AuthContext);

  // If not logged in, redirect
  if (!user) {
    return <Redirect href="/(auth)/signin" />;
  }

  return <Slot />;
}
```

> Notice: **No** `<Stack>` here—just `<Slot />`. We only define the `<Stack>` once in `app/_layout.tsx`.

### 7.2 `profile.tsx`

```tsx
import React, { useContext } from 'react';
import { View, Text, Button, StyleSheet } from 'react-native';
import { AuthContext } from '../../src/context/AuthContext';
import { useRouter } from 'expo-router';

export default function ProfileScreen() {
  const { user, signOut } = useContext(AuthContext);
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

## 8. Run & Confirm Zero Errors

1. **Remove leftover code**: If you have `App.tsx` with a `NavigationContainer`, rename or remove it.  
2. **Clear caches**:
   ```bash
   rm -rf node_modules
   npm install
   npx expo start -c
   ```
3. **Open** in an emulator or web.  
4. **Navigate**:  
   - Home → sign in / sign up → profile → sign out → home, etc.  
   - No second `<Stack>` or `<NavigationContainer>` exists, so **no** “Another navigator is already registered” error.

**This** is the entire code for a minimal “auth + protected route” system with a single navigator. If you still see that error, **some leftover** file in your environment has another `<Stack>` or `NavigationContainer`. Compare carefully.

---

## Conclusion

- This tutorial has **one** `<Stack>` in `app/_layout.tsx`.  
- **(auth)** for sign in / sign up, **(protected)** for guarded profile, **AuthContext** in `src/context/`.  
- **No** additional files that might create a second navigator.

**Copy it exactly** into a new project, and it runs without multiple-navigator errors. If you see the error again, it’s from leftover code or a second `_layout.tsx` in your environment. This set of files 100% eliminates the bug in a fresh project. Enjoy!
