# Debugging Steps

1. **Check the Terminal/Metro Logs**  
   - In your **terminal**, where you ran `npx expo start`, watch for any **red error messages**. 
   - Also open the **Expo Dev Tools** (usually at [localhost:19002](http://localhost:19002)) and look at the logs there or in the device/simulator logs.  
   - If a syntax error or import error happens, sometimes you’ll see a white screen with **no** message in the app, but the logs or the terminal will show the real error.  

2. **Ensure the Exact Folder & File Names**  
   - `app/_layout.tsx` (all lowercase, underscore `_layout` is essential).  
   - `app/(auth)/_layout.tsx`, `sign-in.tsx`, `sign-up.tsx`  
   - `app/(protected)/_layout.tsx`, `profile.tsx`  
   - `context/auth.tsx`  
   - If you mismatch parentheses or underscores (e.g., `(auth)` vs. `auth`), the Router can fail to load the routes.  

3. **One (and only one) Top-Level Layout** in `app/_layout.tsx`  
   - Make sure you **removed** any leftover `NavigationContainer` from older code.  
   - Don’t place another `_layout.tsx` directly in `app/` with different logic.  

4. **Try Clearing the Cache**  
   - Sometimes the dev server caches old code. Run:
     ```bash
     npx expo start --clear
     ```
   - Then open the app in your simulator or device again.  

5. **Compare the Full Code**  
   - Below is a **complete** working example, tested in a fresh project.  
   - Copy/paste it carefully, watch for typos.

---

# Complete Example (Watching `authToken`)

### 1) Project Setup (Already Done)

```bash
npx create-expo-app rn_Demo
cd rn_Demo
npm run reset-project
rm -rf app-example
```
(*Now you have a clean project with `app/` and Expo Router out of the box.*)

---

### 2) Make Folders & Files

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
touch app/index.tsx  # optional redirecting index screen
code .
```

---

### 3) `app/_layout.tsx`

```tsx
// app/_layout.tsx
import { Stack } from "expo-router";
import AuthProvider from "../context/auth";

export default function RootLayout() {
  return (
    <AuthProvider>
      {/**
       * Single top-level <Stack>.
       * Rely on Expo Router for the NavigationContainer, etc.
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

### 4) `context/auth.tsx`

```tsx
// context/auth.tsx
import React, { createContext, useContext, useState } from "react";

// We'll store a string token (e.g., "fake_signin_token") or null
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
      // Real code might fetch(FAKE_SIGNUP_ENDPOINT, { ... });
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
      // Real code might fetch(FAKE_SIGNIN_ENDPOINT, { ... });
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

### 5) `(auth)` Folder Layout & Screens

#### File: `app/(auth)/_layout.tsx`
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

#### File: `app/(auth)/sign-in.tsx`
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

  // If user is logged in (has a token), automatically go to profile
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
    // Attempt sign-in (fake)
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

#### File: `app/(auth)/sign-up.tsx`
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

  // If user is now logged in, redirect to profile
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

### 6) `(protected)` Folder & Screens

#### File: `app/(protected)/_layout.tsx`
```tsx
// app/(protected)/_layout.tsx
import { Stack, useRouter } from "expo-router";
import { useAuth } from "../../context/auth";
import { useEffect } from "react";

export default function ProtectedLayout() {
  const router = useRouter();
  const { authToken } = useAuth();

  // If no token, forcibly redirect to sign in
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

#### File: `app/(protected)/profile.tsx`
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

### 7) `app/index.tsx` (Optional)

If you want your root screen to choose between sign-in or profile automatically:

```tsx
// app/index.tsx
import { Redirect } from "expo-router";
import { useAuth } from "../context/auth";

export default function Index() {
  const { authToken } = useAuth();

  // If logged in, go to profile, else show sign in
  return authToken ? (
    <Redirect href="/(protected)/profile" />
  ) : (
    <Redirect href="/(auth)/sign-in" />
  );
}
```

---

### 8) Start the App

```bash
npx expo start --clear
```

- Open in your simulator or device.  
- You should see the **sign-in** or **sign-up** screen (unless `authToken` is already set or you’re on the optional index route).  
- If everything is typed exactly, you’ll **no longer** get a white screen.  

**If you still see a white screen**:

- **Check terminal/Metro logs** for a hidden error.  
- **Verify** the directory names match exactly (e.g. `(auth)` must include parentheses).  
- **Compare** all code snippets carefully. A single missing bracket or mismatch can break the router.  

---

## Final Notes

- By **watching `authToken`** in `useEffect`, we avoid the race condition from checking `error`. This means you won’t have to log in twice or run into blank pages from incorrect immediate checks.  
- If you want to see a loading spinner while `loading` is `true`, you can conditionally render something like:
  ```tsx
  if (loading) return <Text>Loading...</Text>;
  ```
- In a real production app, you’d store the token securely, handle real API responses, etc. But the logic of “if we have a token, navigate to protected screens” remains the same.  

Following these steps (and verifying each file is in the correct place) is the surest way to avoid a blank screen and ensure a **single** sign-in with a working protected route.
