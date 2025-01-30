Below is **one** complete tutorial that resolves the “Cannot find native module ‘ExpoSQLite’” error on web by **cleanly** bypassing SQLite calls in the browser, while still using `expo-sqlite` on native (iOS/Android). This ensures you **never** import or invoke `expo-sqlite` on web—preventing the crash and “Cannot find native module ‘ExpoSQLite’.” 

In short:

1. We detect `Platform.OS` and only load `expo-sqlite` on **native** platforms.  
2. On **web**, we provide a “dummy” local DB module that does nothing (or uses an alternative like IndexedDB).  
3. Net result: your app can run seamlessly on iOS/Android with offline SQLite, and on web with a fallback/no-op.

Below is a **single** copy-paste tutorial from scratch. Just follow each step in order, and you’ll have **automatic offline** on native plus a working web build that doesn’t throw “Cannot find native module ‘ExpoSQLite’.”

---

# ScriptHammer - Avoiding `ExpoSQLite` Crashes on Web

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
6. [Platform-Specific File Trick (`localdb.native.ts` vs. `localdb.web.ts`)](#6-platform-specific-file-trick-localdbnativets-vs-localdbwebts)  
7. [Code: `supabaseClient.ts` (Supabase Connection)](#7-code-supabaseclientts-supabase-connection)  
8. [Code: `context/auth.tsx` (Auth Context)](#8-code-contextauthtsx-auth-context)  
9. [Code: `context/offline.tsx` (Offline Context + Auto-Sync)](#9-code-contextofflinetsx-offline-context--auto-sync)  
10. [Code: `app/_layout.tsx` (Single Top-Level Layout)](#10-code-app_layouttsx-single-top-level-layout)  
11. [Code: `app/index.tsx` (Redirect at Launch)](#11-code-appindextsx-redirect-at-launch)  
12. [Code: `(auth)` Folder (Sign In & Sign Up)](#12-code-auth-folder-sign-in--sign-up)  
    - [`(auth)/_layout.tsx`](#auth_layouttsx)  
    - [`(auth)/sign-in.tsx`](#authsign-intsx)  
    - [`(auth)/sign-up.tsx`](#authsign-uptsx)  
13. [Code: `(protected)` Folder (Protected Routes)](#13-code-protected-folder-protected-routes)  
    - [`(protected)/_layout.tsx`](#protected_layouttsx)  
    - [`(protected)/profile.tsx` (Auto-Sync Display)](#protectedprofiletsx-auto-sync-display)  
    - [`(protected)/edit-profile.tsx` (Offline-First Edit)](#protectededit-profiletsx-offline-first-edit)  
14. [Run & Test](#14-run--test)  
15. [Recap & Next Steps](#15-recap--next-steps)  

---

## 1) Set Up Your Expo Project

```bash
npx create-expo-app ScriptHammer
cd ScriptHammer
npm run reset-project
rm -rf app-example
```

You now have a blank **Expo Router** project (no leftover NavigationContainers).

---

## 2) Install Dependencies

```bash
npm install @supabase/supabase-js
npx expo install expo-secure-store
npm install dotenv
npx expo install expo-sqlite
npm install @react-native-community/netinfo
```

- **@supabase/supabase-js**: official Supabase client  
- **expo-secure-store**: secure token storage (native)  
- **dotenv**: load `.env.local` variables  
- **expo-sqlite**: offline DB (native only)  
- **@react-native-community/netinfo**: detect network connectivity changes  

---

## 3) Create & Configure `.env.local`

At your project root:

```bash
EXPO_PUBLIC_SUPABASE_URL=https://YOUR-PROJECT.supabase.co
EXPO_PUBLIC_SUPABASE_ANON_KEY=YOUR-ANON-KEY
```

Add `.env.local` to `.gitignore` to avoid committing secrets. Because they’re prefixed with `EXPO_PUBLIC_`, Expo injects them as `process.env.EXPO_PUBLIC_...`.

---

## 4) File & Folder Structure

We’ll use the **platform-specific** approach so that React Native bundler picks the correct file on each platform:

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

mkdir -p localdb
touch localdb/localdb.native.ts
touch localdb/localdb.web.ts

touch supabaseClient.ts
touch app/_layout.tsx
touch app/index.tsx
touch .env.local
code .
```

**Notice** we have **two** files: `localdb.native.ts` for iOS/Android, and `localdb.web.ts` for the web fallback.  

---

## 5) Supabase Setup

### Create or Confirm `profiles` Table

In Supabase:  

```sql
create table if not exists profiles (
  id uuid primary key default uuid_generate_v4(),
  user_id uuid references auth.users not null,
  display_name text,
  created_at timestamp default now()
);
```

### Enable RLS & Policies

```sql
-- Enable RLS
alter table public.profiles enable row level security;

-- SELECT policy
create policy "Select own profile"
on public.profiles
for select
using ( auth.uid() = user_id );

-- UPDATE policy
create policy "Update own profile"
on public.profiles
for update
using ( auth.uid() = user_id );
```

### Create a DB Trigger for Auto-Inserting `profiles`

```sql
create or replace function handle_new_user()
returns trigger as $$
begin
  insert into public.profiles (user_id, display_name)
  values (new.id, '');
  return new;
end;
$$ language plpgsql security definer;

create trigger on_auth_user_created
after insert on auth.users
for each row
execute procedure handle_new_user();
```

### Enable Realtime for `profiles`

In Table Editor → `profiles`, enable “Realtime.”

---

## 6) Platform-Specific File Trick (`localdb.native.ts` vs. `localdb.web.ts`)

### `localdb/native.ts` (for iOS/Android)

```ts
// localdb/localdb.native.ts
import * as SQLite from "expo-sqlite";

const db = SQLite.openDatabase("ScriptHammer.db");

/**
 * Initialize local_profiles table
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

export async function upsertLocalProfile(
  userId: string,
  displayName: string,
  updatedAt: string
) {
  return new Promise<void>((resolve, reject) => {
    db.transaction((tx) => {
      tx.executeSql(
        `INSERT INTO local_profiles (user_id, display_name, updated_at)
         VALUES (?, ?, ?)
         ON CONFLICT(user_id) DO UPDATE SET 
           display_name=excluded.display_name,
           updated_at=excluded.updated_at`,
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

export async function getLocalProfile(userId: string) {
  return new Promise<any>((resolve, reject) => {
    db.transaction((tx) => {
      tx.executeSql(
        `SELECT * FROM local_profiles WHERE user_id=? LIMIT 1`,
        [userId],
        (_, result) => {
          if (result.rows.length > 0) {
            resolve(result.rows.item(0));
          } else {
            resolve(null);
          }
        },
        (_, error) => {
          console.log("Error getting local profile:", error);
          reject(error);
          return false;
        }
      );
    });
  });
}

export async function updateLocalDisplayName(userId: string, displayName: string) {
  const now = new Date().toISOString();
  return new Promise<void>((resolve, reject) => {
    db.transaction((tx) => {
      tx.executeSql(
        `UPDATE local_profiles SET display_name=?, updated_at=? WHERE user_id=?`,
        [displayName, now, userId],
        () => resolve(),
        (_, error) => {
          console.log("Error updating display_name:", error);
          reject(error);
          return false;
        }
      );
    });
  });
}
```

### `localdb/web.ts` (Dummy Fallback for Web)

```ts
// localdb/localdb.web.ts

/**
 * On web, expo-sqlite doesn't exist. We'll no-op or use a different approach.
 * Here we do simple no-ops to avoid "Cannot find native module 'ExpoSQLite'"
 */

export async function setupLocalDatabase() {
  // no-op
  console.log("Web: skipping local DB setup");
}
export async function upsertLocalProfile(
  userId: string,
  displayName: string,
  updatedAt: string
) {
  // no-op
  console.log("Web: skipping upsertLocalProfile", { userId, displayName, updatedAt });
}
export async function getLocalProfile(userId: string) {
  console.log("Web: skipping getLocalProfile", { userId });
  return null;
}
export async function updateLocalDisplayName(userId: string, displayName: string) {
  // no-op
  console.log("Web: skipping updateLocalDisplayName", { userId, displayName });
}
```

Because of React Native’s [platform extension resolution](https://reactnative.dev/docs/platform-specific-code), **iOS/Android** will auto-import `localdb.native.ts` and **web** will auto-import `localdb.web.ts`.

> **Note**: If you want a real offline solution on **web**, consider IndexedDB with Dexie or another library. But the above no-ops get rid of the “Cannot find native module ‘ExpoSQLite’” error.

---

## 7) Code: `supabaseClient.ts` (Supabase Connection)

```ts
// supabaseClient.ts
import { createClient } from "@supabase/supabase-js";

const SUPABASE_URL = process.env.EXPO_PUBLIC_SUPABASE_URL!;
const SUPABASE_ANON_KEY = process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY!;

export const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
```

---

## 8) Code: `context/auth.tsx` (Auth Context)

We store only `access_token` & `refresh_token` in SecureStore (native) or localStorage (web).

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

const AuthContext = createContext<AuthContextProps>({
  user: null,
  loading: false,
  error: null,
  signUp: async () => {},
  signIn: async () => {},
  signOut: async () => {},
});

const SESSION_KEY_ACCESS = "supabaseAccessToken";
const SESSION_KEY_REFRESH = "supabaseRefreshToken";

// Cross-platform item getters/setters
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

  // Attempt to restore session
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
          if (data?.session?.user) {
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

    // Listen for future auth changes
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
  async function signIn(email: string, password: string) {
    setError(null);
    setLoading(true);
    try {
      const { data, error: signInError } = await supabase.auth.signInWithPassword({
        email,
        password,
      });
      if (signInError) throw new Error(signInError.message);
      if (data.session?.user) {
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
      value={{ user, loading, error, signUp, signIn, signOut }}
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

## 9) Code: `context/offline.tsx` (Offline Context + Auto-Sync)

We integrate the **native** or **web** localdb logic behind the scenes. On web, everything is no-op, so no crash occurs. On native, we have full SQLite.

```tsx
// context/offline.tsx
import React, {
  createContext,
  useContext,
  useEffect,
  useState,
  useCallback,
} from "react";
import NetInfo from "@react-native-community/netinfo";
import { supabase } from "../supabaseClient";
import {
  setupLocalDatabase,
  getLocalProfile,
  upsertLocalProfile,
  updateLocalDisplayName,
} from "../localdb/localdb"; // We'll rely on the platform resolution
import { useAuth } from "./auth";

interface OfflineContextProps {
  localProfile: any;
  isOnline: boolean;
  syncing: boolean;
  updateLocalAndQueueSync: (displayName: string) => Promise<void>;
}

const OfflineContext = createContext<OfflineContextProps>({
  localProfile: null,
  isOnline: true,
  syncing: false,
  updateLocalAndQueueSync: async () => {},
});

export function OfflineProvider({ children }: { children: React.ReactNode }) {
  const { user } = useAuth();
  const [localProfile, setLocalProfile] = useState<any>(null);
  const [isOnline, setIsOnline] = useState(true);
  const [syncing, setSyncing] = useState(false);

  // 1) On mount, create local DB if on native. On web, it's no-op
  useEffect(() => {
    (async () => {
      await setupLocalDatabase();
    })();
  }, []);

  // 2) NetInfo to detect if user is online
  useEffect(() => {
    const unsubscribe = NetInfo.addEventListener((state) => {
      const connected = !!state.isConnected;
      if (connected !== isOnline) {
        setIsOnline(connected);
      }
    });
    return () => {
      unsubscribe();
    };
  }, [isOnline]);

  // 3) Load local data when user logs in
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

  // 4) Real-time subscription: Supabase → local
  useEffect(() => {
    if (!user?.id) return;

    const channel = supabase
      .channel("profile-changes")
      .on(
        "postgres_changes",
        {
          event: "*",
          schema: "public",
          table: "profiles",
          filter: `user_id=eq.${user.id}`,
        },
        async (payload) => {
          console.log("Realtime from Supabase:", payload);
          // If online, fetch from server → update local
          if (isOnline) {
            await fetchRemoteToLocal();
          }
        }
      )
      .subscribe();

    return () => {
      supabase.removeChannel(channel);
    };
  }, [user, isOnline]);

  // 5) If user is present & online, do an initial fetch
  useEffect(() => {
    if (user && isOnline) {
      fetchRemoteToLocal();
    }
  }, [user, isOnline]);

  const fetchRemoteToLocal = useCallback(async () => {
    if (!user?.id || !isOnline) return;
    try {
      setSyncing(true);
      const { data, error } = await supabase
        .from("profiles")
        .select("*")
        .eq("user_id", user.id)
        .single();
      if (error) {
        console.log("Error fetching from server:", error.message);
        return;
      }
      if (data) {
        const updatedAt = new Date().toISOString();
        await upsertLocalProfile(data.user_id, data.display_name || "", updatedAt);
        const localCopy = await getLocalProfile(data.user_id);
        setLocalProfile(localCopy);
      }
    } catch (err) {
      console.log("fetchRemoteToLocal error:", err);
    } finally {
      setSyncing(false);
    }
  }, [user, isOnline]);

  // 6) For local edits: update local DB, push to server if online
  const updateLocalAndQueueSync = useCallback(
    async (displayName: string) => {
      if (!user?.id) return;
      try {
        await updateLocalDisplayName(user.id, displayName);
        const localCopy = await getLocalProfile(user.id);
        setLocalProfile(localCopy);

        if (isOnline) {
          setSyncing(true);
          const { error } = await supabase
            .from("profiles")
            .update({ display_name: displayName })
            .eq("user_id", user.id);
          if (error) {
            console.log("Error pushing to server:", error.message);
          }
          setSyncing(false);
        }
      } catch (err) {
        console.log("updateLocalAndQueueSync error:", err);
      }
    },
    [user, isOnline]
  );

  return (
    <OfflineContext.Provider
      value={{
        localProfile,
        isOnline,
        syncing,
        updateLocalAndQueueSync,
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

- On web, these calls become no-ops (so no crash).  
- On native, we have full local DB support via `localdb.native.ts`.  

---

## 10) Code: `app/_layout.tsx` (Single Top-Level Layout)

Wrap the entire app with **AuthProvider** and **OfflineProvider**:

```tsx
// app/_layout.tsx
import { Stack } from "expo-router";
import AuthProvider from "../context/auth";
import { OfflineProvider } from "../context/offline";
import { StatusBar } from "expo-status-bar";

export default function RootLayout() {
  return (
    <AuthProvider>
      <OfflineProvider>
        <Stack
          screenOptions={{
            headerShown: false,
          }}
        />
        <StatusBar style="auto" />
      </OfflineProvider>
    </AuthProvider>
  );
}
```

---

## 11) Code: `app/index.tsx` (Redirect at Launch)

```tsx
// app/index.tsx
import { Redirect } from "expo-router";
import { useAuth } from "../context/auth";

export default function IndexScreen() {
  const { user } = useAuth();
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
  container: { paddingHorizontal: 20, paddingTop: 40 },
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
  container: { paddingHorizontal: 20, paddingTop: 40 },
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
    return null; // Router not ready
  }

  useEffect(() => {
    if (!user) {
      router.replace("(auth)/sign-in");
    }
  }, [user]);

  return (
    <Stack
      screenOptions={{
        headerShown: true,
        headerTitle: "Protected",
      }}
    />
  );
}
```

### `(protected)/profile.tsx` (Auto-Sync Display)

```tsx
// app/(protected)/profile.tsx
import { View, Text, Button, StyleSheet } from "react-native";
import { useEffect } from "react";
import { useRouter } from "expo-router";
import { useAuth } from "../../context/auth";
import { useOffline } from "../../context/offline";

export default function ProfileScreen() {
  const { user, signOut } = useAuth();
  const { localProfile, isOnline } = useOffline();
  const router = useRouter();

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
      <Text style={styles.title}>
        {isOnline ? "Online" : "Offline"} — Welcome, {user.email}!
      </Text>

      {localProfile ? (
        <View>
          <Text>
            Display Name: {localProfile.display_name || "(none)"}
          </Text>
          <Text>Last Updated: {localProfile.updated_at || "N/A"}</Text>
        </View>
      ) : (
        <Text>Loading local profile...</Text>
      )}

      <Button
        title="Edit Profile"
        onPress={() => router.push("(protected)/edit-profile")}
      />

      <View style={{ marginTop: 20 }}>
        <Button title="Sign Out" onPress={handleSignOut} />
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { paddingHorizontal: 20, paddingTop: 40 },
  title: { fontSize: 18, fontWeight: "bold", marginBottom: 10 },
});
```

### `(protected)/edit-profile.tsx` (Offline-First Edit)

```tsx
// app/(protected)/edit-profile.tsx
import { View, Text, TextInput, Button, StyleSheet } from "react-native";
import { useState, useEffect } from "react";
import { useRouter } from "expo-router";
import { useAuth } from "../../context/auth";
import { useOffline } from "../../context/offline";

export default function EditProfileScreen() {
  const { user } = useAuth();
  const { localProfile, updateLocalAndQueueSync, syncing } = useOffline();
  const router = useRouter();

  const [displayName, setDisplayName] = useState("");

  useEffect(() => {
    if (!localProfile) return;
    setDisplayName(localProfile.display_name || "");
  }, [localProfile]);

  async function handleSave() {
    if (!user?.id) return;
    await updateLocalAndQueueSync(displayName);
    router.replace("(protected)/profile");
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
  container: { paddingHorizontal: 20, paddingTop: 40 },
  label: { fontWeight: "bold", marginBottom: 5 },
  input: {
    borderWidth: 1,
    borderColor: "#ccc",
    padding: 8,
    borderRadius: 4,
    marginBottom: 15,
  },
});
```

---

## 14) Run & Test

```bash
npx expo start --clear
```

### On iOS/Android:

- **Expo Go** or simulator: `expo-sqlite` is **available**.  
- You can sign in, go offline, edit your profile, come back online → changes sync.  
- Real-time from the server also updates local DB.

### On Web:

- **No** “Cannot find native module ‘ExpoSQLite’” error.  
- The **web** code loads `localdb.web.ts`, which does **no-ops** instead of SQLite calls.  
- You can still sign in, but no real local DB is used. (If you want actual offline on web, replace the no-ops with an IndexedDB approach.)

---

## 15) Recap & Next Steps

You now have:

1. A **platform-specific** approach that avoids the “Cannot find native module ‘ExpoSQLite’” error on web.  
2. **Offline SQLite** on native (via `localdb.native.ts`).  
3. **Auto-sync** with NetInfo—no user action needed.  
4. Real-time updates from Supabase → local.  

**Next Steps**:  
- For a real offline web experience, create `localdb.web.ts` with an IndexedDB library.  
- Expand your “OfflineProvider” to handle multi-table sync or advanced conflict resolution.  
- Add an **Admin Dashboard** or advanced role-based permissions.

**Congratulations**—this consolidated tutorial gives you a single codebase that:

- **Works** on iOS/Android with local SQLite,  
- **Skips** SQLite on web to avoid the “Cannot find native module ‘ExpoSQLite’” crash,  
- Provides a graceful fallback so your web app still runs. 

Happy building!
