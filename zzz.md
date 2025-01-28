Below is the **same** single, complete **ScriptHammer** tutorial—but now we follow the **best practice** of letting **Supabase** handle `profiles` row creation via a **DB Trigger**, instead of doing it in client code. This way:

- There’s **no** extra insert logic in our `signUp` function.  
- Every newly created user in `auth.users` automatically gets a matching `profiles` row.  
- We still store credentials in **`.env.local`** and use **Expo SecureStore**.  

Everything else (layouts, screens, RLS setup, etc.) stays the same. By the end, you’ll have zero “(none)” issues, and a clean separation where Supabase automatically manages `profiles` creation for new users.

---

# ScriptHammer: Single, Complete Tutorial With DB Trigger for `profiles`

## Table of Contents

1. [Set Up Your Expo Project](#1-set-up-your-expo-project)  
2. [Install Dependencies](#2-install-dependencies)  
3. [Create & Configure `.env.local`](#3-create--configure-envlocal)  
4. [File & Folder Structure](#4-file--folder-structure)  
5. [Supabase Setup](#5-supabase-setup)  
   - [Create or Confirm `profiles` Table](#create-or-confirm-profiles-table)  
   - [Enable RLS & Policies](#enable-rls--policies)  
   - [Create a DB Trigger for Auto-Inserting `profiles`](#create-a-db-trigger-for-auto-inserting-profiles)  
6. [Code: `supabaseClient.ts` (Connecting to Supabase)](#6-code-supabaseclientts-connecting-to-supabase)  
7. [Code: `app/_layout.tsx` (Single Top-Level Layout)](#7-code-app_layouttsx-single-top-level-layout)  
8. [Code: `context/auth.tsx` (Auth Context With **No** Extra Insert)](#8-code-contextauthtsx-auth-context-with-no-extra-insert)  
9. [Code: `(auth)` Folder (Sign In & Sign Up)](#9-code-auth-folder-sign-in--sign-up)  
   - [`(auth)/_layout.tsx`](#auth_layouttsx)  
   - [`(auth)/sign-in.tsx`](#authsign-intsx)  
   - [`(auth)/sign-up.tsx`](#authsign-uptsx)  
10. [Code: `(protected)` Folder (Protected Routes)](#10-code-protected-folder-protected-routes)  
    - [`(protected)/_layout.tsx`](#protected_layouttsx)  
    - [`(protected)/profile.tsx`](#protectedprofiletsx)  
    - [`(protected)/edit-profile.tsx`](#protectededit-profiletsx)  
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
npm install dotenv
```

- **@supabase/supabase-js**: the official Supabase client.  
- **expo-secure-store**: secure session storage on iOS/Android.  
- **dotenv**: to load `.env.local` variables (prefix `EXPO_PUBLIC_`) in Expo.

---

## 3) Create & Configure `.env.local`

At the **root** of your project (same level as `package.json`), create a file named **`.env.local`**:

```bash
EXPO_PUBLIC_SUPABASE_URL=https://YOUR-PROJECT.supabase.co
EXPO_PUBLIC_SUPABASE_ANON_KEY=YOUR-ANON-KEY
```

Replace with **your** real values. If you don’t want them in Git:

```bash
# .gitignore
.env.local
```

Because they start with `EXPO_PUBLIC_`, Expo automatically injects them at build time as `process.env.EXPO_PUBLIC_...`.

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

### Create or Confirm `profiles` Table

1. In [Supabase Dashboard](https://app.supabase.com/) → **Database** → **Tables**:
   ```sql
   create table if not exists profiles (
     id uuid primary key default uuid_generate_v4(),
     user_id uuid references auth.users not null,
     display_name text,
     created_at timestamp default now()
   );
   ```

### Enable RLS & Policies

1. Go to **Table Editor** → `profiles`.  
2. **Enable RLS**.  
3. **SELECT** policy:
   ```sql
   create policy "Select own profile"
   on public.profiles
   for select
   using ( auth.uid() = user_id );
   ```
4. **UPDATE** policy:
   ```sql
   create policy "Update own profile"
   on public.profiles
   for update
   using ( auth.uid() = user_id );
   ```

### Create a DB Trigger for Auto-Inserting `profiles`

Instead of manually inserting rows from the client, define a **trigger** that runs whenever a new user is added to `auth.users`. For example:

```sql
-- 1) Create a function that inserts into `profiles`
create or replace function handle_new_user()
returns trigger as $$
begin
  insert into public.profiles (user_id, display_name)
  values (new.id, '');
  return new;
end;
$$ language plpgsql security definer;

-- 2) Create a trigger that calls our function after user creation
create trigger on_auth_user_created
  after insert on auth.users
  for each row
  execute procedure handle_new_user();
```

- **Explanation**:  
  - Whenever a new row appears in `auth.users` (i.e., after sign-up), `handle_new_user()` runs, inserting a matching row into `profiles` with an empty `display_name`.  
  - This happens server-side, so your client code can remain simple.

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
      {/* One top-level Stack. Expo Router auto-provides NavigationContainer. */}
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

## 8) Code: `context/auth.tsx` (Auth Context **With No Extra Insert**)

Since the **DB trigger** creates rows in `profiles`, we no longer do that in `signUp`.

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

// Fallback for web if expo-secure-store isn't available
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
    // On mount, rehydrate from SecureStore or localStorage
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

  // Sign Up => DB Trigger auto-creates the 'profiles' row
  async function signUp(email: string, password: string) {
    setError(null);
    setLoading(true);
    try {
      const { error: signUpError } = await supabase.auth.signUp({ email, password });
      if (signUpError) {
        throw new Error(signUpError.message);
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
        throw new Error(signInError.message);
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

No need to insert into `profiles` here—our **DB trigger** does it!

---

## 9) Code: `(auth)` Folder (Sign In & Sign Up)

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
    // if signIn is successful, user is set by onAuthStateChange
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
    // DB Trigger automatically creates `profiles` row
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

  // If no user, redirect
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

### `(protected)/profile.tsx`

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

### `(protected)/edit-profile.tsx`

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

Open in iOS/Android/Web:

1. **Sign Up** with a new email.  
2. **DB Trigger** automatically creates a row in `profiles` (`display_name` = `""`).  
3. **Sign In**. You’ll see `(protected)/profile`. If `display_name` is empty, we show `(none)`.  
4. **Edit** the name in `(protected)/edit-profile`.  
5. **Row Level Security** ensures only your row is updated.  
6. **Sign Out** clears the session from SecureStore or localStorage.

No more “(none)” from missing rows—**Supabase** handles it.

---

## 13) Recap & Next Steps

You now have:

1. **Expo Router** + **Supabase** with a **DB Trigger** that auto-creates `profiles` rows—**no** client logic needed.  
2. **Row Level Security** ensures each user sees/updates only their row.  
3. **No** embedded keys in code—**`.env.local`** with `EXPO_PUBLIC_` keeps your anon key out of the repo.  
4. **expo-secure-store** (plus fallback for web) to store sessions securely.

### Next Steps

1. **Avatars**: Use [Supabase Storage](https://supabase.com/docs/guides/storage) for profile pictures, referencing `profiles.avatar_url`.  
2. **Real-Time**: Subscribe to live changes with `supabase.channel(...)`.  
3. **Social Features**: Add “posts,” “friends,” etc. with RLS-based security.

Enjoy your **ScriptHammer** project with a minimal, server-driven approach for `profiles` creation!
