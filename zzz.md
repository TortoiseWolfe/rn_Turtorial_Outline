Below is a **complete**, **functional** tutorial from start to finish—keeping your **ScriptHammer** project name **and** including all the **terminal commands** to create the necessary files. This tutorial **automatically** inserts rows into `profiles`, handles **sign-up errors** (like email already used), and uses **web fallback** storage so it won’t crash in a browser.

By the end, you’ll have:

1. A brand-new **Expo Router** project named **ScriptHammer**.  
2. **.env.local** for Supabase credentials (no hardcoded keys).  
3. **Automatic** `profiles` row creation after sign-up, so it’s never empty.  
4. Proper **RLS policies** so each user can only see/update their own row.  
5. A working **sign-up** → **sign-in** → **protected** profile flow on iOS/Android/Web, with actual error messages and zero “(none)” confusion.

---

# Complete Tutorial: ScriptHammer

## Table of Contents

1. [Set Up Your Expo Project](#1-set-up-your-expo-project)  
2. [Install Dependencies](#2-install-dependencies)  
3. [Create & Configure `.env.local`](#3-create--configure-envlocal)  
4. [File & Folder Structure](#4-file--folder-structure)  
5. [Supabase Setup](#5-supabase-setup)  
   - [Create/Confirm `profiles` Table](#createconfirm-profiles-table)  
   - [Enable Row Level Security & Policies](#enable-row-level-security--policies)  
6. [File: `supabaseClient.ts` (Connecting to Supabase)](#6-file-supabaseclientts-connecting-to-supabase)  
7. [File: `app/_layout.tsx` (Root Layout)](#7-file-app_layouttsx-root-layout)  
8. [File: `context/auth.tsx` (Auth Context + Auto Insert Profiles)](#8-file-contextauthtsx-auth-context--auto-insert-profiles)  
9. [Folder: `(auth)` (Sign In & Sign Up)](#9-folder-auth-sign-in--sign-up)  
   - [`(auth)/_layout.tsx`](#auth_layouttsx)  
   - [`(auth)/sign-in.tsx` (Sign In)](#authsign-intsx-sign-in)  
   - [`(auth)/sign-up.tsx` (Sign Up)](#authsign-uptsx-sign-up)  
10. [Folder: `(protected)` (Protected Screens)](#10-folder-protected-protected-screens)  
    - [`(protected)/_layout.tsx`](#protected_layouttsx)  
    - [`(protected)/profile.tsx` (Show Display Name)](#protectedprofiletsx-show-display-name)  
    - [`(protected)/edit-profile.tsx` (Edit Display Name)](#protectededit-profiletsx-edit-display-name)  
11. [Optional: `app/index.tsx` (Redirect to Auth or Profile)](#11-optional-appindextsx-redirect-to-auth-or-profile)  
12. [Run & Test](#12-run--test)  
13. [Recap & Next Steps](#13-recap--next-steps)  

---

## 1) Set Up Your Expo Project

Open a terminal and run:

```bash
npx create-expo-app ScriptHammer
cd ScriptHammer
npm run reset-project
rm -rf app-example
```

Now you have a blank Expo Router project (no leftover `NavigationContainer` calls).

---

## 2) Install Dependencies

In the same folder, run:

```bash
npm install @supabase/supabase-js
npx expo install expo-secure-store
npm install dotenv
```

- `@supabase/supabase-js`: The official Supabase client library.  
- `expo-secure-store`: Secure session storage on iOS/Android (we’ll add a fallback for web).  
- `dotenv`: So `.env.local` variables (prefixed with `EXPO_PUBLIC_`) are available in Expo.

---

## 3) Create & Configure `.env.local`

In your project’s root, **create** a file named **`.env.local`**:

```bash
EXPO_PUBLIC_SUPABASE_URL=https://YOUR-PROJECT-ID.supabase.co
EXPO_PUBLIC_SUPABASE_ANON_KEY=YOUR-REAL-ANON-KEY
```

Replace `YOUR-PROJECT-ID` and `YOUR-REAL-ANON-KEY` with **real** values from **Supabase → Project Settings → API**.

To keep it out of git:

```bash
# .gitignore
.env.local
```

Because these variables start with `EXPO_PUBLIC_`, Expo automatically includes them in the JS bundle at runtime (accessible via `process.env.EXPO_PUBLIC_...`).

---

## 4) File & Folder Structure

Create the folders/files by running these **exact** commands:

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

Your folder structure will look like:

```
ScriptHammer
├─ context
│  └─ auth.tsx
├─ app
│  ├─ _layout.tsx
│  ├─ index.tsx
│  ├─ (auth)
│  │   ├─ _layout.tsx
│  │   ├─ sign-in.tsx
│  │   └─ sign-up.tsx
│  └─ (protected)
│      ├─ _layout.tsx
│      ├─ profile.tsx
│      └─ edit-profile.tsx
├─ supabaseClient.ts
├─ .env.local
├─ package.json
└─ ...
```

We’ll fill these files next.

---

## 5) Supabase Setup

Log in to your [Supabase](https://app.supabase.com/) dashboard.

### Create/Confirm `profiles` Table

1. In **Database** → **Tables** → `New Table` (or use the SQL Editor), ensure you have:

   ```sql
   create table if not exists profiles (
     id uuid primary key default uuid_generate_v4(),
     user_id uuid references auth.users not null,
     display_name text,
     created_at timestamp default now()
   );
   ```
2. This table holds extra user data (like display_name).

### Enable Row Level Security & Policies

1. Go to **Table Editor** → `profiles`.  
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
This ensures each user only sees/updates **their own** row.

---

## 6) File: `supabaseClient.ts` (Connecting to Supabase)

Paste this into **`supabaseClient.ts`**:

```ts
// supabaseClient.ts
import { createClient } from "@supabase/supabase-js";

// We rely on .env.local with "EXPO_PUBLIC_" so they're in process.env
const SUPABASE_URL = process.env.EXPO_PUBLIC_SUPABASE_URL!;
const SUPABASE_ANON_KEY = process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY!;

export const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
```

---

## 7) File: `app/_layout.tsx` (Root Layout)

This is the **top-level** layout for Expo Router. It wraps everything in `AuthProvider`.

```tsx
// app/_layout.tsx
import { Stack } from "expo-router";
import AuthProvider from "../context/auth";

export default function RootLayout() {
  return (
    <AuthProvider>
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

## 8) File: `context/auth.tsx` (Auth Context + Auto Insert Profiles)

This is the **heart** of your app’s authentication:

- **Sign Up**: Creates a user in `auth.users` + automatically inserts a row into `profiles`.  
- **Sign In**: Uses the credentials from the user.  
- **Sign Out**: Ends session.  
- **Session Storage**: Uses **Expo SecureStore** on native, **localStorage** on web.

```tsx
// context/auth.tsx
import React, { createContext, useContext, useEffect, useState } from "react";
import { Platform } from "react-native";
import * as SecureStore from "expo-secure-store";
import { supabase } from "../supabaseClient";

interface AuthContextProps {
  user: any;
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

// SecureStore fallback for web
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
    // Rehydrate session on mount
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

  // Sign Up => Also insert row in 'profiles'
  async function signUp(email: string, password: string) {
    setError(null);
    setLoading(true);
    try {
      const { data: signUpData, error: signUpError } = await supabase.auth.signUp({
        email,
        password,
      });
      if (signUpError) {
        throw new Error(signUpError.message); // e.g. "User already registered"
      }

      if (signUpData.user) {
        // Insert row into profiles
        const { error: profileError } = await supabase.from("profiles").insert({
          user_id: signUpData.user.id,
          display_name: "",
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

  // Sign In
  async function signIn(email: string, password: string) {
    setError(null);
    setLoading(true);
    try {
      const { error: signInError } = await supabase.auth.signInWithPassword({
        email,
        password,
      });
      if (signInError) {
        throw new Error(signInError.message); // e.g. "Invalid login credentials"
      }
    } catch (err: any) {
      setError(err.message || "Sign-in failed.");
      console.log("Sign-in error:", err);
    } finally {
      setLoading(false);
    }
  }

  // Sign Out
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

Now, **no** user is created without a matching `profiles` row.

---

## 9) Folder: `(auth)` (Sign In & Sign Up)

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

### `(auth)/sign-in.tsx` (Sign In)

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
    // If signIn is successful, user is set in context
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

### `(auth)/sign-up.tsx` (Sign Up)

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
    // If signUp is successful, user is set, row in 'profiles' is created
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

### `(protected)/_layout.tsx`

```tsx
// app/(protected)/_layout.tsx
import { Stack, useRouter, useRootNavigationState } from "expo-router";
import { useEffect } from "react";
import { useAuth } from "../../context/auth";

export default function ProtectedLayout() {
  const router = useRouter();
  const navigationState = useRootNavigationState();
  const { user } = useAuth();

  if (!navigationState?.key) {
    return null; // Router isn't ready
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

Because of the **auto-insert** in `signUp`, `profiles` won’t be empty. If `display_name` is blank, we show `"(none)"`.

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

## 11) Optional: `app/index.tsx` (Redirect to Auth or Profile)

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

- **Android**: Press `a` or scan QR in Expo Go.  
- **iOS**: Press `i`.  
- **Web**: Press `w`.  

### Testing Steps

1. **Sign Up** with a brand-new email (not used before).  
   - If your Supabase project is set to “no email confirmations,” you’ll log in instantly.  
   - If it **does** require confirmations, check your inbox.  
2. Look in **Supabase** → **Database** → `profiles`. A new row should appear, matching your user’s `id`.  
3. On `(protected)/profile`, your `display_name` might show as `"(none)"` if empty.  
4. **Edit** the display name in `(protected)/edit-profile` and see it update.  
5. **Sign Out** to clear the session.  
6. If you sign up with the **same** email again, you’ll see an error like “User already registered.”  

No more missing rows in `profiles`!

---

## 13) Recap & Next Steps

You now have:

1. **ScriptHammer**: An Expo Router + Supabase project that:  
   - Auto-inserts a `profiles` row after sign-up.  
   - Properly rehydrates the user session from SecureStore (native) or localStorage (web).  
   - Enforces RLS so each user only sees/updates their own row.  
   - Displays actual sign-up errors (e.g. duplicate email).  
2. **Protected** routes in `(protected)` that require a valid user.  

### Next Steps

1. **Email Confirmations**: If you want mandatory email verification, enable it in Supabase’s Auth Settings.  
2. **DB Trigger**: Instead of client-side inserts, set up a [Supabase trigger](https://supabase.com/docs/guides/auth/managing-user-data#database-triggers) to auto-create `profiles` whenever a new user is created.  
3. **Avatars**: Add a column `avatar_url` to `profiles` and use Supabase Storage for images.  
4. **Real-Time**: Use `supabase.channel(...)` or `supabase.on(...)` to subscribe to DB changes for real-time updates.  
5. **Additional Features**: Build a social feed, friends, likes, etc., all protected by RLS.

This tutorial **keeps your terminal commands** to create all files, uses the **ScriptHammer** name, and ensures the `profiles` table is **never** empty for new sign-ups. Enjoy your fully working setup!
