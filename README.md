# 1. New Project & Basic Housekeeping

```bash
npx create-expo-app rn_Demo
cd rn_Demo
npm run reset-project
rm -rf app-example
```

At this point, your `app` folder should be nearly empty, but **Expo Router** is included out of the box (no extra install needed). 

---

# 2. Single Top-Level Layout

You only need **one** top-level layout in `app/_layout.tsx`. It wraps everything in an `AuthProvider` plus a single `<Stack>`.

**Terminal**:
```bash
mkdir context
touch context/auth.tsx
mkdir -p app/\(auth\)
touch app/\(auth\)/_layout.tsx
touch app/\(auth\)/sign-in.tsx
touch app/\(auth\)/sign-up.tsx
mkdir -p app/\(protected\)
touch app/\(protected\)/_layout.tsx
touch app/\(protected\)/profile.tsx
touch app/index.tsx   # optional index
code .
```

## File: `app/_layout.tsx`
```tsx
// app/_layout.tsx
import { Stack } from "expo-router";
import AuthProvider from "../context/auth";

export default function RootLayout() {
  return (
    <AuthProvider>
      {/* 
        The single top-level <Stack> for the entire app. 
        Expo Router automatically provides NavigationContainer.
      */}
      <Stack
        screenOptions={{
          headerShown: false,
        }}
      />
    </AuthProvider>
  );
}
```

---

# 3. Auth Context

This **context** holds:
- A `authToken` (fake JWT or user token).
- Async `signIn`, `signUp` methods (simulate API calls).
- A `signOut` method.
- `loading` + `error` states for UI feedback.

## File: `context/auth.tsx`
```tsx
// context/auth.tsx
import React, { createContext, useContext, useState } from "react";

// Types
type AuthToken = string | null;

interface AuthContextProps {
  authToken: AuthToken;
  loading: boolean;
  error: string | null;
  signUp: (email: string, password: string) => Promise<void>;
  signIn: (email: string, password: string) => Promise<void>;
  signOut: () => void;
}

const AuthContext = createContext<AuthContextProps>({
  authToken: null,
  loading: false,
  error: null,
  signUp: async () => {},
  signIn: async () => {},
  signOut: () => {},
});

export default function AuthProvider({ children }: { children: React.ReactNode }) {
  const [authToken, setAuthToken] = useState<AuthToken>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  // Fake endpoints for demonstration
  const FAKE_SIGNUP_ENDPOINT = "https://example.com/api/signup";
  const FAKE_SIGNIN_ENDPOINT = "https://example.com/api/signin";

  async function signUp(email: string, password: string) {
    setLoading(true);
    setError(null);
    try {
      // In a real app, you'd do:
      // const response = await fetch(FAKE_SIGNUP_ENDPOINT, { ... });
      // if (!response.ok) throw new Error("Sign up failed");
      // const data = await response.json();
      // setAuthToken(data.token);

      // Simulate success after 1 second
      await new Promise((resolve) => setTimeout(resolve, 1000));
      setAuthToken("fake_signup_token");
    } catch (err: any) {
      setError(err.message || "Sign up failed");
    } finally {
      setLoading(false);
    }
  }

  async function signIn(email: string, password: string) {
    setLoading(true);
    setError(null);
    try {
      // Real app example:
      // const response = await fetch(FAKE_SIGNIN_ENDPOINT, { ... });
      // if (!response.ok) throw new Error("Sign in failed");
      // const data = await response.json();
      // setAuthToken(data.token);

      // Simulate success
      await new Promise((resolve) => setTimeout(resolve, 1000));
      setAuthToken("fake_signin_token");
    } catch (err: any) {
      setError(err.message || "Sign in failed");
    } finally {
      setLoading(false);
    }
  }

  function signOut() {
    setAuthToken(null);
    setError(null);
  }

  return (
    <AuthContext.Provider
      value={{
        authToken,
        loading,
        error,
        signUp,
        signIn,
        signOut,
      }}
    >
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  return useContext(AuthContext);
}
```

---

# 4. Auth Routes: `(auth)` Folder

We’ll create an **auth flow** in `app/(auth)`. Each file has **email + password** fields, client-side validation, and then uses a more reliable approach (watching `authToken`) to navigate.

## File: `app/(auth)/_layout.tsx`
```tsx
// app/(auth)/_layout.tsx
import { Stack } from "expo-router";

export default function AuthLayout() {
  return (
    <Stack
      screenOptions={{
        headerTitle: "Auth Flow",
        headerShown: true,
      }}
    />
  );
}
```

## File: `app/(auth)/sign-in.tsx`
```tsx
// app/(auth)/sign-in.tsx
import { View, Text, Button, TextInput, StyleSheet } from "react-native";
import { useState, useEffect } from "react";
import { Link, useRouter } from "expo-router";
import { useAuth } from "../../context/auth";

export default function SignInScreen() {
  const { signIn, loading, error, authToken } = useAuth();
  const router = useRouter();

  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [localError, setLocalError] = useState("");

  // ----> The "more reliable" method: watch authToken directly.
  useEffect(() => {
    // If user is successfully logged in, go to the protected screen.
    if (authToken) {
      router.replace("(protected)/profile");
    }
  }, [authToken]);

  async function handleSignIn() {
    // Basic client-side validation
    if (!validateEmail(email)) {
      setLocalError("Invalid email format");
      return;
    }
    if (password.length < 8) {
      setLocalError("Password must be at least 8 characters");
      return;
    }

    setLocalError("");
    await signIn(email, password);
    // We don't navigate here; we rely on useEffect to see if authToken was set.
  }

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Sign In</Text>

      {/* Show local client errors first */}
      {localError ? <Text style={styles.error}>{localError}</Text> : null}

      {/* Show server or signIn errors from Auth Context */}
      {error ? <Text style={styles.error}>{error}</Text> : null}

      <TextInput
        style={styles.input}
        placeholder="Email"
        autoCapitalize="none"
        onChangeText={setEmail}
        value={email}
      />

      <TextInput
        style={styles.input}
        placeholder="Password (min 8 chars)"
        secureTextEntry
        onChangeText={setPassword}
        value={password}
      />

      <Button
        title={loading ? "Signing In..." : "Sign In"}
        onPress={handleSignIn}
      />

      <Link href="/(auth)/sign-up" style={styles.link}>
        Don’t have an account? Sign Up
      </Link>
    </View>
  );
}

function validateEmail(email: string) {
  return /\S+@\S+\.\S+/.test(email);
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 20, justifyContent: "center" },
  title: { fontSize: 24, fontWeight: "bold", marginBottom: 20 },
  input: {
    borderWidth: 1,
    borderColor: "#ccc",
    padding: 8,
    marginVertical: 10,
    borderRadius: 4,
  },
  error: { color: "red", marginBottom: 10 },
  link: { marginTop: 20, color: "blue" },
});
```

## File: `app/(auth)/sign-up.tsx`
```tsx
// app/(auth)/sign-up.tsx
import { View, Text, Button, TextInput, StyleSheet } from "react-native";
import { useState, useEffect } from "react";
import { Link, useRouter } from "expo-router";
import { useAuth } from "../../context/auth";

export default function SignUpScreen() {
  const { signUp, loading, error, authToken } = useAuth();
  const router = useRouter();

  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [confirmPass, setConfirmPass] = useState("");
  const [localError, setLocalError] = useState("");

  // ----> The "more reliable" method: watch authToken directly.
  useEffect(() => {
    // If user is successfully registered (authToken acquired), go to profile
    if (authToken) {
      router.replace("(protected)/profile");
    }
  }, [authToken]);

  async function handleSignUp() {
    // Basic client-side validation
    if (!validateEmail(email)) {
      setLocalError("Invalid email format");
      return;
    }
    if (password.length < 8) {
      setLocalError("Password must be at least 8 characters");
      return;
    }
    if (password !== confirmPass) {
      setLocalError("Passwords do not match");
      return;
    }

    setLocalError("");
    await signUp(email, password);
    // Let useEffect check if we got a token
  }

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Create Account</Text>

      {localError ? <Text style={styles.error}>{localError}</Text> : null}
      {error ? <Text style={styles.error}>{error}</Text> : null}

      <TextInput
        style={styles.input}
        placeholder="Email"
        autoCapitalize="none"
        onChangeText={setEmail}
        value={email}
      />
      <TextInput
        style={styles.input}
        placeholder="Password (min 8 chars)"
        secureTextEntry
        onChangeText={setPassword}
        value={password}
      />
      <TextInput
        style={styles.input}
        placeholder="Confirm Password"
        secureTextEntry
        onChangeText={setConfirmPass}
        value={confirmPass}
      />

      <Button
        title={loading ? "Signing Up..." : "Sign Up"}
        onPress={handleSignUp}
      />

      <Link href="/(auth)/sign-in" style={styles.link}>
        Already have an account? Sign In
      </Link>
    </View>
  );
}

function validateEmail(email: string) {
  return /\S+@\S+\.\S+/.test(email);
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 20, justifyContent: "center" },
  title: { fontSize: 24, fontWeight: "bold", marginBottom: 20 },
  input: {
    borderWidth: 1,
    borderColor: "#ccc",
    padding: 8,
    marginVertical: 10,
    borderRadius: 4,
  },
  error: { color: "red", marginBottom: 10 },
  link: { marginTop: 20, color: "blue" },
});
```

---

# 5. Protected Routes: `(protected)` Folder

This folder has a `_layout.tsx` that **redirects** you to `(auth)/sign-in` if you lack a token. The `profile.tsx` is an example of a protected screen.

## File: `app/(protected)/_layout.tsx`
```tsx
// app/(protected)/_layout.tsx
import { Stack, useRouter } from "expo-router";
import { useAuth } from "../../context/auth";
import { useEffect } from "react";

export default function ProtectedLayout() {
  const router = useRouter();
  const { authToken } = useAuth();

  // If no authToken, forcibly redirect to sign in
  useEffect(() => {
    if (!authToken) {
      router.replace("(auth)/sign-in");
    }
  }, [authToken]);

  return (
    <Stack
      screenOptions={{
        headerTitle: "Protected",
        headerShown: true,
      }}
    />
  );
}
```

## File: `app/(protected)/profile.tsx`
```tsx
// app/(protected)/profile.tsx
import { View, Text, Button } from "react-native";
import { useRouter } from "expo-router";
import { useAuth } from "../../context/auth";

export default function ProfileScreen() {
  const { authToken, signOut } = useAuth();
  const router = useRouter();

  function handleSignOut() {
    signOut();
    // After sign-out, go back to sign-in
    router.replace("(auth)/sign-in");
  }

  return (
    <View style={{ flex: 1, padding: 20, justifyContent: "center" }}>
      <Text style={{ fontSize: 20, marginBottom: 10 }}>
        Welcome! Your token is: {authToken}
      </Text>
      <Button title="Sign Out" onPress={handleSignOut} />
    </View>
  );
}
```

---

# 6. Optional `index.tsx` for Redirection

If you want the **root** (i.e., `app/index.tsx`) to redirect to sign-in or profile based on login state:

## File: `app/index.tsx`
```tsx
// app/index.tsx
import { Redirect } from "expo-router";
import { useAuth } from "../context/auth";

export default function Index() {
  const { authToken } = useAuth();

  // If logged in, show Profile; else show Sign In
  return authToken ? (
    <Redirect href="/(protected)/profile" />
  ) : (
    <Redirect href="/(auth)/sign-in" />
  );
}
```

---

# 7. Run the App

```bash
npx expo start
```

Test in your simulator or device:

1. **Sign Up** or **Sign In** with any email + a password of **at least 8 characters**.  
2. If the **signIn** / **signUp** calls succeed, the `authToken` becomes `fake_signin_token` or `fake_signup_token`.  
3. Because we **watch `authToken`** in a `useEffect`, you’re automatically routed to the `(protected)/profile` screen.  
4. If you lose your token (signOut, or you refresh in dev mode), you’ll be forced back to `(auth)/sign-in`.  

---

# 8. Why This “More Reliable” Method Helps

Previously, you might see code like:

```js
await signIn(email, password);
if (!error) {
  router.replace("(protected)/profile");
}
```

That can break if:
- `error` is not set yet, or 
- The context updates asynchronously, so `error` or `authToken` might still be `null` during the same function call.

**Now**, we do:
```js
useEffect(() => {
  if (authToken) {
    router.replace("(protected)/profile");
  }
}, [authToken]);
```
This means as soon as `authToken` is truly set in the context, you navigate to the protected route—**no** second login attempt needed.

---

# 9. Key Points

1. **One** top-level `<Stack>` in `app/_layout.tsx`—no extra `NavigationContainer`.  
2. **Nested** layouts `(auth)/_layout.tsx`, `(protected)/_layout.tsx` are normal in Expo Router and do **not** create multiple containers.  
3. Rely on the **actual** `authToken` state to confirm that login/registration succeeded, rather than `error` or immediate synchronous checks.  
4. This approach eliminates double login attempts, flickers, or race conditions.  

With this setup:
- You won’t see “Another navigator is already registered…”  
- You have a **fully functional** sign-up/sign-in flow with password validation, plus **protected** routes.  
- And you’re using the **best practice** of checking the actual `authToken` to know if the user is logged in.  

Happy coding!
