# ScriptHammer:

## Table of Contents

1. [Set Up Your Expo Project](#1-set-up-your-expo-project)  
2. [Install Dependencies](#2-install-dependencies)  
3. [Create & Configure `.env.local`](#3-create--configure-envlocal)  
4. [File & Folder Structure](#4-file--folder-structure)  
5. [Supabase Setup](#5-supabase-setup)  
   - [Create or Confirm `profiles` Table](#create-or-confirm-profiles-table)  
   - [Enable RLS & Policies](#enable-rls--policies)  
   - [Create a DB Trigger for Auto-Inserting `profiles`](#create-a-db-trigger-for-auto-inserting-profiles)  
   - [Enable Realtime for `profiles`](#enable-realtime-for-profiles)  
6. [Code: `supabaseClient.ts` (Connecting to Supabase)](#6-code-supabaseclientts-connecting-to-supabase)  
7. [Code: `app/_layout.tsx` (Single Top-Level Layout)](#7-code-app_layouttsx-single-top-level-layout)  
8. [Code: `app/index.tsx` (Redirect at Launch)](#8-code-appindextsx-redirect-at-launch)  
9. [Code: `context/auth.tsx` (Auth Context, Minimal SecureStore)](#9-code-contextauthtsx-auth-context-minimal-securestore)  
10. [Code: `(auth)` Folder (Sign In & Sign Up)](#10-code-auth-folder-sign-in--sign-up)  
    - [`(auth)/_layout.tsx`](#auth_layouttsx)  
    - [`(auth)/sign-in.tsx`](#authsign-intsx)  
    - [`(auth)/sign-up.tsx`](#authsign-uptsx)  
11. [Code: `(protected)` Folder (Protected Routes + Real-Time)](#11-code-protected-folder-protected-routes--real-time)  
    - [`(protected)/_layout.tsx`](#protected_layouttsx)  
    - [`(protected)/profile.tsx` (Displays + Subscribes)](#protectedprofiletsx-displays--subscribes)  
    - [`(protected)/edit-profile.tsx`](#protectededit-profiletsx)  
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

- **@supabase/supabase-js**: the official Supabase client library.  
- **expo-secure-store**: secure session storage on iOS/Android.  
- **dotenv**: to load `.env.local` variables prefixed with `EXPO_PUBLIC_`.

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

In [Supabase Dashboard](https://app.supabase.com/) → **Database** → **Tables**:

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

Rather than manually inserting rows from the client, define a **trigger** that runs whenever a new user is added to `auth.users`. For example:

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

### Enable Realtime for `profiles`

1. Go to **Table Editor** → click **profiles** → ensure “Enable Realtime” is on (if your project plan supports it).  
2. Make sure you have [Replication / Realtime config](https://supabase.com/docs/guides/realtime) set up. Typically it’s on by default for new projects.

Now each time a new user is created in `auth.users`, `profiles` gets a matching row, and changes to `profiles` can emit real-time events.

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

Here we **hide** the header and add a **StatusBar** for proper insets on iOS/Android.

```tsx
// app/_layout.tsx
import { Stack } from "expo-router";
import AuthProvider from "../context/auth";
import { StatusBar } from "expo-status-bar";

export default function RootLayout() {
  return (
    <AuthProvider>
      {/* 
        One top-level Stack.
        Expo Router auto-provides NavigationContainer behind the scenes.
      */}
      <Stack
        screenOptions={{
          headerShown: false,
        }}
      />
      {/* Ensure the OS status bar is managed */}
      <StatusBar style="auto" />
    </AuthProvider>
  );
}
```

---

## 8) Code: `app/index.tsx` (Redirect at Launch)

We no longer consider this optional. It’s your **home screen**, redirecting to either `(protected)/profile` (if user is logged in) or `(auth)/sign-in` (if not).

```tsx
// app/index.tsx
import { Redirect } from "expo-router";
import { useAuth } from "../context/auth";

export default function IndexScreen() {
  const { user } = useAuth();

  // If user is logged in, go to profile; otherwise, sign in
  return user ? (
    <Redirect href="/(protected)/profile" />
  ) : (
    <Redirect href="/(auth)/sign-in" />
  );
}
```

---

## 9) Code: `context/auth.tsx` (Auth Context, Minimal SecureStore)

We’re storing only the `access_token` and `refresh_token` in SecureStore, not the full session object.

```tsx
// context/auth.tsx
import React, { createContext, useContext, useEffect, useState } from "react";
import { Platform } from "react-native";
import * as SecureStore from "expo-secure-store";
import { supabase } from "../supabaseClient";
import { Session } from "@supabase/supabase-js";

interface AuthContextProps {
  user: any;
  loading: boolean;
  error: string | null;
  signUp: (email: string, password: string) => Promise<void>;
  signIn: (email: string, password: string) => Promise<void>;
  signOut: () => Promise<void>;
}

const SESSION_KEY_ACCESS = "supabaseAccessToken";
const SESSION_KEY_REFRESH = "supabaseRefreshToken";
const AuthContext = createContext<AuthContextProps>({
  user: null,
  loading: false,
  error: null,
  signUp: async () => {},
  signIn: async () => {},
  signOut: async () => {},
});

// Helpers for storing small strings only
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

  // Attempt to restore tokens on mount
  useEffect(() => {
    (async () => {
      try {
        const accessToken = await getItem(SESSION_KEY_ACCESS);
        const refreshToken = await getItem(SESSION_KEY_REFRESH);

        if (accessToken && refreshToken) {
          // We can reconstruct a partial session, or call supabase.auth.setSession
          // But in Supabase v2, we typically do signInWithRefreshToken
          const { data, error } = await supabase.auth.setSession({
            access_token: accessToken,
            refresh_token: refreshToken,
          });
          if (data?.session) {
            setUser(data.session.user);
          }
          if (error) {
            console.log("Error restoring session:", error.message);
          }
        }
      } catch (err) {
        console.log("Failed to load tokens:", err);
      }
      setLoading(false);
    })();

    // Listen for auth changes
    const {
      data: { subscription },
    } = supabase.auth.onAuthStateChange((event, session) => {
      console.log("Supabase auth event:", event);
      if (session?.user) {
        setUser(session.user);
        storeTokens(session);
      } else {
        setUser(null);
        clearTokens();
      }
    });

    return () => {
      subscription.unsubscribe();
    };
  }, []);

  // Store just the access and refresh tokens
  async function storeTokens(session: Session) {
    if (!session) return;
    const { access_token, refresh_token } = session;
    if (access_token) await setItem(SESSION_KEY_ACCESS, access_token);
    if (refresh_token) await setItem(SESSION_KEY_REFRESH, refresh_token);
  }

  // Clear tokens on sign-out
  async function clearTokens() {
    await deleteItem(SESSION_KEY_ACCESS);
    await deleteItem(SESSION_KEY_REFRESH);
  }

  // Sign Up => DB Trigger automatically creates the 'profiles' row
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
      const { data, error: signInError } = await supabase.auth.signInWithPassword({
        email,
        password,
      });
      if (signInError) {
        throw new Error(signInError.message);
      }
      if (data?.session?.user) {
        setUser(data.session.user);
        storeTokens(data.session);
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

---

## 10) Code: `(auth)` Folder (Sign In & Sign Up)

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

Notice we removed `justifyContent: "center"` in favor of a simpler layout with **short margin** at the top.

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
  container: {
    paddingHorizontal: 20,
    paddingTop: 40,
  },
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

Similar layout updates.

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
    // DB trigger automatically creates profiles row
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
  container: {
    paddingHorizontal: 20,
    paddingTop: 40,
  },
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

## 11) Code: `(protected)` Folder (Protected Routes + Real-Time)

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

### `(protected)/profile.tsx` (Displays + Subscribes)

```tsx
// app/(protected)/profile.tsx
import { View, Text, Button, ActivityIndicator, StyleSheet } from "react-native";
import { useState, useEffect } from "react";
import { useRouter } from "expo-router";
import { useAuth } from "../../context/auth";
import { supabase } from "../../supabaseClient";

export default function ProfileScreen() {
  const { user, signOut } = useAuth();
  const [profile, setProfile] = useState<any>(null);
  const [loadingProfile, setLoadingProfile] = useState(false);
  const router = useRouter();

  // 1) Initial fetch
  async function fetchProfile() {
    if (!user?.id) return;
    setLoadingProfile(true);
    try {
      const { data, error } = await supabase
        .from("profiles")
        .select("*")
        .eq("user_id", user.id)
        .single();
      if (!error && data) {
        setProfile(data);
      } else if (error) {
        console.log("Error fetching profile:", error.message);
      }
    } finally {
      setLoadingProfile(false);
    }
  }

  // 2) On mount, fetch once
  useEffect(() => {
    fetchProfile();
  }, [user?.id]);

  // 3) Real-time subscription: watch only our row
  useEffect(() => {
    if (!user?.id) return;

    const channel = supabase
      .channel("profile-changes") // pick any unique name
      .on(
        "postgres_changes",
        {
          event: "*", // or "UPDATE", "INSERT", "DELETE"
          schema: "public",
          table: "profiles",
          filter: `user_id=eq.${user.id}`,
        },
        (payload) => {
          console.log("Realtime update on 'profiles':", payload);
          // Easiest approach: re-fetch to update local state
          fetchProfile();
        }
      )
      .subscribe();

    // Cleanup on unmount
    return () => {
      supabase.removeChannel(channel);
    };
  }, [user?.id]);

  async function handleSignOut() {
    await signOut();
    router.replace("(auth)/sign-in");
  }

  if (!user) {
    return (
      <View style={styles.container}>
        <Text>No user found, please sign in.</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
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

const styles = StyleSheet.create({
  container: {
    paddingHorizontal: 20,
    paddingTop: 40,
  },
});
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
  container: {
    paddingHorizontal: 20,
    paddingTop: 40,
  },
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

## 12) Run & Test

```bash
npx expo start --clear
```

Open in iOS/Android/Web:

1. **Sign Up** with a new email.  
   - The **DB Trigger** automatically creates a row in `profiles`.  
2. **Sign In** → `(protected)/profile` fetches your row.  
3. **Edit** the display name in `(protected)/edit-profile`. You’ll see it update.  
4. **Real-Time**: If you open **another** instance or the **Supabase Dashboard** → `profiles` and change `display_name`, the subscription in `(protected)/profile` triggers an update automatically.  
5. **Sign Out**—clears the tokens from SecureStore or localStorage.

---

## 13) Recap & Updated Roadmap

You now have:

1. **DB Trigger**: For auto-creating `profiles`.  
2. **Real-Time Subscription**: In `(protected)/profile` for instant changes.  
3. **RLS** so each user only sees/updates their row.  
4. **`.env.local`** with `EXPO_PUBLIC_` prefix for keys.  
5. **Minimal** SecureStore usage (only `access_token` & `refresh_token`).

All of the above are **complete**.

Below is our extended roadmap, with completed tasks **struck through** and next tasks in recommended order (inspired by Bonfire, Yii2, WordPress):

1. ~~**Real Auth** backend integration (Supabase)~~  
2. ~~**RLS** & DB triggers (auto-create `profiles`)~~  
3. ~~**Minimal tokens** in SecureStore (avoid 2048-byte limit)~~  
4. ~~**Real-time subscription** for `profiles`~~  

**Next Steps**:

5. **Admin Dashboard (Web Interface)**  
   - A custom admin or “back office” for user management, content moderation, etc.  

6. **CLI / CRUD Scaffolding Tools**  
   - Generate tables, endpoints, or “modules” quickly (like Bonfire Builder or Yii2 Gii).  

7. **Advanced Role/Permission System**  
   - Beyond basic RLS checks, introduce `admin`, `editor`, `moderator` roles, etc.  

8. **Additional Social Features**  
   - Posts, comments, likes/follows.  
   - Real-time feed, direct messages, file uploads (Supabase Storage).  

9. **Deployment & Production Hardening**  
   - EAS builds for iOS/Android, robust hosting, environment variables, logging/monitoring, push notifications, etc.  

By layering features **incrementally**, each piece (auth, admin, scaffolding, roles, social) builds on a **solid foundation**. Your “ScriptHammer” app is now a **fully functional** Expo + Supabase project with triggers, RLS, minimal SecureStore usage, and real-time updates—ready to evolve into a production-scale social application!
