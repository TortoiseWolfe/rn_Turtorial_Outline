# 1) Create a New Project & Clean It

```bash
npx create-expo-app rn_Demo
cd rn_Demo
npm run reset-project
rm -rf app-example
```

At this point, you have a fresh Expo project with **Expo Router** included by default. No need to install or wrap `NavigationContainer`—the router is ready out of the box.

---

# 2) Make the Necessary Folders & Files

We’ll create a single top-level layout, a context folder, `(auth)` routes for sign-in/sign-up, `(protected)` routes for secured pages, and optionally an `index.tsx` for default redirection.

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

touch app/_layout.tsx
touch app/index.tsx  # optional file for default redirection
code .
```

---

# 3) `app/_layout.tsx` (Single Top-Level Layout)

This is **the** top-level layout. It wraps the entire app in an `AuthProvider` (for our auth state) and a single `<Stack>` from Expo Router.

```tsx
// app/_layout.tsx
import { Stack } from "expo-router";
import AuthProvider from "../context/auth";

export default function RootLayout() {
  return (
    <AuthProvider>
      {/**
       * The single top-level <Stack> for the app.
       * Expo Router auto-provides NavigationContainer, so no need to import it.
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

# 4) `context/auth.tsx` (Auth State & Methods)

We’ll store a simple `authToken` and track `loading` + `error`. We simulate real API calls with setTimeout. In a real app, you’d fetch from your server.

```tsx
// context/auth.tsx
import React, { createContext, useContext, useState } from "react";

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

  // Demo endpoints
  const FAKE_SIGNUP_ENDPOINT = "https://example.com/api/signup";
  const FAKE_SIGNIN_ENDPOINT = "https://example.com/api/signin";

  async function signUp(email: string, password: string) {
    setLoading(true);
    setError(null);
    try {
      // In real life: fetch(FAKE_SIGNUP_ENDPOINT, { ... });
      await new Promise((r) => setTimeout(r, 1000)); // 1s delay
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
      // In real life: fetch(FAKE_SIGNIN_ENDPOINT, { ... });
      await new Promise((r) => setTimeout(r, 1000)); // 1s delay
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

# 5) `(auth)` Folder: Sign-In & Sign-Up Flows

We’ll create **two** screens: `sign-in.tsx` and `sign-up.tsx`. The `_layout.tsx` simply sets a header. Notice we use `useEffect` to watch `authToken`. Once it’s set, we navigate to the protected profile.

## `app/(auth)/_layout.tsx`
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

## `app/(auth)/sign-in.tsx`
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

  // Watch authToken. If present, go to protected route
  useEffect(() => {
    if (authToken) {
      router.replace("(protected)/profile");
    }
  }, [authToken]);

  async function handleSignIn() {
    // Basic validation
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
  }

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Sign In</Text>

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

      <Button title={loading ? "Signing In..." : "Sign In"} onPress={handleSignIn} />

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

## `app/(auth)/sign-up.tsx`
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

  // Watch authToken. If present, go to protected route
  useEffect(() => {
    if (authToken) {
      router.replace("(protected)/profile");
    }
  }, [authToken]);

  async function handleSignUp() {
    // Basic validation
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

      <Button title={loading ? "Signing Up..." : "Sign Up"} onPress={handleSignUp} />

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

# 6) `(protected)` Folder: Redirect Safely with `useRootNavigationState()`

To avoid “Attempted to navigate before mounting the Root Layout component…,” we’ll use `useRootNavigationState()` in our `(protected)/_layout.tsx`. That ensures the router is fully mounted **before** we try `router.replace`.

## `app/(protected)/_layout.tsx`
```tsx
// app/(protected)/_layout.tsx
import { Stack, useRouter, useRootNavigationState } from "expo-router";
import { useEffect } from "react";
import { useAuth } from "../../context/auth";

export default function ProtectedLayout() {
  const router = useRouter();
  const navigationState = useRootNavigationState();
  const { authToken } = useAuth();

  // If the router isn't ready yet, don't render anything
  // or navigate, or you'll see the "navigate before mounting" error.
  if (!navigationState?.key) {
    return null; // you could also show a loading screen here
  }

  // Now that the router is ready, we can safely do redirects
  useEffect(() => {
    if (!authToken) {
      router.replace("(auth)/sign-in");
    }
  }, [authToken, router]);

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

## `app/(protected)/profile.tsx`
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
    // Now we can navigate away
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

**Explanation**:  
- We call `useRootNavigationState()` to see if the router has a valid `key`. If it’s not ready, we `return null;`.  
- Once it’s ready, we run our `useEffect` check to see if `authToken` is missing, and **only then** do we `router.replace(...)`. This ensures we never navigate too early.

---

# 7) Optional: `app/index.tsx` for Default Redirect

If you want the root of the app (i.e., no path) to redirect to sign-in or profile:

```tsx
// app/index.tsx
import { Redirect } from "expo-router";
import { useAuth } from "../context/auth";

export default function Index() {
  const { authToken } = useAuth();

  // If logged in, go to profile; else go to sign in
  return authToken ? (
    <Redirect href="/(protected)/profile" />
  ) : (
    <Redirect href="/(auth)/sign-in" />
  );
}
```

This is optional. If you **don’t** include this file, you’ll just see the “screens” in `(auth)` or `(protected)` as you navigate.

---

# 8) Run & Test

```bash
npx expo start --clear
```

Open in your **simulator** or **device**. You should see:

1. The **Sign In** or **Sign Up** screen.  
2. **Validation**: Email must be valid, password at least 8 chars.  
3. After a 1s fake “API call,” you’ll get `fake_signin_token` or `fake_signup_token`.  
4. The `(protected)` layout sees that `authToken` is set (and the router is ready), so it navigates to `profile.tsx`.  
5. If you sign out, `authToken` goes to `null`, and `(protected)/_layout.tsx` forces a redirect to `(auth)/sign-in`, again waiting for the router to be ready.

You **won’t** see a white screen or the **“Attempted to navigate before mounting…”** error because we used `useRootNavigationState()` to confirm readiness.

---

## Final Thoughts

- **`useRootNavigationState()`** is the official fix for “navigate before mounting” issues. We simply check if `!navigationState?.key`—then **don’t** redirect or even render the `<Stack>` until it’s ready.  
- This ensures a **smooth** user experience with no blank screen or multiple logins.  
- If you still see a blank screen, **double-check** your folder names `(auth)`, `(protected)`, and file names `_layout.tsx` exactly.  
- In a real app, you’d store your token in secure storage, call real endpoints, etc. But the flow remains the same: once you have a token, you navigate to protected screens.  

That’s it! This **complete** tutorial prevents both blank screens and early navigate errors, giving you a robust sign-in/sign-up flow with protected routes in Expo Router. Enjoy!
