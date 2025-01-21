## 1. Scaffold with Your Commands

**Copy & run these commands to scaffold**:

```bash
npx create-expo-app rn_Protected_Routez
cd rn_Protected_Routez
npm run reset-project
rm -rf app-example
```

At this point, you have a fresh Expo app with TypeScript, Expo Router, and an empty `app` folder. There’s **no need** to reconfigure TS or install Expo Router because your reset script already sets it up.

If you don’t have **Formik**, **Yup**, or **Async Storage** yet, install them:
```bash
npm install formik yup @react-native-async-storage/async-storage
```

Otherwise, skip.

---

## 2. Project Structure

We’ll structure it like Next.js:

```
app/
├ (auth)/
│   ├ signin.tsx
│   ├ signup.tsx
├ (protected)/
│   ├ _layout.tsx     <-- Protect all screens in (protected) folder
│   └ profile.tsx
├ context/
│   └ AuthContext.tsx <-- Global auth state
├ _layout.tsx          <-- Root layout (wrap everything in AuthProvider)
└ index.tsx            <-- Home screen
```

1. `app/_layout.tsx` wraps the entire app with `<AuthProvider>`.  
2. `(auth)` is for sign-in / sign-up.  
3. `(protected)` is for any restricted screen (like Profile).  
4. `context/AuthContext.tsx` holds your shared Auth logic.

---

## 3. Global Auth Context

We need a file `app/context/AuthContext.tsx` that:
- Stores `user`, `token`, and a `loading` state.
- Provides `signIn`, `signUp`, and `signOut` methods.
- Checks Async Storage at startup to persist sessions.

### 3.1 Create & Paste

```bash
mkdir -p app/context
touch app/context/AuthContext.tsx
```

**app/context/AuthContext.tsx**:
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

  // Fake sign-in (replace with real backend call)
  const signIn = async (email: string, _password: string) => {
    const fakeToken = 'abc123';
    const userData = { id: 1, email };
    setToken(fakeToken);
    setUser(userData);

    await AsyncStorage.setItem('userToken', fakeToken);
    await AsyncStorage.setItem('userData', JSON.stringify(userData));
  };

  // Fake sign-up (replace with real backend call)
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

## 4. Root Layout: `app/_layout.tsx`

We wrap **all** routes with `AuthProvider`. `_layout.tsx` is like `_app.tsx` in Next.js.

### 4.1 Create & Paste

```bash
touch app/_layout.tsx
```

**app/_layout.tsx**:
```tsx
import React from 'react';
import { Slot, Stack } from 'expo-router';
import { AuthProvider } from './context/AuthContext';

export default function RootLayout() {
  return (
    <AuthProvider>
      {/* Optional: customize your Stack UI */}
      <Stack screenOptions={{ headerShown: true }} />
      {/* Slot renders whatever route is active (index, (auth), (protected), etc.) */}
      <Slot />
    </AuthProvider>
  );
}
```

---

## 5. Home Screen: `app/index.tsx`

The “home” route that either greets the user or shows links to sign in / sign up.

```bash
touch app/index.tsx
```

**app/index.tsx**:
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
          <Button title="Go to Profile" onPress={() => router.push('/(protected)/profile')} />
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
  container: {
    flex: 1, 
    alignItems: 'center',
    justifyContent: 'center',
  },
  title: {
    fontSize: 22, 
    marginBottom: 20,
  },
  link: {
    color: 'blue',
    fontSize: 18,
    marginVertical: 8,
  },
});
```

---

## 6. Public Auth Routes: `(auth)/signin.tsx` & `(auth)/signup.tsx`

We’ll group sign-in and sign-up under `(auth)`. (Parentheses means it’s a separate route group that won’t appear in the URL path.)

### 6.1 Create the `(auth)` folder and screens

```bash
mkdir -p app/\(auth\)
touch app/\(auth\)/signin.tsx
touch app/\(auth\)/signup.tsx
```

#### **app/(auth)/signin.tsx**:
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
  container: {
    flex: 1, 
    alignItems: 'center',
    justifyContent: 'center',
  },
  header: {
    fontSize: 24, 
    marginBottom: 20,
  },
  form: {
    width: '80%',
  },
  input: {
    backgroundColor: '#eee',
    marginVertical: 6,
    padding: 10,
    borderRadius: 6,
  },
  error: {
    color: 'red',
    marginBottom: 5,
  },
});
```

#### **app/(auth)/signup.tsx**:
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
  container: {
    flex: 1, 
    alignItems: 'center',
    justifyContent: 'center',
  },
  header: {
    fontSize: 24, 
    marginBottom: 20,
  },
  form: {
    width: '80%',
  },
  input: {
    backgroundColor: '#eee',
    marginVertical: 6,
    padding: 10,
    borderRadius: 6,
  },
  error: {
    color: 'red',
    marginBottom: 5,
  },
});
```

---

## 7. Protected Routes: `(protected)/_layout.tsx` & `(protected)/profile.tsx`

We guard these routes in `(protected)/_layout.tsx`. If no user, we redirect to `(auth)/signin`.

### 7.1 Create the `(protected)` folder & files

```bash
mkdir -p app/\(protected\)
touch app/\(protected\)/_layout.tsx
touch app/\(protected\)/profile.tsx
```

#### **app/(protected)/_layout.tsx**:
```tsx
import React, { useContext } from 'react';
import { AuthContext } from '../context/AuthContext';
import { Slot, Redirect } from 'expo-router';

export default function ProtectedLayout() {
  const { user } = useContext(AuthContext);

  // If user not logged in, redirect to (auth)/signin
  if (!user) {
    return <Redirect href="/(auth)/signin" />;
  }

  // Otherwise, render the protected screens
  return <Slot />;
}
```

#### **app/(protected)/profile.tsx**:
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
    // Go back to home on sign out
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
  container: {
    flex: 1, 
    alignItems: 'center',
    justifyContent: 'center',
  },
  header: {
    fontSize: 24, 
    marginBottom: 10,
  },
  info: {
    fontSize: 16,
    marginBottom: 20,
  },
});
```

> Since `(protected)/_layout.tsx` **automatically** checks for a user, each protected screen is safe. The parentheses in the folder name `(protected)` means it’s grouped, but not in the visible URL path (like Next.js). So visiting `/profile` effectively is `/(protected)/profile`.

---

## 8. Testing & Summary

1. **Run**:
   ```bash
   npm start
   ```
   or 
   ```bash
   npx expo start
   ```
2. **Home**: (`/`)
   - If not logged in, you see “Home Screen” with “Sign In” / “Sign Up” links.
   - If logged in, you see “Welcome back, [user.email]!” and a button to Profile.
3. **Sign In**: (`/(auth)/signin`)
   - After sign in, you’re redirected to `/(protected)/profile`.
4. **Protected**: (`/(protected)/profile`)
   - If you try to visit this link when not logged in, `_layout.tsx` redirects to `/(auth)/signin`.
   - Once logged in, you see “Logged in as: [email]” and can sign out.

### Production Tips

- Replace the **fake** sign-in / sign-up calls with real API logic.  
- Consider `expo-secure-store` for token storage if you need more security.  
- Handle token refresh if tokens expire.  
- Add robust error handling / user feedback.

**Done!** You have a **Next.js-style** Expo Router setup with:

- **TypeScript** (already configured).
- **AuthContext** for global auth state.
- **(auth)** folder for sign in / sign up.
- **(protected)** folder for guarded routes (Profile).
- Form validation with **Formik + Yup**.

Enjoy your production-ready protected routes!
