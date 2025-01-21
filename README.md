## 1. New Project & Basic Housekeeping

```bash
npx create-expo-app rn_Demo
cd rn_Demo
npm run reset-project
rm -rf app-example
```

---

## 2. Single Top-Level Layout

Create a **single** `_layout.tsx` in `app/`, which is your **only** top-level `<Stack>`:

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
code .

```

**Paste:**

```tsx
// app/_layout.tsx
import { Stack } from "expo-router";
import AuthProvider from "../context/auth";

export default function RootLayout() {
  return (
    <AuthProvider>
      {/**
       * The single top-level Stack for the entire app.
       * Expo Router will handle everything behind the scenes.
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

## 3. Auth Context

We’ll store:

1. An `authToken` (or `user`)  
2. `signIn`, `signUp`, `signOut` methods  
3. A minimal “fake” HTTP request to simulate server calls  

```bash
mkdir context
touch context/auth.tsx
```

**Paste:**

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

  // Fake API endpoints for demonstration:
  const FAKE_SIGNUP_ENDPOINT = "https://example.com/api/signup";
  const FAKE_SIGNIN_ENDPOINT = "https://example.com/api/signin";

  async function signUp(email: string, password: string) {
    setLoading(true);
    setError(null);
    try {
      // Fake HTTP request
      // In real life, you'd do something like:
      // const response = await fetch(FAKE_SIGNUP_ENDPOINT, { ... });
      // if (!response.ok) throw new Error("Sign up failed");
      // const data = await response.json();
      // setAuthToken(data.token);

      // For this demo, let's simulate success after 1 second
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
      // Fake HTTP request
      // In a real app:
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

## 4. Auth Routes: `(auth)` Folder

We’ll create `(auth)` with `_layout.tsx`, `sign-in.tsx`, and `sign-up.tsx`, each with **email + password** fields and **basic validation**.

```bash
mkdir -p app/\(auth\)
touch app/\(auth\)/_layout.tsx
touch app/\(auth\)/sign-in.tsx
touch app/\(auth\)/sign-up.tsx
```

### (auth)/_layout.tsx

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

### sign-in.tsx

```tsx
// app/(auth)/sign-in.tsx
import { View, Text, Button, TextInput, StyleSheet } from "react-native";
import { useState } from "react";
import { Link, useRouter } from "expo-router";
import { useAuth } from "../../context/auth";

export default function SignInScreen() {
  const { signIn, loading, error } = useAuth();
  const router = useRouter();
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [localError, setLocalError] = useState("");

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

    setLocalError(""); // Clear local error
    await signIn(email, password);
    if (!error) {
      // If sign in is successful, route to protected Profile
      router.replace("(protected)/profile");
    }
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
  // Basic email check
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

### sign-up.tsx

```tsx
// app/(auth)/sign-up.tsx
import { View, Text, Button, TextInput, StyleSheet } from "react-native";
import { useState } from "react";
import { Link, useRouter } from "expo-router";
import { useAuth } from "../../context/auth";

export default function SignUpScreen() {
  const { signUp, loading, error } = useAuth();
  const router = useRouter();
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [confirmPass, setConfirmPass] = useState("");
  const [localError, setLocalError] = useState("");

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

    setLocalError(""); // Clear local error
    await signUp(email, password);
    if (!error) {
      // If sign up is successful, route to protected Profile
      router.replace("(protected)/profile");
    }
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

## 5. Protected Routes: `(protected)` Folder

Create `(protected)/_layout.tsx` to automatically **redirect** if not logged in, and a `profile.tsx` to test protected content.

```bash
mkdir -p app/\(protected\)
touch app/\(protected\)/_layout.tsx
touch app/\(protected\)/profile.tsx
```

### (protected)/_layout.tsx

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

### profile.tsx

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
    // After sign-out, back to sign-in
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

## 6. Add an `index.tsx` (Optional)

An index route that **redirects** based on login status. If you do **not** have an `app/index.tsx` yet:

```bash
touch app/index.tsx
```

**Paste:**

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

## 7. Run the App

Finally, run:

```bash
npx expo start
```

Open the app in your simulator or device. You should see:

1. **Sign In** or Sign Up if not logged in.  
2. Basic validation on email + password.  
3. On success, you’ll see “fake_signin_token” or “fake_signup_token” on the Profile screen.  
4. The “Profile” route is **protected**—if you try to navigate to `(protected)/profile` without being authenticated, you’ll be redirected to sign in.  

---

## 8. Important Reminders

1. **No** `NavigationContainer` usage: With Expo Router, that’s automatically handled.  
2. Only **one** top-level `<Stack>` in `app/_layout.tsx`.  
3. Nested `<Stack>` usage in `(auth)/_layout.tsx` and `(protected)/_layout.tsx` is normal—Expo Router merges them into a single navigation tree behind the scenes.  

This ensures you **never** see “Another navigator is already registered for this container…” again, while providing a **more complete** sign-up/sign-in flow with password checks, validation, and protected routes!
