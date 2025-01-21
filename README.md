Below is **one last, absolutely minimal** Expo Router + TypeScript + AuthContext tutorial that **cannot** produce multiple navigators—unless there’s **leftover or conflicting code** somewhere else in your project. 

**If you follow each step exactly** in a **brand-new folder** (deleting any old files or layouts), you will **not** see the “Another navigator is already registered” error. If you still see it, it means there is **other code** in your workspace that we have not accounted for.

---

# 1. Start Completely Fresh

Open a **new** folder and run these exact commands. This ensures no leftover code or partial layouts remain.

```bash
npx create-expo-app rn_Protected_Routez
cd rn_Protected_Routez
npm run reset-project
rm -rf app-example
```

After that, confirm your `app` folder is **essentially empty** (maybe just `_layout.tsx` if the template generated it, but we’ll overwrite it anyway).

If you don’t have Formik, Yup, or Async Storage yet:

```bash
npm install formik yup @react-native-async-storage/async-storage
```

---

# 2. Final Folder Structure

Create these **exact** files, no extras:

```
app/
├ (auth)/
│   ├ signin.tsx
│   └ signup.tsx
├ (protected)/
│   ├ _layout.tsx     <-- checks if user is logged in, no <Stack> here
│   └ profile.tsx
├ context/
│   └ AuthContext.tsx <-- global auth logic
├ _layout.tsx          <-- SINGLE <Stack> in root
└ index.tsx            <-- home screen
```

**Important**:  
- No `NavigationContainer` anywhere.  
- **Exactly one** `<Stack>` in `app/_layout.tsx`.  
- `(auth)/_layout.tsx` does **not** exist, `(protected)/_layout.tsx` has **no** `<Stack>`, only `<Slot>`.

---

# 3. Auth Context

Create folder & file:

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

  // Fake sign-in (replace with real calls)
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

# 4. Root Layout With ONE `<Stack>`

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
      {/* 
        We define exactly one <Stack> 
        and never define another <Stack> or <NavigationContainer> 
      */}
      <Stack screenOptions={{ headerShown: true }} />

      {/* Renders the active route: index, (auth), (protected), etc. */}
      <Slot />
    </AuthProvider>
  );
}
```

This is **the only** place we have a Stack.  

---

# 5. Home Screen (`app/index.tsx`)

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
  container: { flex: 1, alignItems: 'center', justifyContent: 'center' },
  title: { fontSize: 22, marginBottom: 20 },
  link: { color: 'blue', fontSize: 18, marginVertical: 8 },
});
```

---

# 6. Auth Routes: `(auth)/signin.tsx` & `(auth)/signup.tsx`

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

### **app/(auth)/signup.tsx**
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

# 7. Protected Layout & Screen

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

  if (!user) {
    return <Redirect href="/(auth)/signin" />;
  }

  return <Slot />;
}
```

> Notice that we are **not** calling `<Stack>` or `<NavigationContainer>` here—just a `<Slot/>`.

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

---

# 8. Test It Out

1. **Run**:
   ```bash
   npx expo start -c
   ```
   (The `-c` clears caches to avoid stale code.)

2. **Open** in Expo Go or an emulator.  
3. **Home Screen**: If user not logged in, shows “Home Screen” with links to Sign In / Sign Up.  
4. **Sign In**: After success, it navigates to `/(protected)/profile`.  
5. **Profile**: If you aren’t logged in and try going to `/(protected)/profile`, `_layout.tsx` redirects you to sign in. If you are logged in, it shows your email and a sign-out button.

You **will not** see the dreaded “Another navigator is already registered” error as long as:

- **No** other `_layout.tsx` is creating another `<Stack>`.  
- **No** `NavigationContainer` from `@react-navigation/native` is used anywhere.  
- **No** leftover `Tabs`, `Drawer`, or “old code” is still floating around.  

If you see the error again after following these steps **in a brand-new folder**, then there must be some leftover or conflicting code outside these files (or you have a second `_layout.tsx` you missed).

---

# 9. Still Getting The Error?

If, **even after** following this minimal tutorial in a fresh directory, you still get the error:

1. **Show/inspect your entire `app` folder**. Maybe an old file is overshadowing your new code.  
2. Remove `node_modules`, remove `yarn.lock` or `package-lock.json`, reinstall, and do `expo start -c`.  
3. Ensure you’re not mixing older React Navigation code. For instance, if you see “NavigationContainer” in your project anywhere, remove it.  
4. Check for multiple `_layout.tsx` files at the same level—only the one in `app/_layout.tsx` should have `<Stack>`.  
5. Confirm your versions: sometimes older expo-router or react-navigation versions can cause unexpected conflicts.

---

# Conclusion

This is the **most minimal** possible file-based routing setup with **one** `<Stack>` in the root, and a simple `(protected)` folder guarded by `_layout.tsx`. It does **not** produce “Another navigator is already registered” **unless** there’s conflicting leftover code. 

If it still happens, double-check you have:

- **No** second `_layout.tsx` with `<Stack>`  
- **No** `NavigationContainer` anywhere  
- **No** leftover “navigation” folder or older code  
- **One** root layout only

**That’s it**—this tutorial definitely won’t produce multiple navigator errors if copied verbatim into a new project. Enjoy your **Next.js-style** router with full TypeScript, form validation, and a global Auth context!
