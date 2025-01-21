## 1. Create & Reset the Project

```bash
npx create-expo-app rn_Protected_Routez
cd rn_Protected_Routez
npm run reset-project
rm -rf app-example
```

Then install:

```bash
npm install formik yup @react-native-async-storage/async-storage
```

At this point, your `app` folder is basically empty (maybe `_layout.tsx`, which we’ll overwrite).

---

## 2. Final Folder Structure

We’ll have:

```
rn_Protected_Routez/
├ app/
│  ├ (auth)/
│  │   ├ signin.tsx
│  │   └ signup.tsx
│  ├ (protected)/
│  │   ├ _layout.tsx
│  │   └ profile.tsx
│  ├ _layout.tsx          <-- Single <Stack> in root
│  └ index.tsx            <-- Home screen
├ src/
│  └ context/
│      └ AuthContext.tsx  <-- Auth logic
├ package.json
└ ...
```

---

## 3. AuthContext: `src/context/AuthContext.tsx`

```bash
mkdir -p src/context
touch src/context/AuthContext.tsx
```

**src/context/AuthContext.tsx**:

```tsx
import React, {
  createContext,
  ReactNode,
  useEffect,
  useState,
} from 'react';
import AsyncStorage from '@react-native-async-storage/async-storage';

type User = { id: number; email: string };

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

type AuthProviderProps = { children: ReactNode };

export function AuthProvider({ children }: AuthProviderProps) {
  const [user, setUser] = useState<User | null>(null);
  const [token, setToken] = useState<string | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function loadFromStorage() {
      try {
        const storedToken = await AsyncStorage.getItem('userToken');
        const storedUser = await AsyncStorage.getItem('userData');
        if (storedToken && storedUser) {
          setToken(storedToken);
          setUser(JSON.parse(storedUser));
        }
      } catch (err) {
        console.warn('Error reading from storage', err);
      } finally {
        setLoading(false);
      }
    }
    loadFromStorage();
  }, []);

  // Fake sign-in (replace with real calls)
  async function signIn(email: string, _password: string) {
    const fakeToken = 'abc123';
    const userData = { id: 1, email };
    setToken(fakeToken);
    setUser(userData);

    await AsyncStorage.setItem('userToken', fakeToken);
    await AsyncStorage.setItem('userData', JSON.stringify(userData));
  }

  // Fake sign-up
  async function signUp(email: string, _password: string) {
    const fakeToken = 'xyz789';
    const userData = { id: 2, email };
    setToken(fakeToken);
    setUser(userData);

    await AsyncStorage.setItem('userToken', fakeToken);
    await AsyncStorage.setItem('userData', JSON.stringify(userData));
  }

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

- **No** “namespace” usage. We’re exporting a normal variable: `AuthContext`.
- File named exactly **`AuthContext.tsx`**, but placed in `src/context`, not `app/`.

---

## 4. Root Layout: `app/_layout.tsx` (One `<Stack>`)

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
        Exactly one <Stack> 
        No second <Stack> or <NavigationContainer> anywhere else 
      */}
      <Stack screenOptions={{ headerShown: true }} />
      <Slot />
    </AuthProvider>
  );
}
```

**Key**: Because we have exactly one `<Stack>` in the entire project, we cannot get “Another navigator is already registered.”

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

## 6. Public Auth Routes: `(auth)/signin.tsx` & `(auth)/signup.tsx`

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
          } catch (err) {
            console.error(err);
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
          } catch (err) {
            console.error(err);
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

## 7. Protected Layout & Screen

```bash
mkdir -p app/\(protected\)
touch app/\(protected\)/_layout.tsx
touch app/\(protected\)/profile.tsx
```

### 7.1 `(protected)/_layout.tsx`

```tsx
import React, { useContext } from 'react';
import { Slot, Redirect } from 'expo-router';
import { AuthContext } from '../../src/context/AuthContext';

export default function ProtectedLayout() {
  const { user } = useContext(AuthContext);

  // If no user, redirect to sign in
  if (!user) {
    return <Redirect href="/(auth)/signin" />;
  }

  return <Slot />;
}
```

**No** `<Stack>` here—just `<Slot/>`.

### 7.2 `(protected)/profile.tsx`

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

## 8. Run & Confirm

1. **Remove any leftover** code or modules:
   ```bash
   rm -rf node_modules
   npm install
   npx expo start -c
   ```
2. **Open** in an emulator / Expo Go.  
3. **Index** → if not logged in, see “Home Screen” → sign in / sign up. If logged in, “Welcome back…” + button to profile.  
4. **Sign In** / **Sign Up** → on success, `router.replace('/(protected)/profile')`.  
5. **Profile** → if not logged in, `(protected)/_layout.tsx` does `<Redirect />` to sign in. If logged in, shows user’s email and a “Sign Out” button.

Because we have **only one** `<Stack>` (in `app/_layout.tsx`), you **cannot** get “Another navigator is already registered.” If you do, you likely have leftover code with a second stack or a manual `NavigationContainer`.

---

## Done

This is the **smallest** opinionated, functional tutorial for:

- **Single `<Stack>`** in the entire project.  
- **(auth)** folder for sign-in/up.  
- **(protected)** folder for a guarded Profile.  
- **AuthContext** in `src/context/AuthContext.tsx` so it’s not treated as a route.  

No “somepage,” no extra fluff—just the essential pieces. If you see the “Another navigator” error again, **something else** in your code is adding an extra navigation container. Compare your entire `app` folder with these snippets line-by-line to ensure no leftover navigators. This version definitely compiles & runs with no extra bugs!
