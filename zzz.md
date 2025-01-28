# ScripitHammer

1. [Set Up Your Expo Project](#1-set-up-your-expo-project)  
2. [Install Dependencies](#2-install-dependencies)  
3. [Create & Configure `.env.local`](#3-create--configure-envlocal)  
4. [File & Folder Structure](#4-file--folder-structure)  
5. [Supabase Setup](#5-supabase-setup)  
   - [Create or Confirm `profiles` Table](#create-or-confirm-profiles-table)  
   - [Enable RLS & Policies](#enable-rls--policies)  
6. [Code: `supabaseClient.ts` (Connecting to Supabase)](#6-code-supabaseclientts-connecting-to-supabase)  
7. [Code: `app/_layout.tsx` (Single Top-Level Layout)](#7-code-app_layouttsx-single-top-level-layout)  
8. [Code: `context/auth.tsx` (Auth Context with SecureStore + Supabase)](#8-code-contextauthtsx-auth-context-with-securestore--supabase)  
9. [Code: `(auth)` Folder (Sign In & Sign Up)](#9-code-auth-folder-sign-in--sign-up)  
   - [`(auth)/_layout.tsx`](#auth_layouttsx)  
   - [`(auth)/sign-in.tsx`](#authsign-intsx)  
   - [`(auth)/sign-up.tsx`](#authsign-uptsx)  
10. [Code: `(protected)` Folder (Protected Routes)](#10-code-protected-folder-protected-routes)  
    - [`(protected)/_layout.tsx`](#protected_layouttsx)  
    - [`(protected)/profile.tsx` (Displays + Fetches Profile)](#protectedprofiletsx-displays--fetches-profile)  
    - [`(protected)/edit-profile.tsx` (Edit Display Name)](#protectededit-profiletsx-edit-display-name)  
11. [Optional: `app/index.tsx` (Default Redirection)](#11-optional-appindextsx-default-redirection)  
12. [Run & Test](#12-run--test)  
13. [Recap & Next Steps](#13-recap--next-steps)  

---

## 1) Set Up Your Expo Project

```bash
npx create-expo-app ScriptHammer
cd ScriptHammer
npm run reset-project
rm -rf app-example
```

You now have a blank Expo Router project (no leftover `NavigationContainer` calls).

---

## 2) Install Dependencies

We need:

```bash
npm install @supabase/supabase-js
npx expo install expo-secure-store
```

Additionally, for environment variables:

```bash
npm install dotenv
```

Expo’s bundler automatically picks up variables that start with **`EXPO_PUBLIC_`**, so no further custom configuration is needed beyond having your `.env.local`.  

---

## 3) Create & Configure `.env.local`

In the root of your project (same level as `package.json`), **create** a file named **`.env.local`** with the following:

```bash
EXPO_PUBLIC_SUPABASE_URL=https://lll.supabase.co
EXPO_PUBLIC_SUPABASE_ANON_KEY=...
```

(This is just an example key; use your real Supabase details.)

> **Important**: Because this is a **public** anon key, it’s not as secret as a private key, but you still might not want it in public repos. For truly private secrets, use a server-side approach or other secure methods.  

---

## 4) File & Folder Structure

We’ll create these folders & files:

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
touch app/\(protected\)/edit-profile.tsx

touch supabaseClient.ts
touch app/_layout.tsx
touch app/index.tsx
touch .env.local
code .
```

---

## 5) Supabase Setup

1. Log in to [Supabase](https://app.supabase.com/).  
2. **Project Settings** -> **API**: Copy your **Project URL** and **anon key**.  
3. Create a `profiles` table or confirm it exists:

### Create or Confirm `profiles` Table

In your Supabase project, run something like:

```sql
create table if not exists profiles (
  id uuid primary key default uuid_generate_v4(),
  user_id uuid references auth.users not null,
  display_name text,
  created_at timestamp default now()
);
```

### Enable RLS & Policies

1. Go to **Table Editor** -> `profiles`.  
2. **Enable RLS**.  
3. Add a **SELECT** policy:

```sql
create policy "Select own profile"
on public.profiles
for select
using ( auth.uid() = user_id );
```

4. Add an **UPDATE** policy:

```sql
create policy "Update own profile"
on public.profiles
for update
using ( auth.uid() = user_id );
```

This ensures each logged-in user can only see/update their own row in `profiles`.

---

## 6) Code: `supabaseClient.ts` (Connecting to Supabase)

**File: `supabaseClient.ts`**

```ts
// supabaseClient.ts
import { createClient } from "@supabase/supabase-js";

// Because we used the prefix "EXPO_PUBLIC_", 
// these are automatically available as process.env.<VAR_NAME>
// in the Expo environment. 
const SUPABASE_URL = process.env.EXPO_PUBLIC_SUPABASE_URL!;
const SUPABASE_ANON_KEY = process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY!;

export const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
```

> Note the exclamation points `!` in TypeScript, which tell the compiler “trust me, these exist.”  

Because these variables are prefixed with `EXPO_PUBLIC_`, **Expo** will include them at build time. If you used any other prefix, you’d have to configure bundler behavior.  

---

## 7) Code: `app/_layout.tsx` (Single Top-Level Layout)

**File: `app/_layout.tsx`**

```tsx
// app/_layout.tsx
import { Stack } from "expo-router";
import AuthProvider from "../context/auth";

export default function RootLayout() {
  return (
    <AuthProvider>
      {/* 
        One top-level <Stack>.
        Expo Router auto-provides NavigationContainer behind the scenes.
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

## 8) Code: `context/auth.tsx` (Auth Context with SecureStore + Supabase)

This handles:

- **Sign Up / Sign In** via Supabase.  
- **Session** rehydration from **Expo SecureStore**.  
- On mount, if we find an existing session, we set the user.  
- We subscribe to Supabase `onAuthStateChange` to keep the local user in sync with server changes.  

**File: `context/auth.tsx`**

```tsx
// context/auth.tsx
import React, { createContext, useContext, useEffect, useState } from "react";
import * as SecureStore from "expo-secure-store";
import { supabase } from "../supabaseClient";

interface AuthContextProps {
  user: any; // Could define a stricter type from Supabase if you want
  loading: boolean;
  error: string | null;
  signUp: (email: string, password: string) => Promise<void>;
  signIn: (email: string, password: string) => Promise<void>;
  signOut: () => Promise<void>;
}

const SESSION_KEY = "mySupabaseSession";

const AuthContext = createContext<AuthContextProps>({
  user: null,
  loading: false,
  error: null,
  signUp: async () => {},
  signIn: async () => {},
  signOut: async () => {},
});

export default function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<any>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    // On mount, rehydrate from SecureStore
    (async () => {
      try {
        const sessionStr = await SecureStore.getItemAsync(SESSION_KEY);
        if (sessionStr) {
          const session = JSON.parse(sessionStr);
          supabase.auth.setSession(session);
          setUser(session.user);
        }
      } catch (err) {
        console.log("Failed to load session:", err);
      }
      setLoading(false);
    })();

    // Listen for auth changes
    const {
      data: { subscription },
    } = supabase.auth.onAuthStateChange(async (event, session) => {
      console.log("Supabase auth event:", event);
      if (session?.user) {
        setUser(session.user);
        await SecureStore.setItemAsync(SESSION_KEY, JSON.stringify(session));
      } else {
        setUser(null);
        await SecureStore.deleteItemAsync(SESSION_KEY);
      }
    });

    return () => {
      subscription.unsubscribe();
    };
  }, []);

  async function signUp(email: string, password: string) {
    setError(null);
    setLoading(true);
    try {
      const { error: signUpError } = await supabase.auth.signUp({ email, password });
      if (signUpError) throw new Error(signUpError.message);
    } catch (err: any) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }

  async function signIn(email: string, password: string) {
    setError(null);
    setLoading(true);
    try {
      const { error: signInError } = await supabase.auth.signInWithPassword({
        email,
        password,
      });
      if (signInError) throw new Error(signInError.message);
    } catch (err: any) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }

  async function signOut() {
    setError(null);
    setLoading(true);
    try {
      const { error: signOutError } = await supabase.auth.signOut();
      if (signOutError) throw new Error(signOutError.message);
    } catch (err: any) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }

  return (
    <AuthContext.Provider
      value={{
        user,
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

## 9) Code: `(auth)` Folder (Sign In & Sign Up)

### **File**: `app/(auth)/_layout.tsx`

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

### **File**: `app/(auth)/sign-in.tsx`

```tsx
// app/(auth)/sign-in.tsx
import { View, Text, Button, TextInput, StyleSheet } from "react-native";
import { useState, useEffect } from "react";
import { Link, useRouter } from "expo-router";
import { useAuth } from "../../context/auth";

export default function SignInScreen() {
  const { signIn, loading, error, user } = useAuth();
  const router = useRouter();

  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [localError, setLocalError] = useState("");

  // If user is already logged in, skip sign-in
  useEffect(() => {
    if (user) {
      router.replace("(protected)/profile");
    }
  }, [user]);

  async function handleSignIn() {
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
    // if signIn successful, user is set by onAuthStateChange
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

### **File**: `app/(auth)/sign-up.tsx`

```tsx
// app/(auth)/sign-up.tsx
import { View, Text, Button, TextInput, StyleSheet } from "react-native";
import { useState, useEffect } from "react";
import { Link, useRouter } from "expo-router";
import { useAuth } from "../../context/auth";

export default function SignUpScreen() {
  const { signUp, loading, error, user } = useAuth();
  const router = useRouter();

  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [confirmPass, setConfirmPass] = useState("");
  const [localError, setLocalError] = useState("");

  // If user is already logged in, skip sign-up
  useEffect(() => {
    if (user) {
      router.replace("(protected)/profile");
    }
  }, [user]);

  async function handleSignUp() {
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
    // if signUp is successful, user is set by onAuthStateChange
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

## 10) Code: `(protected)` Folder (Protected Routes)

We’ll see **two** screens:

1. **profile.tsx**: Fetches data from the `profiles` table to display `display_name`.  
2. **edit-profile.tsx**: Lets the user update their `display_name`.  

### **File**: `app/(protected)/_layout.tsx`

```tsx
// app/(protected)/_layout.tsx
import { Stack, useRouter, useRootNavigationState } from "expo-router";
import { useEffect } from "react";
import { useAuth } from "../../context/auth";

export default function ProtectedLayout() {
  const router = useRouter();
  const navigationState = useRootNavigationState();
  const { user } = useAuth();

  // If the router isn't ready, don't do anything
  if (!navigationState?.key) {
    return null;
  }

  // If no user, redirect to sign in
  useEffect(() => {
    if (!user) {
      router.replace("(auth)/sign-in");
    }
  }, [user]);

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

### **File**: `app/(protected)/profile.tsx`

```tsx
// app/(protected)/profile.tsx
import { View, Text, Button, ActivityIndicator } from "react-native";
import { useState, useEffect } from "react";
import { useRouter } from "expo-router";
import { useAuth } from "../../context/auth";
import { supabase } from "../../supabaseClient";

export default function ProfileScreen() {
  const { user, signOut } = useAuth();
  const [profile, setProfile] = useState<any>(null);
  const [loadingProfile, setLoadingProfile] = useState(false);
  const router = useRouter();

  async function fetchProfile() {
    if (!user?.id) return;
    setLoadingProfile(true);
    try {
      const { data, error } = await supabase
        .from("profiles")
        .select("*")
        .eq("user_id", user.id)
        .single();

      if (error) {
        console.log("Error fetching profile:", error.message);
      } else {
        setProfile(data);
      }
    } finally {
      setLoadingProfile(false);
    }
  }

  useEffect(() => {
    fetchProfile();
  }, [user?.id]);

  async function handleSignOut() {
    await signOut();
    router.replace("(auth)/sign-in");
  }

  if (!user) {
    return (
      <View style={{ flex: 1, justifyContent: "center", padding: 20 }}>
        <Text>No user found, please sign in.</Text>
      </View>
    );
  }

  return (
    <View style={{ flex: 1, padding: 20, justifyContent: "center" }}>
      <Text style={{ fontSize: 20, marginBottom: 10 }}>Welcome, {user.email}!</Text>

      {loadingProfile ? (
        <ActivityIndicator />
      ) : (
        <View>
          <Text>Display Name: {profile?.display_name || "(none)"} </Text>
        </View>
      )}

      <View style={{ marginTop: 20 }}>
        <Button
          title="Edit Profile"
          onPress={() => router.push("(protected)/edit-profile")}
        />
      </View>

      <View style={{ marginTop: 20 }}>
        <Button title="Sign Out" onPress={handleSignOut} />
      </View>
    </View>
  );
}
```

### **File**: `app/(protected)/edit-profile.tsx`

```tsx
// app/(protected)/edit-profile.tsx
import { View, Text, TextInput, Button, StyleSheet } from "react-native";
import { useState, useEffect } from "react";
import { useRouter } from "expo-router";
import { useAuth } from "../../context/auth";
import { supabase } from "../../supabaseClient";

export default function EditProfileScreen() {
  const { user } = useAuth();
  const router = useRouter();

  const [displayName, setDisplayName] = useState("");
  const [saving, setSaving] = useState(false);
  const [loading, setLoading] = useState(true);

  async function loadCurrentProfile() {
    if (!user?.id) return;
    try {
      const { data, error } = await supabase
        .from("profiles")
        .select("*")
        .eq("user_id", user.id)
        .single();

      if (!error && data) {
        setDisplayName(data.display_name || "");
      }
    } catch (err) {
      console.log("Error loading profile:", err);
    } finally {
      setLoading(false);
    }
  }

  useEffect(() => {
    loadCurrentProfile();
  }, [user?.id]);

  async function handleSave() {
    if (!user?.id) return;
    setSaving(true);
    try {
      const { error } = await supabase
        .from("profiles")
        .update({ display_name: displayName })
        .eq("user_id", user.id);

      if (error) {
        console.log("Update error:", error.message);
      } else {
        router.replace("(protected)/profile");
      }
    } finally {
      setSaving(false);
    }
  }

  if (loading) {
    return (
      <View style={styles.container}>
        <Text>Loading Profile...</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <Text style={styles.label}>Display Name:</Text>
      <TextInput
        style={styles.input}
        value={displayName}
        onChangeText={setDisplayName}
        placeholder="Enter a display name"
      />
      <Button
        title={saving ? "Saving..." : "Save Changes"}
        onPress={handleSave}
        disabled={saving}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 20, justifyContent: "center" },
  label: { fontWeight: "bold", marginBottom: 5 },
  input: {
    borderWidth: 1,
    borderColor: "#ccc",
    padding: 8,
    marginBottom: 15,
    borderRadius: 4,
  },
});
```

---

## 11) Optional: `app/index.tsx` (Default Redirection)

If you want `/` to redirect to `(auth)/sign-in` or `(protected)/profile`:

```tsx
// app/index.tsx
import { Redirect } from "expo-router";
import { useAuth } from "../context/auth";

export default function Index() {
  const { user } = useAuth();

  return user ? (
    <Redirect href="/(protected)/profile" />
  ) : (
    <Redirect href="/(auth)/sign-in" />
  );
}
```

---

## 12) Run & Test

```bash
npx expo start --clear
```

1. **Sign Up**. If your Supabase project requires email confirmation, check your inbox.  
2. **Sign In**. You’ll see `(protected)/profile`.  
3. **`profiles`**: If you haven’t inserted a row automatically, you might see `"(none)"` for `display_name`. You can manually insert a row in the Supabase SQL editor or do it automatically after sign-up.  
4. **Edit Display Name**: Try editing your display name in `(protected)/edit-profile`. It should update in your Supabase `profiles` table.  
5. **Sign Out**—the SecureStore session is cleared, and you’re redirected to sign-in.  

---

## 13) Recap & Next Steps

You now have:

1. **Real Supabase Auth** with session management in **Expo SecureStore**.  
2. **(Protected)** routes in Expo Router (requires a valid `user`).  
3. **`profiles`** table with **Row Level Security**—each user can see/update only their own row.  
4. **Environment variables** in `.env.local` using `EXPO_PUBLIC_` prefixes.

### Next Steps

1. **Automatically Create `profiles` Rows**: Either add code in `signUp(...)` or use a [Supabase DB Trigger](https://supabase.com/docs/guides/auth/managing-user-data#database-triggers) so every new user automatically gets a `profiles` row.  
2. **Avatars**: Use Supabase Storage for profile pictures, referencing an image URL in `profiles.avatar_url`.  
3. **Real-Time**: Subscribe to real-time changes with `supabase.channel("...")` so the user sees updates instantly.  
4. **Social Features**: Create a “posts” table with RLS. Let users post, like, comment, etc.

By combining these steps, you now have a robust, secure foundation for your Expo Router + Supabase app. Enjoy—and don’t forget to store your environment variables in `.env.local` safely!
