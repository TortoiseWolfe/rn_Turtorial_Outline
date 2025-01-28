Below is a **complete** step-by-step tutorial—no partial snippets—showing **exactly** how to build an Expo Router + Supabase app that:

1. **Requires** real Supabase credentials (URL + anon key) via `.env.local`.  
2. **Auto-creates** a row in `profiles` after each successful sign-up (so the table is **never** empty).  
3. **Shows** sign-up **error messages** (e.g. "User already registered").  
4. **Stores** sessions in **Expo SecureStore** on iOS/Android and **localStorage** on web (so it won’t crash on web).  
5. **Enforces** Row Level Security in Supabase, so each user only sees/updates their own `display_name`.  
6. **Displays & Updates** the user’s display name on a **protected** screen.

By the end, you can sign up (creating a row in `profiles`), sign in, see your `display_name` on `(protected)/profile`, edit it in `(protected)/edit-profile`, and sign out—**all** with error handling for email duplication or other sign-up issues.

---

# Complete Working Tutorial

## Table of Contents

1. [Create a New Expo Router Project](#1-create-a-new-expo-router-project)  
2. [Install Dependencies](#2-install-dependencies)  
3. [Create & Configure `.env.local`](#3-create--configure-envlocal)  
4. [Project File Structure](#4-project-file-structure)  
5. [Supabase Setup](#5-supabase-setup)  
   - [Create or Confirm `profiles` Table](#create-or-confirm-profiles-table)  
   - [Enable Row Level Security & Policies](#enable-row-level-security--policies)  
6. [File: `supabaseClient.ts` (Supabase Connection)](#6-file-supabaseclientts-supabase-connection)  
7. [File: `app/_layout.tsx` (Root Layout)](#7-file-app_layouttsx-root-layout)  
8. [File: `context/auth.tsx` (Auth Context + Automatic Profiles Insert)](#8-file-contextauthtsx-auth-context--automatic-profiles-insert)  
9. [Folder: `(auth)` (Sign In & Sign Up)](#9-folder-auth-sign-in--sign-up)  
   - [`(auth)/_layout.tsx`](#auth_layouttsx)  
   - [`(auth)/sign-in.tsx`](#authsign-intsx)  
   - [`(auth)/sign-up.tsx`](#authsign-uptsx)  
10. [Folder: `(protected)` (Protected Screens)](#10-folder-protected-protected-screens)  
    - [`(protected)/_layout.tsx` (Requires Login)](#protected_layouttsx-requires-login)  
    - [`(protected)/profile.tsx` (Show Display Name)](#protectedprofiletsx-show-display-name)  
    - [`(protected)/edit-profile.tsx` (Edit Display Name)](#protectededit-profiletsx-edit-display-name)  
11. [Optional: `app/index.tsx` (Root Redirect)](#11-optional-appindextsx-root-redirect)  
12. [Run & Test](#12-run--test)  
13. [Recap & Next Steps](#13-recap--next-steps)  

---

## 1) Create a New Expo Router Project

```bash
npx create-expo-app MySupabaseApp
cd MySupabaseApp
npm run reset-project
rm -rf app-example
```

You now have a blank Expo Router project (no leftover `NavigationContainer` calls).

---

## 2) Install Dependencies

Install **Supabase**, **Expo SecureStore**, and **dotenv**:

```bash
npm install @supabase/supabase-js
npx expo install expo-secure-store
npm install dotenv
```

- `@supabase/supabase-js`: the official Supabase client.  
- `expo-secure-store`: secure storage on iOS/Android (not web).  
- `dotenv`: ensures that environment variables in `.env.local` are picked up at build time (when prefixed with `EXPO_PUBLIC_`).

---

## 3) Create & Configure `.env.local`

At the root of the project (same level as `package.json`), **create** a file named **`.env.local`**:

```bash
EXPO_PUBLIC_SUPABASE_URL=https://YOUR-SUPABASE-PROJECT-URL.supabase.co
EXPO_PUBLIC_SUPABASE_ANON_KEY=YOUR-REAL-ANON-KEY
```

Replace with **real** values from **Supabase → Project Settings → API**.  

> Because these variables start with `EXPO_PUBLIC_`, Expo automatically makes them available at runtime (`process.env.EXPO_PUBLIC_...`).

To avoid committing them to Git:

```bash
# .gitignore
.env.local
```

---

## 4) Project File Structure

Inside your `MySupabaseApp` folder, ensure you have the following structure:

```
MySupabaseApp
├─ .env.local
├─ app
│  ├─ _layout.tsx
│  ├─ index.tsx (optional)
│  ├─ (auth)
│  │   ├─ _layout.tsx
│  │   ├─ sign-in.tsx
│  │   └─ sign-up.tsx
│  └─ (protected)
│      ├─ _layout.tsx
│      ├─ profile.tsx
│      └─ edit-profile.tsx
├─ context
│  └─ auth.tsx
├─ supabaseClient.ts
├─ package.json
└─ ...
```

We’ll fill these files with the code below.

---

## 5) Supabase Setup

### Create or Confirm `profiles` Table

1. In your Supabase project’s **SQL editor** (or **Table editor**), create:

   ```sql
   create table if not exists profiles (
     id uuid primary key default uuid_generate_v4(),
     user_id uuid references auth.users not null,
     display_name text,
     created_at timestamp default now()
   );
   ```

   This table holds extra user data. The foreign key references the built-in `auth.users` table.

### Enable Row Level Security & Policies

1. Go to **Database** → **Table Editor** → select the `profiles` table.  
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

Now each user can only see/update their own row in `profiles`.

---

## 6) File: `supabaseClient.ts` (Supabase Connection)

```ts
// supabaseClient.ts
import { createClient } from "@supabase/supabase-js";

const SUPABASE_URL = process.env.EXPO_PUBLIC_SUPABASE_URL!;
const SUPABASE_ANON_KEY = process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY!;

// The exclamation (!) tells TypeScript "trust me, these exist."
export const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
```

This sets up the client with your `.env.local` credentials.

---

## 7) File: `app/_layout.tsx` (Root Layout)

```tsx
// app/_layout.tsx
import { Stack } from "expo-router";
import AuthProvider from "../context/auth";

export default function RootLayout() {
  return (
    <AuthProvider>
      {/* The top-level <Stack> from Expo Router. No header. */}
      <Stack
        screenOptions={{
          headerShown: false,
        }}
      />
    </AuthProvider>
  );
}
```

This wraps our entire app in `AuthProvider`, making the user context available everywhere.

---

## 8) File: `context/auth.tsx` (Auth Context + Automatic Profiles Insert)

Key features:

- **Sign up** → Insert user in `auth.users` + **automatically** create a row in `profiles`.  
- **Sign in** → Surfaces error if the email or password is wrong.  
- **Sign out** → Clears the stored session.  
- **SecureStore** fallback to `localStorage` on web.  
- **RLS** usage is respected because we run inserts/updates as the logged-in user.

```tsx
// context/auth.tsx
import React, { createContext, useContext, useEffect, useState } from "react";
import { Platform } from "react-native";
import * as SecureStore from "expo-secure-store";
import { supabase } from "../supabaseClient";

interface AuthContextProps {
  user: any; // You can define a stricter User type if desired
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

// Fallback: SecureStore on mobile, localStorage on web
async function setItem(key: string, value: string) {
  if (Platform.OS === "web") {
    window.localStorage.setItem(key, value);
  } else {
    await SecureStore.setItemAsync(key, value);
  }
}

async function getItem(key: string) {
  if (Platform.OS === "web") {
    return window.localStorage.getItem(key) ?? null;
  } else {
    return await SecureStore.getItemAsync(key);
  }
}

async function deleteItem(key: string) {
  if (Platform.OS === "web") {
    window.localStorage.removeItem(key);
  } else {
    await SecureStore.deleteItemAsync(key);
  }
}

export default function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<any>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    // On mount, rehydrate from storage
    (async () => {
      try {
        const sessionStr = await getItem(SESSION_KEY);
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
        await setItem(SESSION_KEY, JSON.stringify(session));
      } else {
        setUser(null);
        await deleteItem(SESSION_KEY);
      }
    });

    return () => {
      subscription.unsubscribe();
    };
  }, []);

  // SIGN UP with auto-profiles insert
  async function signUp(email: string, password: string) {
    setError(null);
    setLoading(true);
    try {
      const { data: signUpData, error: signUpError } = await supabase.auth.signUp({
        email,
        password,
      });
      if (signUpError) {
        // Example: "User already registered"
        throw new Error(signUpError.message);
      }

      // If signUp succeeded, create row in "profiles" table
      if (signUpData?.user) {
        const userId = signUpData.user.id;
        const { error: profileError } = await supabase.from("profiles").insert({
          user_id: userId,
          display_name: "", // or any default
        });
        if (profileError) {
          throw new Error(profileError.message);
        }
      }
    } catch (err: any) {
      setError(err.message || "Sign-up failed.");
      console.log("Sign-up error:", err);
    } finally {
      setLoading(false);
    }
  }

  // SIGN IN
  async function signIn(email: string, password: string) {
    setError(null);
    setLoading(true);
    try {
      const { error: signInError } = await supabase.auth.signInWithPassword({
        email,
        password,
      });
      if (signInError) {
        // Example: "Invalid login credentials"
        throw new Error(signInError.message);
      }
    } catch (err: any) {
      setError(err.message || "Sign-in failed.");
      console.log("Sign-in error:", err);
    } finally {
      setLoading(false);
    }
  }

  // SIGN OUT
  async function signOut() {
    setError(null);
    setLoading(true);
    try {
      const { error: signOutError } = await supabase.auth.signOut();
      if (signOutError) {
        throw new Error(signOutError.message);
      }
    } catch (err: any) {
      setError(err.message || "Sign-out failed.");
      console.log("Sign-out error:", err);
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

**Important**: Notice how we **await** the `.insert` into `profiles` right after sign-up. That ensures the `profiles` table is **never** empty if sign-up succeeds.

---

## 9) Folder: `(auth)` (Sign In & Sign Up)

These screens let a user sign up or sign in. We display `error` messages from the context if sign-up or sign-in fails (e.g. “User already registered,” “Invalid credentials,” etc.).

### `(auth)/_layout.tsx`

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

### `(auth)/sign-in.tsx`

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
    // If signIn is successful, user is set in context, and we navigate away
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

### `(auth)/sign-up.tsx`

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
    // If signUp is successful, user is set in context, row is inserted in 'profiles'
    // (See logs for any errors.)
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

## 10) Folder: `(protected)` (Protected Screens)

### `(protected)/_layout.tsx` (Requires Login)

```tsx
// app/(protected)/_layout.tsx
import { Stack, useRouter, useRootNavigationState } from "expo-router";
import { useEffect } from "react";
import { useAuth } from "../../context/auth";

export default function ProtectedLayout() {
  const router = useRouter();
  const navigationState = useRootNavigationState();
  const { user } = useAuth();

  // If the router isn't ready, do nothing yet
  if (!navigationState?.key) {
    return null;
  }

  // If no user, redirect to (auth)/sign-in
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

Any route within `(protected)` will only be accessible if `user` exists in the auth context.

### `(protected)/profile.tsx` (Show Display Name)

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
      <Text style={{ fontSize: 20, marginBottom: 10 }}>
        Welcome, {user.email}!
      </Text>

      {loadingProfile ? (
        <ActivityIndicator />
      ) : (
        <View>
          <Text>Display Name: {profile?.display_name || "(none)"}</Text>
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

Because we **auto-insert** a row in `profiles` at sign-up, you’ll never have an empty table. If you see `(none)`, that’s just the empty string `display_name`.

### `(protected)/edit-profile.tsx` (Edit Display Name)

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

## 11) Optional: `app/index.tsx` (Root Redirect)

If you want `/` to automatically redirect:

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

Open in:

- **iOS simulator** (`i`),  
- **Android emulator** (`a`),  
- or **Web** (`w`).  

### Testing Steps

1. **Sign Up** with a brand-new email:
   - Check your console or Supabase logs if you see errors.  
   - If “User already registered,” you tried an existing email. Use a truly new email.  
2. After sign-up, go to **Supabase** → **Table Editor** → `profiles` and confirm you see the **new** row (with `user_id` = your new user’s ID).  
3. **Sign In**: You should see `(protected)/profile`.  
4. If `display_name` is currently empty, it will say `(none)`.  
5. **Edit Profile**: Type something, hit **Save**, and it updates in your `profiles` table.  
6. **Sign Out** → session is cleared from SecureStore or localStorage, and you’re returned to sign-in.

No more empty `profiles` table!

---

## 13) Recap & Next Steps

You now have a fully working:

- **Expo Router** project with real **Supabase Auth**.  
- **Automatic** `profiles` row creation after sign-up, so no missing data.  
- **Row Level Security** in Supabase so each user only sees/updates their own row.  
- **Error messages** for sign-up and sign-in (e.g., if the email is taken).  
- **Secure** session storage on iOS/Android using `expo-secure-store`, with a **web fallback** to `localStorage`.  

### Possible Enhancements

1. **Email Confirmations**: Enable email confirmations in **Supabase** → **Auth Settings** so new users must confirm before fully signing in.  
2. **DB Trigger**: Instead of inserting to `profiles` from the client, you can define a [Supabase DB Trigger](https://supabase.com/docs/guides/auth/managing-user-data#database-triggers) that auto-creates a `profiles` row whenever a new user is added to `auth.users`.  
3. **Avatars**: Store user avatars in [Supabase Storage](https://supabase.com/docs/guides/storage), reference them in `profiles.avatar_url`, and display them in your app.  
4. **Real-Time**: Use `supabase.channel(...)` or `supabase.on(...)` to subscribe to changes in real time.  
5. **Additional Tables**: Build social features, posts, comments, etc. with RLS-based security.

With this tutorial, your `profiles` table will **never** be empty for newly created users, sign-up error messages will appear, and the entire flow is guaranteed to run on iOS, Android, or web. **Enjoy your working setup!**
