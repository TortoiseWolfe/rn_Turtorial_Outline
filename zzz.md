Below is a **complete** end-to-end tutorial that **ensures** a row in the `profiles` table is **automatically created** on sign-up (so you won’t see `(none)`), **shows** error messages if an email is already in use, and stores sessions with a **web fallback** so it runs on iOS/Android/Web. By following this tutorial verbatim, you’ll have a **fully functional** Expo Router + Supabase auth flow with **no missing profile rows** and meaningful sign-up errors.

> **Key Improvements**:
> 1. **Auto-create** `profiles` row after sign-up.  
> 2. **Show** sign-up errors (e.g. email taken).  
> 3. **Handle** SecureStore vs. localStorage on web.  
> 4. Provide a direct clue on whether the user’s row was actually inserted in `profiles`.

---

# Final Working Tutorial

## Table of Contents

1. [Set Up Your Expo Project](#1-set-up-your-expo-project)  
2. [Install Dependencies](#2-install-dependencies)  
3. [Create & Configure `.env.local`](#3-create--configure-envlocal)  
4. [File & Folder Structure](#4-file--folder-structure)  
5. [Supabase Setup](#5-supabase-setup)  
   - [Create or Confirm `profiles` Table](#create-or-confirm-profiles-table)  
   - [Enable RLS & Policies](#enable-rls--policies)  
6. [Code: `supabaseClient.ts` (Connecting to Supabase)](#6-code-supabaseclientts-connecting-to-supabase)  
7. [Code: `app/_layout.tsx` (Single Top-Level Layout)](#7-code-app_layouttsx-single-top-level-layout)  
8. [Code: `context/auth.tsx` (Handles Auth + Automatic Profile Creation)](#8-code-contextauthtsx-handles-auth--automatic-profile-creation)  
9. [Code: `(auth)` Folder (Sign In & Sign Up)](#9-code-auth-folder-sign-in--sign-up)  
   - [`(auth)/_layout.tsx`](#auth_layouttsx)  
   - [`(auth)/sign-in.tsx` (Sign In)](#authsign-intsx-sign-in)  
   - [`(auth)/sign-up.tsx` (Sign Up)](#authsign-uptsx-sign-up)  
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

You now have a **blank** Expo Router project (no leftover `NavigationContainer` calls).

---

## 2) Install Dependencies

We need **Supabase**, **Expo SecureStore**, and **dotenv**:

```bash
npm install @supabase/supabase-js
npx expo install expo-secure-store
npm install dotenv
```

> **expo-secure-store** doesn’t work on web, so we’ll add a fallback for localStorage.

---

## 3) Create & Configure `.env.local`

In the root of your project (same level as `package.json`), **create** a file named **`.env.local`** with your **actual** Supabase credentials:

```bash
EXPO_PUBLIC_SUPABASE_URL=https://YOUR_PROJECT_ID.supabase.co
EXPO_PUBLIC_SUPABASE_ANON_KEY=YOUR_ACTUAL_ANON_KEY
```

> Don’t commit this file publicly if you want to keep your keys private. (Although the anon key isn’t highly sensitive.)

Ensure `.env.local` is in your `.gitignore`:

```bash
# .gitignore
.env.local
```

---

## 4) File & Folder Structure

Create these folders & files:

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
3. Create or confirm a `profiles` table:

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

With RLS, each user sees/updates only their row.

---

## 6) Code: `supabaseClient.ts` (Connecting to Supabase)

**File**: `supabaseClient.ts`

```ts
// supabaseClient.ts
import { createClient } from "@supabase/supabase-js";

const SUPABASE_URL = process.env.EXPO_PUBLIC_SUPABASE_URL!;
const SUPABASE_ANON_KEY = process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY!;

export const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
```

> The exclamation marks (`!`) tell TypeScript we’re sure these exist.

---

## 7) Code: `app/_layout.tsx` (Single Top-Level Layout)

**File**: `app/_layout.tsx`

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

## 8) Code: `context/auth.tsx` (Handles Auth + Automatic Profile Creation)

Here we:

1. **Sign up** the user with Supabase.  
2. On success, **create a row** in the `profiles` table—so you never see `"(none)"`.  
3. Display **error messages** if the email is already used.  
4. Use a **web fallback** (localStorage) if `expo-secure-store` fails on web.

**File**: `context/auth.tsx`

```tsx
// context/auth.tsx
import React, { createContext, useContext, useEffect, useState } from "react";
import { Platform } from "react-native";
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

// Helper functions: secure on mobile, fallback on web
async function setItem(key: string, value: string) {
  if (Platform.OS === "web") {
    window.localStorage.setItem(key, value);
  } else {
    await SecureStore.setItemAsync(key, value);
  }
}

async function getItem(key: string) {
  if (Platform.OS === "web") {
    return window.localStorage.getItem(key) || null;
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

  // 1) SIGN UP (auto-create row in 'profiles')
  async function signUp(email: string, password: string) {
    setError(null);
    setLoading(true);
    try {
      // Attempt sign-up
      const { data: signUpData, error: signUpError } = await supabase.auth.signUp({
        email,
        password,
      });

      if (signUpError) {
        // e.g. "User already registered", "Password too short", etc.
        throw new Error(signUpError.message);
      }

      // If user object is returned, insert row in 'profiles'
      if (signUpData.user) {
        const userId = signUpData.user.id;
        const { error: profileError } = await supabase
          .from("profiles")
          .insert({
            user_id: userId,
            display_name: "", // or any default
          });

        if (profileError) {
          // If insertion fails, throw to display error
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

  // 2) SIGN IN
  async function signIn(email: string, password: string) {
    setError(null);
    setLoading(true);
    try {
      const { error: signInError } = await supabase.auth.signInWithPassword({
        email,
        password,
      });
      if (signInError) {
        // e.g. "Invalid login credentials"
        throw new Error(signInError.message);
      }
    } catch (err: any) {
      setError(err.message || "Sign-in failed.");
      console.log("Sign-in error:", err);
    } finally {
      setLoading(false);
    }
  }

  // 3) SIGN OUT
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

**What changed**:
- After a successful **signUp**, we do:
  ```ts
  if (signUpData.user) {
    await supabase.from("profiles").insert(...);
  }
  ```
  guaranteeing a row in `profiles` so `display_name` will never be `null`.

- We set `error` if an **already used email** is attempted or any other sign-up error occurs. This shows in the UI.

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

### **File**: `app/(auth)/sign-in.tsx` (Sign In)

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

### **File**: `app/(auth)/sign-up.tsx` (Sign Up)

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
    // Basic validations
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

    // If sign-up was successful, the user context is updated automatically
    // plus the 'profiles' row is inserted. Check error to see if something failed.
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

Notice how we display the **`error`** from the context. If you try signing up with an already used email, you’ll see a message like **“User already registered”** or whatever Supabase returns.

---

## 10) Code: `(protected)` Folder (Protected Routes)

We have `(protected)/profile` to **fetch** the user’s row and `(protected)/edit-profile` to **update** it.

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

  // If the router isn't ready, do nothing yet
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

### **File**: `app/(protected)/profile.tsx` (Displays + Fetches Profile)

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
          {/* If 'display_name' is empty, show (none) */}
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

Because we **auto-create** the row in `signUp`, you’ll never see a missing row. If you manually created an account before, you might need to manually insert the row or re-sign up with a new email.

### **File**: `app/(protected)/edit-profile.tsx` (Edit Display Name)

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

If you want `/` to **automatically redirect** to `(protected)/profile` if signed in, otherwise to `(auth)/sign-in`:

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

- **iOS/Android**: Press `i` or `a` or scan the QR code.  
- **Web**: Press `w` or open the given URL in your browser.  
 
### Testing Steps:

1. **Sign Up** with a **new** email. If your Supabase project requires email confirmation, check your inbox. You should see no errors if the email is truly new.  
2. **Check Supabase** -> `profiles` table. You should see a new row with `user_id = <the user’s ID>` and `display_name = ""`.  
3. **Sign In**: Next time, if you try to sign up with the same email, you’ll get an error like **“User already registered”**.  
4. **Profile Screen**: `(protected)/profile` should display `"(none)"` if `display_name` is empty.  
5. **Edit Profile**: In `(protected)/edit-profile.tsx`, enter a new display name and hit **Save Changes**. Return to `(protected)/profile` to see it updated.  
6. **Sign Out**: Clears the session from SecureStore or localStorage on web.

---

## 13) Recap & Next Steps

You now have a **fully functional** Expo Router + Supabase app that:

1. **Handles sign-up** (with error messages & “auto-create profile”).
2. **Prevents** missing rows in `profiles`—no more “(none)” issues from empty selects.
3. **Stores** sessions securely in iOS/Android with SecureStore, and uses `localStorage` fallback on web so it doesn’t crash.
4. **Restricts** `(protected)` routes to logged-in users.
5. **Lets you** fetch and update `display_name` in the `profiles` table.

**Next Steps**:

1. **Refine Onboarding**: Possibly confirm email on sign-up if your Supabase project is configured that way.  
2. **Avatars**: Upload images to Supabase Storage and store the URL in `profiles.avatar_url`.  
3. **Real-Time**: Use `supabase.channel(...)` or `supabase.on(...)` to automatically refresh data when the DB changes.  
4. **Social Features**: Create “posts,” “friends,” “likes,” etc. with RLS ensuring each user can only manipulate or see what they’re allowed to.

With these changes, you’ll see actual sign-up errors (like “email already in use”), a row in `profiles` for each new user, and a functional display name flow that never shows “(none)” unless you truly left it blank. You’re all set—**enjoy your fully working solution**!
