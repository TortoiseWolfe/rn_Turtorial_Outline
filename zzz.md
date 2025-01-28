Below is the **same full tutorial** (with the same file structure, commands, and code as before).  
However, we now incorporate **two additional tweaks**—**(1)** inserting a default `profiles` row right after sign-up, and **(2)** subscribing to real-time changes—since both can help avoid the “no profile row” bug (and also allow the user to see updates immediately).

You can keep **both** approaches or pick whichever suits your workflow:

1. **Insert a default row on sign-up** ensures a `profiles` row **always** exists when a new user registers.  
2. **Real-time subscription** to `profiles` means if data changes on that row, you see it right away without needing to re-fetch.

> **Important**: All the original steps, including `npm run reset-project` and folder/file creation, remain **unchanged** below. We’ve simply added the relevant lines in the code for default-row insertion (inside `signUp`) and a real-time subscription (inside `profile.tsx`).

---

# Table of Contents

1. [Set Up Your Expo Project](#1-set-up-your-expo-project)  
2. [Install Dependencies](#2-install-dependencies)  
3. [File & Folder Structure](#3-file--folder-structure)  
4. [Supabase Setup](#4-supabase-setup)  
   - [Create or Confirm `profiles` Table](#create-or-confirm-profiles-table)  
   - [Enable RLS & Policies](#enable-rls--policies)  
5. [Code: `supabaseClient.ts` (Connecting to Supabase)](#5-code-supabaseclientts-connecting-to-supabase)  
6. [Code: `app/_layout.tsx` (Single Top-Level Layout)](#6-code-app_layouttsx-single-top-level-layout)  
7. [Code: `context/auth.tsx` (Auth Context with SecureStore + Supabase)](#7-code-contextauthtsx-auth-context-with-securestore--supabase)  
   - [**New**: Insert a Default Profile Row on Sign-Up (Optional)](#new-insert-a-default-profile-row-on-sign-up-optional)  
8. [Code: `(auth)` Folder (Sign In & Sign Up)](#8-code-auth-folder-sign-in--sign-up)  
   - [`(auth)/_layout.tsx`](#auth_layouttsx)  
   - [`(auth)/sign-in.tsx`](#authsign-intsx)  
   - [`(auth)/sign-up.tsx`](#authsign-uptsx)  
9. [Code: `(protected)` Folder (Protected Routes)](#9-code-protected-folder-protected-routes)  
   - [`(protected)/_layout.tsx`](#protected_layouttsx)  
   - [`(protected)/profile.tsx` (Now includes real-time subscription)](#protectedprofiletsx-now-includes-real-time-subscription)  
   - [`(protected)/edit-profile.tsx` (Uses `upsert`)](#protectededit-profiletsx-uses-upsert)  
10. [Optional: `app/index.tsx` (Default Redirection)](#10-optional-appindextsx-default-redirection)  
11. [Run & Test](#11-run--test)  
12. [Recap & Next Steps](#12-recap--next-steps)  

---

## 1) Set Up Your Expo Project

```bash
npx create-expo-app rn_SupabaseDemo
cd rn_SupabaseDemo
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

---

## 3) File & Folder Structure

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
code .
```

---

## 4) Supabase Setup

1. Log in to [Supabase](https://app.supabase.com/).  
2. **Project Settings** -> **API**: Copy your **Project URL** and **anon key**.  
3. Create a `profiles` table or confirm it exists:

### Create or Confirm `profiles` Table

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

Now each user can only see/update **their** `profiles` row.

---

## 5) Code: `supabaseClient.ts` (Connecting to Supabase)

**File: `supabaseClient.ts`**

```ts
// supabaseClient.ts
import { createClient } from "@supabase/supabase-js";

const SUPABASE_URL = "https://YOUR_PROJECT_ID.supabase.co";
const SUPABASE_ANON_KEY = "YOUR_ANON_KEY_FROM_DASHBOARD";

export const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
```

> The **anon key** is **not** secret; RLS policies protect your data.

---

## 6) Code: `app/_layout.tsx` (Single Top-Level Layout)

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

## 7) Code: `context/auth.tsx` (Auth Context with SecureStore + Supabase)

This handles:

- **Sign Up / Sign In** with Supabase.  
- **Session** rehydration from **Expo SecureStore** (so users stay logged in).  
- We subscribe to `onAuthStateChange` to keep our local `user` in sync if the token changes.

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
      const { data, error: signUpError } = await supabase.auth.signUp({
        email,
        password,
      });
      if (signUpError) throw new Error(signUpError.message);

      // If no email confirmation is required, signUpData.user will be the new user object.
      // If email confirmation IS required, signUpData.user might be null at this point.
      // See "Insert a Default Profile Row on Sign-Up" below if you want a guaranteed row.

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

### **New**: Insert a Default Profile Row on Sign-Up (Optional)

If you want to **guarantee** each new user has a row in `profiles` at sign-up time (instead of waiting for them to create/update it later), **add** the following snippet right after you get a successful sign-up:

```diff
// inside signUp(...)
+    if (!signUpError && data?.user) {
+      const newUser = data.user; // the newly created user
+      const { error: insertError } = await supabase.from("profiles").insert({
+        user_id: newUser.id,
+        display_name: "", // or some default name
+      });
+      if (insertError) {
+        console.warn("Could not create initial profile row:", insertError.message);
+      }
+    }
```

That way, by the time the user logs in for the first time, they **already** have a row in `profiles`. (If your Supabase project requires email confirmation, note that `data.user` might be `null` until after they verify their email.)

---

## 8) Code: `(auth)` Folder (Sign In & Sign Up)

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
    // If sign-up is successful, user might still be null until email is confirmed
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

## 9) Code: `(protected)` Folder (Protected Routes)

We have two screens here:

1. **`profile.tsx`**: Displays the user’s email + their `profiles` data. We’ll now add a **real-time** subscription to the `profiles` table so changes update instantly.  
2. **`edit-profile.tsx`**: Allows updating (or creating via upsert) the `display_name`.

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

### **File**: `app/(protected)/profile.tsx` (Now includes real-time subscription)

```tsx
// app/(protected)/profile.tsx
import { View, Text, Button, ActivityIndicator } from "react-native";
import { useState, useEffect } from "react";
import { useRouter } from "expo-router";
import { useAuth } from "../../context/auth";
import { supabase } from "../../supabaseClient";

export default function ProfileScreen() {
  const { user, signOut } = useAuth();
  const router = useRouter();

  const [profile, setProfile] = useState<any>(null);
  const [loadingProfile, setLoadingProfile] = useState(false);

  useEffect(() => {
    if (!user?.id) return;
    fetchProfile();

    // 1. Subscribe to changes in "profiles" for this user_id
    const channel = supabase
      .channel("any_unique_channel_name")
      .on(
        "postgres_changes",
        {
          event: "*", // could be 'UPDATE' or 'INSERT'
          schema: "public",
          table: "profiles",
          filter: `user_id=eq.${user.id}`,
        },
        (payload) => {
          console.log("Realtime profile change:", payload);
          if (payload.new) {
            // If there's updated data, set that as our profile state
            setProfile(payload.new);
          }
        }
      )
      .subscribe();

    // Cleanup subscription on unmount
    return () => {
      supabase.removeChannel(channel);
    };
  }, [user?.id]);

  async function fetchProfile() {
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

> **How Real-Time Helps**: As soon as you call `.upsert(...)` or `.update(...)` from another device (or your own), the subscription callback runs, updating your local `profile` state.

### **File**: `app/(protected)/edit-profile.tsx` (Uses `upsert`)

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

  useEffect(() => {
    loadCurrentProfile();
  }, [user?.id]);

  async function loadCurrentProfile() {
    if (!user?.id) return;
    setLoading(true);
    const { data, error } = await supabase
      .from("profiles")
      .select("*")
      .eq("user_id", user.id)
      .single();

    if (!error && data) {
      setDisplayName(data.display_name || "");
    }
    setLoading(false);
  }

  async function handleSave() {
    if (!user?.id) return;
    setSaving(true);
    try {
      // Use upsert so if there's no row for this user_id, it gets created:
      const { error } = await supabase
        .from("profiles")
        .upsert(
          { user_id: user.id, display_name: displayName },
          { onConflict: "user_id" }
        );

      if (error) {
        console.log("Upsert error:", error.message);
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

## 10) Optional: `app/index.tsx` (Default Redirection)

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

## 11) Run & Test

```bash
npx expo start --clear
```

Open in a simulator or real device:

1. **Sign Up**. If your Supabase project **does** require email confirmation, check your inbox.  
2. **Sign In**. You’ll land on `(protected)/profile`.  
3. **Watch Real-Time**: Because we subscribed to changes in `profile.tsx`, if you or another device changes your `profiles` row, you’ll see it instantly.  
4. **Edit Display Name** with `(protected)/edit-profile.tsx`:
   - We use `.upsert(...)` so your row is **guaranteed** to exist if it didn’t already.  
   - The real-time subscription in `profile.tsx` updates your UI immediately.  
5. **Sign Out**—the session is removed from SecureStore, returning you to the sign-in screen.

---

## 12) Recap & Next Steps

You now have:

- **Phase One**: Real Supabase Auth + secure session storage with Expo SecureStore.  
- **Phase Two**: A `profiles` table with RLS so each user can only see/update their own row.  
- **(Optional)** **Insert a default row** on sign-up, so no one ends up without a `profiles` row.  
- **(Optional)** **Real-time** subscription in `profile.tsx`, so changes appear instantly without manual re-fetch.

**Additional Steps**:

1. **Avatars**: Use Supabase Storage for profile pictures (store the image URL in `profiles.avatar_url`).  
2. **More Real-Time**: Extend the subscription approach to “posts” or “messages.”  
3. **Social Features**: Create a “posts” table with RLS for likes, comments, or follows.  
4. **Production**: Use [EAS Build](https://docs.expo.dev/eas/) to deploy your app to the stores, set up `.env` for secrets, etc.

By layering these enhancements (default row creation and real-time subscriptions) onto the existing phases, you further reduce chances of the “missing row” bug and give users an immediately responsive profile screen. Enjoy!
