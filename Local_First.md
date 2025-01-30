Below is a **complete, functional** example that **automatically** handles syncing—no “Sync” button required—and avoids the `SQLite.openDatabase is not a function` error that appears in the **web** environment. We’ll do the following:

1. **Platform Check** to only use `expo-sqlite` on **iOS/Android**. On **web**, we’ll skip local DB calls to prevent crashes.  
2. **NetInfo** integration to automatically detect when the user is online or offline.  
3. **Automatic** local-to-remote push when the user reconnects, so no manual “Sync” button is required.

> **Important**  
> - `expo-sqlite` **does not** work in a standard web browser environment. It’s for native iOS/Android. If you need offline DB on web, consider [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API) with a library like [Dexie](https://dexie.org/).  
> - This tutorial shows how to keep local data in sync **on native devices** without forcing the user to tap a “Sync” button.  
> - If you try this in the Expo Web (browser) preview, you’ll see `SQLite.openDatabase is not a function`. The code below gracefully **skips** local DB usage on web to avoid the error.

---

# ScriptHammer - Auto Sync + Local-First (No Manual “Sync” Button)

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
7. [Code: `localdb.ts` (Platform-Aware SQLite Helpers)](#7-code-localdbts-platform-aware-sqlite-helpers)  
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

Now you have a blank Expo Router project (no leftover `NavigationContainer` calls).

---

## 2) Install Dependencies

```bash
npm install @supabase/supabase-js
npx expo install expo-secure-store
npm install dotenv
npx expo install expo-sqlite
npm install @react-native-community/netinfo
```

**Why**:

1. `@supabase/supabase-js`: Supabase client.  
2. `expo-secure-store`: Secure session storage on native.  
3. `dotenv`: so we can load `.env.local` variables.  
4. `expo-sqlite`: local DB on iOS/Android.  
5. `@react-native-community/netinfo`: detects when user toggles offline/online.

---

## 3) Create & Configure `.env.local`

```bash
EXPO_PUBLIC_SUPABASE_URL=https://YOUR-PROJECT.supabase.co
EXPO_PUBLIC_SUPABASE_ANON_KEY=YOUR-ANON-KEY
```

Adjust to your real credentials. Add `.env.local` to `.gitignore` if you don’t want them committed.

Because we prefixed with `EXPO_PUBLIC_`, these are automatically injected as `process.env.EXPO_PUBLIC_...` by Expo.

---

## 4) File & Folder Structure

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

1. **Enable RLS** on `profiles`.  
2. Add a **SELECT** policy:
   ```sql
   create policy "Select own profile"
   on public.profiles
   for select
   using ( auth.uid() = user_id );
   ```
3. Add an **UPDATE** policy:
   ```sql
   create policy "Update own profile"
   on public.profiles
   for update
   using ( auth.uid() = user_id );
   ```

### Create a DB Trigger for Auto-Inserting `profiles`

So each new user automatically gets a `profiles` row:

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

In **Table Editor** → **profiles**, enable “Realtime” so changes can push events to your client.

---

## 6) Code: `supabaseClient.ts` (Connecting to Supabase)

```ts
// supabaseClient.ts
import { createClient } from "@supabase/supabase-js";

const SUPABASE_URL = process.env.EXPO_PUBLIC_SUPABASE_URL!;
const SUPABASE_ANON_KEY = process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY!;

export const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
```

---

## 7) Code: `localdb.ts` (Platform-Aware SQLite Helpers)

We must **skip** calling `SQLite.openDatabase()` on web to avoid the `openDatabase is not a function` error. We’ll check `Platform.OS`:

```ts
// localdb.ts
import { Platform } from "react-native";
import * as SQLite from "expo-sqlite";

let db: SQLite.WebSQLDatabase | null = null;

// Only open the database on native (iOS, Android). Skip on web.
if (Platform.OS !== "web") {
  db = SQLite.openDatabase("ScriptHammer.db");
}

/**
 * Setup local_profiles if on native. Otherwise no-op on web.
 */
export async function setupLocalDatabase() {
  if (!db) return;
  return new Promise<void>((resolve, reject) => {
    db?.transaction((tx) => {
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
 * Upsert (insert or update) a local profile if on native.
 * On web, it does nothing (to avoid errors).
 */
export async function upsertLocalProfile(
  userId: string,
  displayName: string,
  updatedAt: string
) {
  if (!db) return; // skip on web
  return new Promise<void>((resolve, reject) => {
    db?.transaction((tx) => {
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
 * Fetch a local profile by user_id, or null if on web or not found.
 */
export async function getLocalProfile(userId: string) {
  if (!db) return null; // skip on web
  return new Promise<any>((resolve, reject) => {
    db?.transaction((tx) => {
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
 * Update local display name + updated_at, skip on web
 */
export async function updateLocalDisplayName(userId: string, displayName: string) {
  if (!db) return;
  const now = new Date().toISOString();
  return new Promise<void>((resolve, reject) => {
    db?.transaction((tx) => {
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
```

**Result**:  
- On **native** (iOS/Android), we truly store data in `local_profiles`.  
- On **web**, these calls become **no-ops** or return `null` so you don’t crash.

---

## 8) Code: `context/auth.tsx` (Auth Context)

Handles sign-up/sign-in with minimal token storage in SecureStore or localStorage. Same as previous ScriptHammer examples:

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

// Cross-platform local store
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

  // Attempt token restore
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

We’ll detect **online/offline** using NetInfo, store local changes offline in `localdb`, and automatically push them to Supabase the moment the device is connected. **No user action required**.

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
} from "../localdb";
import { useAuth } from "./auth";

interface OfflineContextProps {
  localProfile: any; // the user's offline profile
  isOnline: boolean;
  syncing: boolean;
  updateLocalAndQueueSync: (newDisplayName: string) => Promise<void>;
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

  // 1) Setup local DB on mount (native only)
  useEffect(() => {
    (async () => {
      try {
        await setupLocalDatabase();
      } catch (err) {
        console.log("Error setting up local DB:", err);
      }
    })();
  }, []);

  // 2) Listen for NetInfo changes (online/offline)
  useEffect(() => {
    const unsubscribe = NetInfo.addEventListener((state) => {
      const currentlyOnline = !!state.isConnected;
      if (currentlyOnline !== isOnline) {
        setIsOnline(currentlyOnline);
      }
    });
    return () => {
      unsubscribe();
    };
  }, [isOnline]);

  // 3) Whenever user logs in or changes, load local data
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

  // 4) Real-time subscription from Supabase → local
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
          console.log("Realtime event from Supabase:", payload);
          // Just re-fetch from server if online
          if (isOnline) {
            await syncRemoteToLocal();
          }
        }
      )
      .subscribe();

    return () => {
      supabase.removeChannel(channel);
    };
  }, [user, isOnline]);

  // 5) On user sign-in and net connected, do initial sync
  useEffect(() => {
    if (user && isOnline) {
      syncRemoteToLocal();
    }
  }, [user, isOnline]);

  /**
   * Pull from Supabase → local
   */
  const syncRemoteToLocal = useCallback(async () => {
    if (!user?.id || !isOnline) return;
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
        const updatedAt = new Date().toISOString();
        await upsertLocalProfile(data.user_id, data.display_name ?? "", updatedAt);
        const localData = await getLocalProfile(user.id);
        setLocalProfile(localData);
      }
    } catch (err) {
      console.log("syncRemoteToLocal error:", err);
    } finally {
      setSyncing(false);
    }
  }, [user, isOnline]);

  /**
   * For local edits: store in local DB, then if online push to server
   */
  const updateLocalAndQueueSync = useCallback(
    async (newDisplayName: string) => {
      if (!user?.id) return;
      try {
        // 1) Update local DB immediately
        await updateLocalDisplayName(user.id, newDisplayName);
        const updatedLocal = await getLocalProfile(user.id);
        setLocalProfile(updatedLocal);

        // 2) If we are online, push to Supabase right away
        if (isOnline) {
          setSyncing(true);
          const { error } = await supabase
            .from("profiles")
            .update({ display_name: newDisplayName })
            .eq("user_id", user.id);

          if (error) {
            console.log("Error pushing changes to Supabase:", error.message);
          }
          setSyncing(false);
        } else {
          // If offline, it stays in local DB. 
          // Next time we detect isOnline === true, we re-fetch from server
          // or you could implement a queue that tries to push local changes automatically.
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

### Explanation

- **NetInfo**: We store `isOnline` in state.  
- **syncRemoteToLocal**: If the device becomes online, we fetch the user’s row from Supabase → store in local DB.  
- **updateLocalAndQueueSync**: Whenever the user changes `display_name`, we update local DB. If we’re online, push immediately to remote. If offline, it stays local until next time we check the server or implement an automatic push queue.

---

## 10) Code: `app/_layout.tsx` (Single Top-Level Layout)

We wrap **AuthProvider** and **OfflineProvider** so every screen can access them.

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

Simple home screen that redirects based on whether the user is logged in.

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

  // If no user, redirect
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

We **automatically** sync from remote to local when the device is online. No manual button needed. We just display `localProfile` and `isOnline` status.

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
      <Text style={styles.heading}>
        {isOnline ? "Online" : "Offline"} - Welcome, {user.email}!
      </Text>

      {localProfile ? (
        <View>
          <Text style={styles.label}>
            Display Name: {localProfile.display_name || "(none)"}
          </Text>
          <Text>Last Updated: {localProfile.updated_at || "(unknown)"}</Text>
        </View>
      ) : (
        <Text>Loading local profile...</Text>
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
  container: { paddingHorizontal: 20, paddingTop: 40 },
  heading: { fontSize: 18, fontWeight: "bold", marginBottom: 10 },
  label: { fontSize: 16 },
});
```

### `(protected)/edit-profile.tsx` (Offline-First Edit)

When user updates the display name, we update local DB. If `isOnline`, push to Supabase immediately. Otherwise, next time we come online, we fetch or push changes.

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
        <Text>No user found.</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <Text style={styles.label}>Display Name:</Text>
      <TextInput
        style={styles.input}
        placeholder="Enter a name"
        value={displayName}
        onChangeText={setDisplayName}
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

- On **iOS/Android** device or emulator, you’ll have a **local** SQLite table.  
- On **web**, the local DB calls become **no-ops** (no offline storage).  
- **Sign Up** or **Sign In**.  
- **Go Offline** by disabling Wi-Fi or airplane mode:
  - If you edit your display name, it updates in your local DB only.  
- **Go Online** again:
  - The next time the offline context triggers a sync, it pushes your local changes to Supabase (or you can pull from server).  
- Real-time events from Supabase also come in automatically if you’re online.

---

## 15) Recap & Next Steps

You now have:

1. **Platform-Aware** local DB usage (only iOS/Android).  
2. **Automatic** offline editing (no extra button).  
3. **NetInfo** to detect connectivity and push changes once online.  
4. **Minimal** token storage, real-time subscription for changes from server.  

**Extended Roadmap**:

- For **web** offline: use a real IndexedDB-based approach.  
- For more robust conflict resolution: track versions/timestamps, or queue pending updates.  
- Add an **Admin Dashboard** (web), advanced role-based policies, or push notifications.

**Congratulations**—you now have a working, **offline-first** ScriptHammer setup that **doesn’t** rely on a manual “Sync” button, gracefully handles web vs. native differences, and ensures your user’s data is seamlessly updated whether they’re connected or not!
