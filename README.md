## 1. Brand-New Project

```bash
npx create-expo-app rn_Protected_Routez
cd rn_Protected_Routez
npm run reset-project
rm -rf app-example
```
At this point, your `app` folder is basically empty (maybe has `_layout.tsx`). We’ll overwrite that anyway.

If you need **Formik**, **Yup**, or **Async Storage**:
```bash
npm install formik yup @react-native-async-storage/async-storage
```

---

## 2. Final File Structure

Create a new folder `src/context` (or any name) **outside** `app/`. Then your `app/` folder only contains route files. This way, **Expo Router** won’t parse your context as a screen.

```
rn_Protected_Routez/
├ app/
│  ├ (auth)/
│  │   ├ signin.tsx
│  │   └ signup.tsx
│  ├ (protected)/
│  │   ├ _layout.tsx
│  │   └ profile.tsx
│  ├ _layout.tsx
│  └ index.tsx
├ src/
│  └ context/
│      └ AuthContext.ts
├ package.json
└ etc.
```

Note:
- We renamed `AuthContext.tsx` → `AuthContext.ts`, placed in `src/context/`.  
- `(auth)` and `(protected)` have only their screens + `_layout.tsx`.  
- The root `app/_layout.tsx` has exactly one `<Stack>`.  

This ensures **only** route files are in `app/`, and **no** extraneous `.tsx` files that might get registered as routes.

---

## 3. Create the Auth Context

### 3.1 `src/context/AuthContext.ts`

```bash
mkdir -p src/context
touch src/context/AuthContext.ts
```

**src/context/AuthContext.ts**:
```ts
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

export function AuthProvider({ children }: Props) {
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

We used `.ts` (no `x`), so **Expo Router** will ignore it as a route even if it’s in the `app` folder. But we placed it outside `app` anyway, for clarity.

---

## 4. Root Layout (Single `<Stack>`)

**app/_layout.tsx** (overwriting any existing one):

```bash
touch app/_layout.tsx
```

```tsx
import React from 'react';
import { Slot, Stack } from 'expo-router';
import { AuthProvider } from '../src/context/AuthContext';

export default function RootLayout() {
  return (
    <AuthProvider>
      {/* 
        Single <Stack> 
        No other <Stack> or <NavigationContainer> anywhere else 
      */}
      <Stack screenOptions={{ headerShown: true }} />
      <Slot />
    </AuthProvider>
  );
}
```

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

## 6. Auth Screens: `(auth)/signin.tsx` & `(auth)/signup.tsx`

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

### **app/(auth)/signup.tsx**
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

## 7. Protected Layout & Screen

```bash
mkdir -p app/\(protected\)
touch app/\(protected\)/_layout.tsx
touch app/\(protected\)/profile.tsx
```

### **app/(protected)/_layout.tsx**
```tsx
import React, { useContext } from 'react';
import { Slot, Redirect } from 'expo-router';
import { AuthContext } from '../../src/context/AuthContext';

export default function ProtectedLayout() {
  const { user } = useContext(AuthContext);

  if (!user) {
    return <Redirect href="/(auth)/signin" />;
  }

  return <Slot />;
}
```
> No `<Stack>` or `<NavigationContainer>` here—just `<Slot/>`.

### **app/(protected)/profile.tsx**
```tsx
import React, { useContext } from 'react';
import { View, Text, Button, StyleSheet } from 'react-native';
import { AuthContext } from '../../src/context/AuthContext';
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

---

## 8. Test It (No Multiple Navigator Error)

1. **Clear caches**:
   ```bash
   rm -rf node_modules
   npm install
   npx expo start -c
   ```
2. **Open** the app in Expo Go or an emulator.  
3. **Home**: If not logged in, see “Home Screen” → Sign In / Sign Up.  
4. **Sign In** / **Sign Up**: On success, you’ll land at `/profile` (which is `/(protected)/profile`).  
5. **Protected**: If not logged in, `_layout.tsx` in `(protected)` automatically redirects to `/(auth)/signin`. Logged in? See your user, can sign out.

**No** “Another navigator is already registered” error, because:

- `AuthContext.ts` is outside `app/`, so Expo Router doesn’t parse it as a route.  
- We have **only one** `<Stack>` in `app/_layout.tsx`.  
- No `NavigationContainer` or additional stacks anywhere.  

If you **still** see the same error after following these steps in a brand-new project, then you likely have:

1. Another leftover `_layout.tsx` in `(auth)` or `(protected)` that we didn’t remove.  
2. Another file with `.tsx` in `app/` that’s missing a default export, or that tries to define another navigator.  
3. Mixed code from older React Navigation setups (`NavigationContainer`).  
4. Additional custom scripts that modify the router.

---

## 9. Production Tips

- Replace fake signIn/signUp with real API calls.  
- For extra security, use `expo-secure-store` for tokens.  
- Add error handling for sign-in failures.  
- Test thoroughly on Android, iOS, and Web.

**Done!** With this approach:

- Your `AuthContext.ts` is **outside** the `app` folder, so it’s never treated like a route.  
- You have exactly one `<Stack>` in the root layout, so no “multiple navigators.”  
- `(protected)/_layout.tsx` just checks `user` and returns `<Slot/>`.  

No leftover route definitions means **no** “Another navigator is already registered for this container.” Enjoy your bulletproof Next.js-style routing with Expo Router!
