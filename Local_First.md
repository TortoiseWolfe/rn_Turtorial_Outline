# ScriptHammer - Offline/Local-First Integration

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
7. [Code: `localdb.ts` (Local SQLite Helpers)](#7-code-localdbts-local-sqlite-helpers)  
8. [Code: `context/auth.tsx` (Auth Context)](#8-code-contextauthtsx-auth-context)  
9. [Code: `context/offline.tsx` (Offline Context)](#9-code-contextofflinetsx-offline-context)  
10. [Code: `app/_layout.tsx` (Single Top-Level Layout)](#10-code-app_layouttsx-single-top-level-layout)  
11. [Code: `app/index.tsx` (Redirect at Launch)](#11-code-appindextsx-redirect-at-launch)  
12. [Code: `(auth)` Folder (Sign In & Sign Up)](#12-code-auth-folder-sign-in--sign-up)  
    - [`(auth)/_layout.tsx`](#auth_layouttsx)  
    - [`(auth)/sign-in.tsx`](#authsign-intsx)  
    - [`(auth)/sign-up.tsx`](#authsign-uptsx)  
13. [Code: `(protected)` Folder (Protected Routes)](#13-code-protected-folder-protected-routes)  
    - [`(protected)/_layout.tsx`](#protected_layouttsx)  
    - [`(protected)/profile.tsx` (Offline-First Display)](#protectedprofiletsx-offline-first-display)  
    - [`(protected)/edit-profile.tsx` (Offline-First Edit)](#protectededit-profiletsx-offline-first-edit)  
14. [Run & Test](#14-run--test)  
15. [Recap & Extended Roadmap](#15-recap--extended-roadmap)  

> **Important**: This tutorial extends our baseline Supabase + Expo Router project with **offline** capabilities. If you haven’t yet followed a prior ScriptHammer iteration, **start here**—this tutorial is all-inclusive.  

---

## 1) Set Up Your Expo Project

Create a new Expo project using **Expo Router**:

```bash
npx create-expo-app ScriptHammer
cd ScriptHammer
npm run reset-project
rm -rf app-example
```

You now have a blank Expo Router project (no leftover `NavigationContainer` calls).

---

## 2) Install Dependencies

We’ll need:

```bash
npm install @supabase/supabase-js
npx expo install expo-secure-store
npm install dotenv
npx expo install expo-sqlite
```

1. **@supabase/supabase-js**: official Supabase client.  
2. **expo-secure-store**: secure token storage on iOS/Android.  
3. **dotenv**: load `.env.local` variables (prefixed with `EXPO_PUBLIC_`).  
4. **expo-sqlite**: built-in local database for offline caching.

*(Optional)* If you want to detect network changes automatically, you can also:  

```bash
npm install @react-native-community/netinfo
```

---

## 3) Create & Configure `.env.local`

At the **root** of your project (same level as `package.json`), create a file named **`.env.local`**:

```bash
EXPO_PUBLIC_SUPABASE_URL=https://YOUR-PROJECT.supabase.co
EXPO_PUBLIC_SUPABASE_ANON_KEY=YOUR-ANON-KEY
```

Replace those with **your** real values from your Supabase Dashboard. Then, ensure you **ignore** the file in Git:

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
touch context/offline.tsx

mkdir -p app/\(auth\)
touch app/\(auth\)/_layout.tsx
touch app/\(auth\)/sign-in.tsx
touch app/\(auth\)/sign-up.tsx

mkdir -p app/\(protected\)
touch app/\(protected\)/_layout.tsx
touch app/\(protected\)/profile.tsx
touch app/\(protected\)/edit-profile.tsx

touch supabaseClient.ts
touch localdb.ts

touch app/_layout.tsx
touch app/index.tsx
touch .env.local
code .
```

---

## 5) Supabase Setup

### Create or Confirm `profiles` Table

In [Supabase Dashboard](https://app.supabase.com/) → **Database** → **Tables**, run:

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

*(No special INSERT policy needed if you’re only updating your own profile. The “auto-insert” is handled by a DB trigger below.)*

### Create a DB Trigger for Auto-Inserting `profiles`

Whenever a new user signs up, automatically add a row in `profiles`:

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

1. In your Supabase Dashboard → **Table Editor** → `profiles`, ensure **“Enable Realtime”** is turned on (if your plan supports it).  
2. Confirm [Replication / Realtime config](https://supabase.com/docs/guides/realtime) is correct. By default, new projects have it enabled.

---

## 6) Code: `supabaseClient.ts` (Connecting to Supabase)

Create a single Supabase client for the entire project:

```ts
// supabaseClient.ts
import { createClient } from "@supabase/supabase-js";

const SUPABASE_URL = process.env.EXPO_PUBLIC_SUPABASE_URL!;
const SUPABASE_ANON_KEY = process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY!;

export const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
```

---

## 7) Code: `localdb.ts` (Local SQLite Helpers)

We’ll create a **local** SQLite table called `local_profiles` that **mirrors** the remote `profiles` table but with a simplified schema. This allows us to read/write offline, then sync to Supabase when online.

```ts
// localdb.ts
import * as SQLite from "expo-sqlite";

// Open or create the local database
const db = SQLite.openDatabase("ScriptHammer.db");

/**
 * Run on app start to ensure local_profiles table is created
 */
export async function setupLocalDatabase() {
  return new Promise<void>((resolve, reject) => {
    db.transaction((tx) => {
      tx.executeSql(
        `CREATE TABLE IF NOT EXISTS local_profiles (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          user_id TEXT UNIQUE,
          display_name TEXT,
          updated_at TEXT
        );`,
        [],
        () => resolve(),
        (_, error) => {
          console.log("Error creating local_profiles table:", error);
          reject(error);
          return false;
        }
      );
    });
  });
}

/**
 * Upsert (insert or update) a local profile
 */
export async function upsertLocalProfile(
  userId: string,
  displayName: string,
  updatedAt: string
) {
  return new Promise<void>((resolve, reject) => {
    db.transaction((tx) => {
      tx.executeSql(
        `
        INSERT INTO local_profiles (user_id, display_name, updated_at)
        VALUES (?, ?, ?)
        ON CONFLICT(user_id) DO UPDATE 
        SET display_name=excluded.display_name,
            updated_at=excluded.updated_at
      `,
        [userId, displayName, updatedAt],
        () => resolve(),
        (_, error) => {
          console.log("Error upserting local profile:", error);
          reject(error);
          return false;
        }
      );
    });
  });
}

/**
 * Fetch a local profile by user_id
 */
export async function getLocalProfile(userId: string) {
  return new Promise<any>((resolve, reject) => {
    db.transaction((tx) => {
      tx.executeSql(
        `SELECT * FROM local_profiles WHERE user_id = ? LIMIT 1`,
        [userId],
        (_, result) => {
          if (result.rows.length > 0) {
            resolve(result.rows.item(0));
          } else {
            resolve(null);
          }
        },
        (_, error) => {
          console.log("Error fetching local profile:", error);
          reject(error);
          return false;
        }
      );
    });
  });
}

/**
 * Update local display name + updated_at
 */
export async function updateLocalDisplayName(userId: string, displayName: string) {
  const now = new Date().toISOString();
  return new Promise<void>((resolve, reject) => {
    db.transaction((tx) => {
      tx.executeSql(
        `UPDATE local_profiles SET display_name=?, updated_at=? WHERE user_id=?`,
        [displayName, now, userId],
        () => resolve(),
        (_, error) => {
          console.log("Error updating local profile:", error);
          reject(error);
          return false;
        }
      );
    });
  });
}

export { db };
```

- **`setupLocalDatabase()`**: called once on app startup to ensure the table is created.  
- **`upsertLocalProfile()`**: “insert or update” logic.  
- **`getLocalProfile()`**: returns row data from local DB.  
- **`updateLocalDisplayName()`**: for local edits.  

---

## 8) Code: `context/auth.tsx` (Auth Context)

Manages Supabase **auth** sessions and stores just the `access_token` and `refresh_token` in SecureStore. (Same as our prior ScriptHammer iteration.)

```tsx
// context/auth.tsx
import React, { createContext, useContext, useEffect, useState } from "react";
import { Platform } from "react-native";
import * as SecureStore from "expo-secure-store";
import { supabase } from "../supabaseClient";
import { Session } from "@supabase/supabase-js";

interface AuthContextProps {
  user: any; // Typically Supabase "User" type
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

  // Restore tokens on mount
  useEffect(() => {
    (async () => {
      try {
        const accessToken = await getItem(SESSION_KEY_ACCESS);
        const refreshToken = await getItem(SESSION_KEY_REFRESH);

        if (accessToken && refreshToken) {
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

    // Listen for subsequent auth changes
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

  async function storeTokens(session: Session) {
    const { access_token, refresh_token } = session;
    if (access_token) await setItem(SESSION_KEY_ACCESS, access_token);
    if (refresh_token) await setItem(SESSION_KEY_REFRESH, refresh_token);
  }

  async function clearTokens() {
    await deleteItem(SESSION_KEY_ACCESS);
    await deleteItem(SESSION_KEY_REFRESH);
  }

  // Sign Up (DB trigger auto-creates row in profiles)
  async function signUp(email: string, password: string) {
    setError(null);
    setLoading(true);
    try {
      const { error: signUpError } = await supabase.auth.signUp({ email, password });
      if (signUpError) throw new Error(signUpError.message);
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
      if (signInError) throw new Error(signInError.message);
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
      if (signOutError) throw new Error(signOutError.message);
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

## 9) Code: `context/offline.tsx` (Offline Context)

This is where we integrate **localdb** + Supabase. We want:

1. A local copy of the user’s profile in `local_profiles`.  
2. Automatic real-time updates from Supabase → local.  
3. Edits to the profile to be stored **locally first**, then synced to Supabase.

```tsx
// context/offline.tsx
import React, {
  createContext,
  useContext,
  useState,
  useEffect,
  useCallback,
} from "react";
import { supabase } from "../supabaseClient";
import {
  setupLocalDatabase,
  getLocalProfile,
  upsertLocalProfile,
  updateLocalDisplayName,
} from "../localdb";
import { useAuth } from "./auth";

interface OfflineContextProps {
  localProfile: any; // user's local row
  syncing: boolean;
  fetchRemoteProfile: () => Promise<void>;
  updateLocalThenRemote: (newDisplayName: string) => Promise<void>;
}

const OfflineContext = createContext<OfflineContextProps>({
  localProfile: null,
  syncing: false,
  fetchRemoteProfile: async () => {},
  updateLocalThenRemote: async () => {},
});

export function OfflineProvider({ children }: { children: React.ReactNode }) {
  const { user } = useAuth();
  const [localProfile, setLocalProfile] = useState<any>(null);
  const [syncing, setSyncing] = useState(false);

  // 1) Setup local DB on mount
  useEffect(() => {
    (async () => {
      try {
        await setupLocalDatabase();
      } catch (err) {
        console.log("Error setting up local DB:", err);
      }
    })();
  }, []);

  // 2) Load local profile whenever user changes (e.g., on login)
  useEffect(() => {
    if (!user) {
      setLocalProfile(null);
      return;
    }
    (async () => {
      const lp = await getLocalProfile(user.id);
      setLocalProfile(lp);
    })();
  }, [user]);

  // 3) Force a fetch from remote → local
  const fetchRemoteProfile = useCallback(async () => {
    if (!user?.id) return;
    try {
      setSyncing(true);
      const { data, error } = await supabase
        .from("profiles")
        .select("*")
        .eq("user_id", user.id)
        .single();
      if (error) {
        console.log("Error fetching from Supabase:", error.message);
        return;
      }
      if (data) {
        const remoteUpdatedAt = new Date().toISOString(); // naive approach
        await upsertLocalProfile(data.user_id, data.display_name ?? "", remoteUpdatedAt);
        const updatedLocal = await getLocalProfile(user.id);
        setLocalProfile(updatedLocal);
      }
    } catch (err) {
      console.log("fetchRemoteProfile error:", err);
    } finally {
      setSyncing(false);
    }
  }, [user]);

  // 4) Listen for real-time changes from Supabase (remote → local)
  useEffect(() => {
    if (!user?.id) return;

    const channel = supabase
      .channel("profile-changes-offline")
      .on(
        "postgres_changes",
        {
          event: "*",
          schema: "public",
          table: "profiles",
          filter: `user_id=eq.${user.id}`,
        },
        (payload) => {
          console.log("Realtime change to profiles:", payload);
          // Easiest approach: re-fetch from remote
          fetchRemoteProfile();
        }
      )
      .subscribe();

    return () => {
      supabase.removeChannel(channel);
    };
  }, [user, fetchRemoteProfile]);

  // 5) Local → Remote: 
  //    - Update local DB first
  //    - Then push to Supabase
  const updateLocalThenRemote = useCallback(
    async (newDisplayName: string) => {
      if (!user?.id) return;
      setSyncing(true);
      try {
        // Update local
        await updateLocalDisplayName(user.id, newDisplayName);
        const updatedLocal = await getLocalProfile(user.id);
        setLocalProfile(updatedLocal);

        // Push to server (naive last-write-wins)
        const { error } = await supabase
          .from("profiles")
          .update({ display_name: newDisplayName })
          .eq("user_id", user.id);

        if (error) {
          console.log("Error pushing changes to server:", error.message);
        }
      } catch (err) {
        console.log("updateLocalThenRemote error:", err);
      } finally {
        setSyncing(false);
      }
    },
    [user]
  );

  // 6) On user sign-in, do an initial fetch
  useEffect(() => {
    if (user) {
      fetchRemoteProfile();
    }
  }, [user, fetchRemoteProfile]);

  return (
    <OfflineContext.Provider
      value={{
        localProfile,
        syncing,
        fetchRemoteProfile,
        updateLocalThenRemote,
      }}
    >
      {children}
    </OfflineContext.Provider>
  );
}

export function useOffline() {
  return useContext(OfflineContext);
}
```

**Key Points**:

- **localProfile** is the offline copy of the user’s profile.  
- We do a naive **last-write-wins** approach: after editing locally, we overwrite the remote `profiles` row. Then real-time events from Supabase can update local data again if needed.

---

## 10) Code: `app/_layout.tsx` (Single Top-Level Layout)

We wrap **both** contexts: `AuthProvider` (for sign-in/out) and `OfflineProvider` (for local data caching).

```tsx
// app/_layout.tsx
import { Stack } from "expo-router";
import AuthProvider from "../context/auth";
import { OfflineProvider } from "../context/offline"; // offline context
import { StatusBar } from "expo-status-bar";

export default function RootLayout() {
  return (
    <AuthProvider>
      <OfflineProvider>
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
      </OfflineProvider>
    </AuthProvider>
  );
}
```

---

## 11) Code: `app/index.tsx` (Redirect at Launch)

Just a home screen that redirects based on `user` status.

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

## 12) Code: `(auth)` Folder (Sign In & Sign Up)

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
    // user is set by onAuthStateChange
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

## 13) Code: `(protected)` Folder (Protected Routes)

### `(protected)/_layout.tsx`

We add a **Stack** with a visible header and a check for `user`. If the user is missing, we redirect to sign-in.

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
    return null; // Router isn't ready yet
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

### `(protected)/profile.tsx` (Offline-First Display)

This screen displays the user’s profile from **local SQLite** (via `OfflineContext`), plus a button to **“Sync from Server”** to force a remote → local fetch.

```tsx
// app/(protected)/profile.tsx
import { View, Text, Button, ActivityIndicator, StyleSheet } from "react-native";
import { useEffect } from "react";
import { useRouter } from "expo-router";
import { useAuth } from "../../context/auth";
import { useOffline } from "../../context/offline";

export default function ProfileScreen() {
  const { user, signOut } = useAuth();
  const { localProfile, syncing, fetchRemoteProfile } = useOffline();
  const router = useRouter();

  // If not logged in, bounce to sign-in
  useEffect(() => {
    if (!user) {
      router.replace("(auth)/sign-in");
    }
  }, [user]);

  async function handleSignOut() {
    await signOut();
    router.replace("(auth)/sign-in");
  }

  if (!user) {
    return (
      <View style={styles.container}>
        <Text>Please sign in.</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Welcome, {user.email}!</Text>

      {localProfile ? (
        <View>
          <Text style={styles.label}>
            Display Name (Offline-First): {localProfile.display_name || "(none)"}
          </Text>
          <Text style={{ marginBottom: 10 }}>
            Last Updated: {localProfile.updated_at || "N/A"}
          </Text>
        </View>
      ) : (
        <Text>Loading local profile...</Text>
      )}

      {syncing && <ActivityIndicator />}

      <Button
        title="Sync from Server"
        onPress={fetchRemoteProfile}
        disabled={syncing}
      />

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
  title: {
    fontSize: 20,
    marginBottom: 10,
    fontWeight: "bold",
  },
  label: {
    fontSize: 16,
  },
});
```

### `(protected)/edit-profile.tsx` (Offline-First Edit)

When the user edits their display name, we **update local** immediately, then push changes to Supabase.

```tsx
// app/(protected)/edit-profile.tsx
import { View, Text, TextInput, Button, StyleSheet } from "react-native";
import { useState, useEffect } from "react";
import { useRouter } from "expo-router";
import { useAuth } from "../../context/auth";
import { useOffline } from "../../context/offline";

export default function EditProfileScreen() {
  const { user } = useAuth();
  const { localProfile, updateLocalThenRemote, syncing } = useOffline();
  const router = useRouter();

  const [displayName, setDisplayName] = useState("");
  const [loadingLocal, setLoadingLocal] = useState(true);

  // Load local profile data into form
  useEffect(() => {
    if (!localProfile) return;
    setDisplayName(localProfile.display_name || "");
    setLoadingLocal(false);
  }, [localProfile]);

  async function handleSave() {
    if (!user?.id) return;
    await updateLocalThenRemote(displayName);
    router.replace("(protected)/profile");
  }

  if (!user) {
    return (
      <View style={styles.container}>
        <Text>Please sign in.</Text>
      </View>
    );
  }

  if (loadingLocal) {
    return (
      <View style={styles.container}>
        <Text>Loading local profile...</Text>
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
        title={syncing ? "Saving..." : "Save Changes"}
        onPress={handleSave}
        disabled={syncing}
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

## 14) Run & Test

1. **Start** the development server (iOS, Android, or web):
   ```bash
   npx expo start --clear
   ```
2. **Sign Up** with a new email.  
   - Behind the scenes, Supabase calls your DB trigger → creates a `profiles` row.  
3. **Sign In** → The app sets your tokens in SecureStore.  
4. **Offline-First** check:
   - On the **Profile** screen, you’ll see “Display Name (Offline-First)” from **local** DB.  
   - If you disable your network and update your name, it saves locally. Once you reconnect, that local change is pushed to Supabase (naive last-write-wins).  
5. **Realtime** check:
   - If you open the **Supabase Dashboard** or another device and change `display_name` in `profiles`, the real-time subscription will trigger an update in your app, refreshing local DB.  
6. **Sign Out** → Tokens cleared from SecureStore.

---

## 15) Recap & Extended Roadmap

You now have a **complete** Expo + Supabase + offline SQLite setup with:

1. **RLS** to ensure each user can only see/update their own row.  
2. **DB Trigger** to auto-create `profiles` when a user signs up.  
3. **Minimal** token storage (only access & refresh tokens) in SecureStore or localStorage (web).  
4. **Local SQLite** caching (`local_profiles`), so user data remains accessible offline.  
5. **Real-Time** subscription that pushes remote changes → local.  
6. **Naive** conflict strategy (“last-write-wins”) for local → remote.  

### Extended Roadmap

- **Admin Dashboard (Web Interface)**: For user management, content moderation, advanced analytics.  
- **CLI / Scaffolding Tools**: Generate modules or CRUD interfaces automatically, inspired by Bonfire’s or Yii2’s “Builder.”  
- **Advanced Role/Permission System**: Beyond basic RLS, define `admin`, `editor`, `superuser` roles, etc.  
- **Social Features**: E.g., posts, comments, direct messages, file uploads (Supabase Storage), notifications.  
- **Offline Queues & Conflict Resolution**: Instead of naive last-write-wins, store a revision ID or timestamp to resolve merges more robustly.  
- **Production Builds & DevOps**: EAS (Expo Application Services) for iOS/Android, environment variable management, logging, crash reporting, etc.

**Congratulations!** Your ScriptHammer project now includes a fully functioning offline-first approach with secure Supabase authentication, real-time updates, and local caching for resilience in low-connectivity environments. Happy building!
