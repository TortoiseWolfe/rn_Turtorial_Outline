## 1. Scaffold With Your Commands

**Copy & run these commands to scaffold**:

```bash
npx create-expo-app rn_Protected_Routez
cd rn_Protected_Routez
npm run reset-project
rm -rf app-example
```

At this point, you have a fresh Expo app with TypeScript, Expo Router, and an empty `app` folder. **No need** to install or configure Expo Router again—your reset script took care of that.

If you don’t already have **Formik**, **Yup**, or **Async Storage**, install them now:

```bash
npm install formik yup @react-native-async-storage/async-storage
```

Otherwise, skip.

---

## 2. Final Folder Structure

We’ll use parentheses-based routing to separate the **auth** screens (`(auth)`) and the **protected** screens (`(protected)`). The key difference to avoid the “multiple navigators” error is that we’ll **only** place a `<Stack>` in the **root** `app/_layout.tsx`, and **no** additional stacks or containers anywhere else.

```
app/
├ (auth)/
│   ├ signin.tsx
│   └ signup.tsx
├ (protected)/
│   ├ _layout.tsx          <-- Protect everything in (protected)
│   └ profile.tsx
├ context/
│   └ AuthContext.tsx      <-- Global auth state
├ _layout.tsx              <-- Root layout (single <Stack>)
└ index.tsx                <-- Home screen
```

- `app/_layout.tsx` has **one** `<Stack>`, ensuring we only have a single navigation container.  
- `(protected)/_layout.tsx` does **not** add another `<Stack>`, it merely checks if the user is logged in (auth guard) and then renders `<Slot/>`.  
- `(auth)/signin.tsx` and `(auth)/signup.tsx` contain our sign-in and sign-up screens.  
- `context/AuthContext.tsx` holds a minimal “fake” authentication flow using Async Storage.

Following this arrangement will keep Expo Router happy and avoid the “Another navigator is already registered” error.

---

## 3. Auth Context

### 3.1 Create the AuthContext File

```bash
mkdir -p app/context
touch app/context/AuthContext.tsx
```

### **app/context/AuthContext.tsx**

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

  // Fake sign-in (replace with real backend call in production)
  const signIn = async (email: string, _password: string) => {
    const fakeToken = 'abc123';
    const userData = { id: 1, email };
    setToken(fakeToken);
    setUser(userData);

    await AsyncStorage.setItem('userToken', fakeToken);
    await AsyncStorage.setItem('userData', JSON.stringify(userData));
  };

  // Fake sign-up (replace with real backend call in production)
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

## 4. Root Layout (Single `<Stack>`)

**Important**: We only declare **one** `<Stack>` in the root layout, so we don’t trigger multiple navigator conflicts.

### 4.1 Create the Root Layout

```bash
touch app/_layout.tsx
```

### **app/_layout.tsx**

```tsx
import React from 'react';
import { Slot, Stack } from 'expo-router';
import { AuthProvider } from './context/AuthContext';

export default function RootLayout() {
  return (
    <AuthProvider>
      {/* We define ONE Stack here, so there's only a single navigation container */}
      <Stack screenOptions={{ headerShown: true }} />
      {/* Slot renders the active route (index, (auth), (protected), etc.) */}
      <Slot />
    </AuthProvider>
  );
}
```

> Note that we **do not** put another `<Stack>` in `(protected)/_layout.tsx`. That’s what can cause the “multiple navigator” error.

---

## 5. Home Screen: `app/index.tsx`

A basic home screen. If the user is logged in, show a “Welcome” message and a button to Profile. Otherwise, show links to sign in / sign up.

```bash
touch app/index.tsx
```

### **app/index.tsx**

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

## 6. Public Auth Screens: `(auth)/signin.tsx` & `(auth)/signup.tsx`

We’ll group sign-in and sign-up in a folder named `(auth)`. The parentheses mean it won’t show up in the URL path, just like Next.js route groups.

### 6.1 Create the `(auth)` Folder & Screens

```bash
mkdir -p app/\(auth\)
touch app/\(auth\)/signin.tsx
touch app/\(auth\)/signup.tsx
```

#### **app/(auth)/signin.tsx**

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

#### **app/(auth)/signup.tsx**

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

We do **not** add another `<Stack>` here. Instead, we simply check if the user is logged in. If not, redirect to `/(auth)/signin`. If yes, render `<Slot/>` to show protected screens.

### 7.1 Create the `(protected)` Folder & Files

```bash
mkdir -p app/\(protected\)
touch app/\(protected\)/_layout.tsx
touch app/\(protected\)/profile.tsx
```

#### **app/(protected)/_layout.tsx**

```tsx
import React, { useContext } from 'react';
import { AuthContext } from '../context/AuthContext';
import { Slot, Redirect } from 'expo-router';

export default function ProtectedLayout() {
  const { user } = useContext(AuthContext);

  // If user is NOT logged in, redirect to (auth)/signin
  if (!user) {
    return <Redirect href="/(auth)/signin" />;
  }

  // Otherwise, render protected screens
  return <Slot />;
}
```

#### **app/(protected)/profile.tsx**

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
    // After sign-out, go back to home
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

> Because `(protected)/_layout.tsx` checks for `user` before rendering `<Slot/>`, any screen placed in `(protected)` is automatically protected. Visiting `/(protected)/profile` logs you in if you have a user, otherwise it redirects you to `/(auth)/signin`.

---

## 8. Test & Verify (No Multiple Navigator Error)

1. **Start**:
   ```bash
   npm start
   ```
   or
   ```bash
   npx expo start
   ```

2. **Home**:  
   - Open `/`. If not logged in, see “Home Screen” → links to sign in / sign up.  
   - If logged in, “Welcome back, [user.email]!” plus a button to go to `/profile`.

3. **Sign In** (`/(auth)/signin`):  
   - Provide an email/password. On success, you’ll land at `/(protected)/profile`.

4. **Profile** (`/(protected)/profile`):  
   - If you attempt to visit it while logged out, `_layout.tsx` in `(protected)` redirects to `/(auth)/signin`.
   - Logged in? See your user email and a sign-out button.

**You will not** see the “Another navigator is already registered” error because:

- We have **only one** `<Stack>` in the root layout (`app/_layout.tsx`).  
- Child layout `_layout.tsx` files just do `<Slot/>`, no nested stacks.  
- No `<NavigationContainer>` usage anywhere else—Expo Router manages all that under the hood.

---

## 9. Production Tips

- **Real Backend**: Replace the `signIn` / `signUp` mocks with actual API calls, token validation, etc.  
- **Secure Storage**: For extra security, consider `expo-secure-store` instead of `@react-native-async-storage/async-storage`.  
- **Handle Token Expiry**: If your tokens expire, add refresh logic or re-check token validity on app load.  
- **Error Handling**: In production, display user-friendly errors on sign-in failure.  
- **Testing**: Use unit tests for the AuthContext logic and integration tests for sign-in flows.

---

# Complete Functional Tutorial

By following the **exact** file structure and code above, you’ll have:

1. A **single** navigation container (the `<Stack>` in the root layout).  
2. **AuthContext** to store user & token, plus `signIn`, `signUp`, and `signOut` methods.  
3. **(auth)** folder for public routes (Sign In & Sign Up).  
4. **(protected)** folder for secure routes.  
   - `_layout.tsx` checks `user` to guard all screens.  
5. **No** “Another navigator is already registered” error, since we only define one `<Stack>` at the top level.

**Enjoy your production-ready, Next.js-style Expo Router app**—no extra navigation containers required!
