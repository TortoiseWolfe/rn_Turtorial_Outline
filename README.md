
## 1. Scaffold with Your Commands

**Copy & run these commands to scaffold**:

```bash
npx create-expo-app rn_Protected_Routez
cd rn_Protected_Routez
npm run reset-project
rm -rf app-example
```

At this point, you have a fresh Expo app with TypeScript, Expo Router, and an empty `app` folder. **No need** to reconfigure TS or install Expo Router because your reset script already sets it all up.

If you haven’t added **Formik**, **Yup**, or **Async Storage**, run:
```bash
npm install formik yup @react-native-async-storage/async-storage
```

---

## 2. Project Structure

We’ll use **(auth)** for public auth screens (Sign In, Sign Up) and **(protected)** for restricted screens (Profile). The `app/_layout.tsx` wraps **everything** in an `AuthProvider`. Then `(protected)/_layout.tsx` checks if the user is logged in before rendering the sub-routes.

A final structure:

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
├ _layout.tsx          <-- Root layout
└ index.tsx            <-- Home screen
```

Below, we’ll **create** each file with `touch` and then **paste** the code.

---

## 3. Global Auth Context

We need a `context/AuthContext.tsx` to manage:
- `user` and `token`
- `signIn`, `signUp`, `signOut`
- A `loading` state while we check storage

### 3.1 Create & Paste

```bash
mkdir -p app/context
touch app/context/AuthContext.tsx
```

Now **paste**:

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

We wrap **all** routes with `AuthProvider`. `_layout.tsx` is like Next.js’s `_app.tsx`.

### 4.1 Create & Paste

```bash
touch app/_layout.tsx
```

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

A simple welcome page that either greets the user or links to the auth screens.

```bash
touch app/index.tsx
```

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

### 6.1 Create the `(auth)` folder and screens

```bash
mkdir -p app/\(auth\)
touch app/\(auth\)/signin.tsx
touch app/\(auth\)/signup.tsx
```

#### `signin.tsx`:

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

#### `signup.tsx`:

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

### 7.1 Create the `(protected)` folder & files

```bash
mkdir -p app/\(protected\)
touch app/\(protected\)/_layout.tsx
touch app/\(protected\)/profile.tsx
```

### 7.2 `_layout.tsx` (the actual “protected route” logic)

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

### 7.3 `profile.tsx` (example protected screen)

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

> Since `(protected)/_layout.tsx` **automatically** redirects if there’s no user, you don’t need to do extra checks in `profile.tsx`. The parentheses in the folder name `(protected)` ensures it **won’t** appear in the URL. So visiting `/profile` effectively is `/(protected)/profile`.

---

## 8. Testing & Summary

1. **Run**:
   ```bash
   npm start
   ```
   (or `npx expo start`)

2. **Home**: (`/`)  
   - If not logged in, you’ll see “Home Screen” → links to sign in/up.  
   - If logged in, you see “Welcome back, [email]!” and a button to Profile.

3. **Sign In**: (`/(auth)/signin`)  
   - After signing in, you’re redirected to `/(protected)/profile`.  

4. **Protected**: (`/(protected)/profile`)  
   - If you try to visit this link while logged out, `_layout.tsx` redirects you to `/(auth)/signin`.  
   - Once logged in, you see “Logged in as: [email]” and “Sign Out.”

This approach **matches** Next.js-like routing. Your `(protected)/_layout.tsx` is the **official** “guard” for protected screens.

### Production Notes

- Replace the **fake** `signIn`/`signUp` logic with real API calls.  
- For more secure token storage, consider `expo-secure-store`.  
- Handle token refresh if using short-lived access tokens.

**That’s it!** You now have a **modern, Next.js-style** Expo Router app with parentheses for protected routes, TypeScript, form validation, and a global Auth context—starting from the **exact** commands you provided. Enjoy building!
