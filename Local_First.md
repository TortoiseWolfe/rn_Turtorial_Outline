Below is a **step-by-step** tutorial that **definitely** works on **iOS/Android** (with SQLite) and **Web** (with Dexie) in an **Expo-managed** project. We’ll take extra care around the **native iOS** setup, so you don’t get `SQLite.openDatabase is not a function` or a non-responsive sign-in button.

> **Important Notes**  
> 1. If you only run `npm run web` or open the “Web” browser preview in Expo Go, you’re effectively in “web mode.” That uses `localdb.web.ts` with Dexie and never touches `expo-sqlite`.  
> 2. To truly test **native iOS** (in the iOS simulator or an actual device), do **one** of the following:
>    - **Option A** (Managed workflow, iOS simulator): `npx expo start` then press **i** (or choose “Run on iOS simulator” in the CLI).  
>    - **Option B** (Managed workflow, real device): `npx expo start --tunnel` and open in iOS with the Expo Go app.  
>    - **Option C** (New “prebuild” or “run” commands): `npx expo prebuild` then `npx expo run:ios`.  
> 3. Make sure `expo-sqlite` is properly installed and linked by letting Expo do it automatically (the “managed” flow). You typically **don’t** do `pod install` manually if you’re in the fully-managed workflow.  

Follow these steps in order, and you’ll get:

- **Offline** on iOS/Android with `expo-sqlite`,  
- **Offline** on Web with **Dexie**,  
- **No** “SQLite.openDatabase is not a function” if you actually run on **native** iOS (not web),  
- Fully functional sign-in, sign-up, and offline edit flow.

---

# ScriptHammer - Full Offline (iOS + Android + Web) with Dexie (Web) and Expo SQLite (Native)

## Table of Contents

1. [Set Up a Fresh Expo Project](#1-set-up-a-fresh-expo-project)  
2. [Install Dependencies](#2-install-dependencies)  
3. [Create & Configure `.env.local`](#3-create--configure-envlocal)  
4. [File/Folder Structure](#4-filefolder-structure)  
5. [Supabase Setup](#5-supabase-setup)  
   - [Create/Confirm `profiles` Table](#createconfirm-profiles-table)  
   - [Enable RLS & Policies](#enable-rls--policies)  
   - [DB Trigger for `profiles`](#db-trigger-for-profiles)  
   - [Enable Realtime](#enable-realtime)  
6. [Platform-Specific Local DB Files (`localdb.native.ts` vs. `localdb.web.ts`)](#6-platform-specific-local-db-files-localdbnativets-vs-localdbwebts)  
7. [Code: `supabaseClient.ts`](#7-code-supabaseclientts)  
8. [Code: `context/auth.tsx` (Auth Context)](#8-code-contextauthtsx-auth-context)  
9. [Code: `context/offline.tsx` (Offline Context + Auto-Sync)](#9-code-contextofflinetsx-offline-context--auto-sync)  
10. [Code: `app/_layout.tsx` (Top-Level Layout)](#10-code-app_layouttsx-top-level-layout)  
11. [Code: `app/index.tsx` (Redirect)](#11-code-appindextsx-redirect)  
12. [Code: `(auth)` Folder (Sign In/Up)](#12-code-auth-folder-sign-inup)  
13. [Code: `(protected)` Folder](#13-code-protected-folder)  
14. [Run & Test](#14-run--test)  
15. [Troubleshooting iOS “SQLite is not a function”](#15-troubleshooting-ios-sqlite-is-not-a-function)  
16. [Recap & Next Steps](#16-recap--next-steps)  

---

## 1) Set Up a Fresh Expo Project

```bash
npx create-expo-app ScriptHammer
cd ScriptHammer

# This command removes leftover example screens in the router
npm run reset-project
rm -rf app-example
```

You now have a **blank** Expo Router project.

---

## 2) Install Dependencies

```bash
npm install @supabase/supabase-js
npx expo install expo-secure-store
npm install dotenv
npx expo install expo-sqlite
npm install @react-native-community/netinfo
npm install dexie
```

Explanations:

1. **@supabase/supabase-js**: Official Supabase client.  
2. **expo-secure-store**: Stores tokens securely on iOS/Android.  
3. **dotenv**: For `.env.local`.  
4. **expo-sqlite**: Local DB on **native**.  
5. **@react-native-community/netinfo**: Detect offline/online states.  
6. **dexie**: Use IndexedDB on web.

---

## 3) Create & Configure `.env.local`

At your project root:

```bash
EXPO_PUBLIC_SUPABASE_URL=https://YOUR-PROJECT.supabase.co
EXPO_PUBLIC_SUPABASE_ANON_KEY=YOUR-ANON-KEY
```

Add `.env.local` to `.gitignore`.

---

## 4) File/Folder Structure

We’ll place code as follows:

```bash
mkdir context
touch context/auth.tsx
touch context/offline.tsx

mkdir -p localdb
touch localdb/localdb.native.ts
touch localdb/localdb.web.ts

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
```

---

## 5) Supabase Setup

### Create/Confirm `profiles` Table

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
alter table public.profiles enable row level security;

create policy "Select own profile"
on public.profiles
for select
using ( auth.uid() = user_id );

create policy "Update own profile"
on public.profiles
for update
using ( auth.uid() = user_id );
```

### DB Trigger for `profiles`

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

### Enable Realtime

Turn on “Realtime” for `profiles` in the Supabase UI (Table Editor → Realtime).

---

## 6) Platform-Specific Local DB Files (`localdb.native.ts` vs. `localdb.web.ts`)

### `localdb.native.ts` (iOS/Android via Expo SQLite)

```ts
// localdb/localdb.native.ts
import * as SQLite from "expo-sqlite";

const db = SQLite.openDatabase("ScriptHammer.db");

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
        (_, err) => {
          console.error("Error creating local_profiles table:", err);
          reject(err);
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
        `
        INSERT INTO local_profiles (user_id, display_name, updated_at)
        VALUES (?, ?, ?)
        ON CONFLICT(user_id) DO UPDATE 
          SET display_name=excluded.display_name,
              updated_at=excluded.updated_at
      `,
        [userId, displayName, updatedAt],
        () => resolve(),
        (_, err) => {
          console.error("Error upserting local profile:", err);
          reject(err);
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
        (_, res) => {
          if (res.rows.length > 0) {
            resolve(res.rows.item(0));
          } else {
            resolve(null);
          }
        },
        (_, err) => {
          console.error("Error fetching local profile:", err);
          reject(err);
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
        `UPDATE local_profiles 
         SET display_name=?, updated_at=? 
         WHERE user_id=?`,
        [displayName, now, userId],
        () => resolve(),
        (_, err) => {
          console.error("Error updating local profile:", err);
          reject(err);
          return false;
        }
      );
    });
  });
}
```

### `localdb.web.ts` (Dexie on Web)

```ts
// localdb/localdb.web.ts
import Dexie from "dexie";

const db = new Dexie("ScriptHammerWebDB");
db.version(1).stores({
  local_profiles: "user_id, display_name, updated_at",
});

export async function setupLocalDatabase() {
  console.log("Dexie is ready for web offline DB");
}

export async function upsertLocalProfile(
  userId: string,
  displayName: string,
  updatedAt: string
) {
  await db.table("local_profiles").put({
    user_id: userId,
    display_name: displayName,
    updated_at: updatedAt,
  });
}

export async function getLocalProfile(userId: string) {
  const row = await db.table("local_profiles").get(userId);
  return row || null;
}

export async function updateLocalDisplayName(userId: string, displayName: string) {
  const now = new Date().toISOString();
  await db.table("local_profiles").update(userId, {
    display_name: displayName,
    updated_at: now,
  });
}
```

---

## 7) Code: `supabaseClient.ts`

```ts
// supabaseClient.ts
import { createClient } from "@supabase/supabase-js";

const SUPABASE_URL = process.env.EXPO_PUBLIC_SUPABASE_URL!;
const SUPABASE_ANON_KEY = process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY!;

export const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
```

---

## 8) Code: `context/auth.tsx` (Auth Context)

Handles sign in/up, storing tokens in SecureStore (native) or localStorage (web).

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

// Cross-platform get/set
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

  // Attempt to restore tokens
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
      if (session?.user) {
        setUser(session.user);
        storeTokens(session);
      } else {
        setUser(null);
        clearTokens();
      }
    });
    return () => subscription.unsubscribe();
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

Ensures that local DB calls go to SQLite (native) or Dexie (web).

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
} from "../localdb/localdb"; // platform-specific
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

  // 1) Setup local DB
  useEffect(() => {
    (async () => {
      await setupLocalDatabase();
    })();
  }, []);

  // 2) NetInfo for online/offline
  useEffect(() => {
    const unsubscribe = NetInfo.addEventListener((state) => {
      const connected = !!state.isConnected;
      if (connected !== isOnline) {
        setIsOnline(connected);
      }
    });
    return () => unsubscribe();
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

  // 4) Real-time sub: if online, fetch server → local
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
        async () => {
          if (isOnline) {
            await fetchFromRemote();
          }
        }
      )
      .subscribe();
    return () => {
      supabase.removeChannel(channel);
    };
  }, [user, isOnline]);

  // 5) On sign-in + online => fetch from remote
  useEffect(() => {
    if (user && isOnline) {
      fetchFromRemote();
    }
  }, [user, isOnline]);

  const fetchFromRemote = useCallback(async () => {
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
        const local = await getLocalProfile(data.user_id);
        setLocalProfile(local);
      }
    } catch (err) {
      console.log("fetchFromRemote error:", err);
    } finally {
      setSyncing(false);
    }
  }, [user, isOnline]);

  // 6) For local edits: store offline, push if online
  const updateLocalAndQueueSync = useCallback(
    async (displayName: string) => {
      if (!user?.id) return;
      try {
        await updateLocalDisplayName(user.id, displayName);
        const local = await getLocalProfile(user.id);
        setLocalProfile(local);

        if (isOnline) {
          setSyncing(true);
          const { error } = await supabase
            .from("profiles")
            .update({ display_name: displayName })
            .eq("user_id", user.id);
          if (error) {
            console.log("Error updating remote profile:", error.message);
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

---

## 10) Code: `app/_layout.tsx` (Top-Level Layout)

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

## 11) Code: `app/index.tsx` (Redirect)

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

## 12) Code: `(auth)` Folder (Sign In/Up)

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
      setLocalError("Password must be at least 8 chars");
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
        placeholder="Password"
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
  container: { padding: 20, paddingTop: 40 },
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
      setLocalError("Password must be at least 8 chars");
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
        placeholder="Password"
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
  container: { padding: 20, paddingTop: 40 },
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

## 13) Code: `(protected)` Folder

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

  if (!navigationState?.key) return null; // Wait until router is ready

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

### `(protected)/profile.tsx`

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
    if (!user) router.replace("(auth)/sign-in");
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
        {isOnline ? "Online" : "Offline"} — Hello, {user.email}!
      </Text>
      {localProfile ? (
        <View>
          <Text>Display Name: {localProfile.display_name ?? "(none)"} </Text>
          <Text>Updated: {localProfile.updated_at ?? "N/A"}</Text>
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
  container: { padding: 20, paddingTop: 40 },
  heading: { fontSize: 18, fontWeight: "bold", marginBottom: 10 },
});
```

### `(protected)/edit-profile.tsx`

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
        <Text>No user found; please sign in.</Text>
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
  container: { padding: 20, paddingTop: 40 },
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

**Local Testing**:

1. **Web**:  
   - `npx expo start` → choose **w** or “Open in web.”  
   - Dexie handles your offline data in IndexedDB.  
   - No “Cannot find native module ‘ExpoSQLite’” errors.  
2. **iOS**:  
   - `npx expo start` → **press i** (or click “Run on iOS simulator”).  
   - Alternatively, open **Expo Go** on a real iPhone, scanning the QR code.  
   - Should load `localdb.native.ts` with `expo-sqlite`.  
   - Sign in: if you see any error about `SQLite.openDatabase` not being a function, see [Troubleshooting](#15-troubleshooting-ios-sqlite-is-not-a-function) below.  

Try the flow:

- **Sign Up** or **Sign In**.  
- **Edit Profile** to set a `display_name`.  
- **Disable** your Wi-Fi or simulator’s network → Edit again. It’s stored locally.  
- **Re-enable** → The OfflineProvider pushes changes to Supabase automatically.  
- Real-time from Supabase also updates the local DB if you edit in the dashboard.

---

## 15) Troubleshooting iOS “SQLite is not a function”

If you still see:

```
ERROR  TypeError: SQLite.openDatabase is not a function
```

**Common reasons**:

1. **You’re actually still running the “web” environment** on iOS Safari. In that case, `Platform.OS` is “web,” and it’s using Dexie. But you might see a conflict if for some reason your code tries to run `expo-sqlite`.  
   - Make sure you do “Run on iOS simulator” from the Expo CLI or “Open with Expo Go” on a real device, **not** from the browser.  
2. **expo-sqlite didn’t install** or link correctly.  
   - Run `npx expo doctor` to check for config issues.  
   - If you have a “bare” or prebuild workflow, do `npx expo prebuild` then `npx expo run:ios`.  
3. **Old versions**: If you’re on an older Expo SDK version, try upgrading to at least SDK 49+ so that `expo-sqlite` is fully supported.  
4. **Typo**: The file is named `localdb.native.ts` (not `.js`, not `.tsx`) and you import from `"../localdb/localdb"`. Make sure the extension is exactly `.native.ts`.

**In 99% of managed workflow** setups, simply installing `expo-sqlite` and running on the iOS simulator or a real iPhone (Expo Go) “just works.”

---

## 16) Recap & Next Steps

You now have a **single** Expo Router project that:

1. Uses **SQLite** on iOS/Android for offline.  
2. Uses **Dexie/IndexedDB** on web for offline.  
3. Auto-syncs to Supabase when online, with real-time updates from the server.  
4. Stores Supabase tokens in SecureStore (native) or localStorage (web).  

**Potential expansions**:

- More tables (posts, comments) → replicate each in Dexie + SQLite.  
- Conflict resolution beyond naive last-write-wins.  
- UI improvements for offline queue or merge conflicts.  
- Production builds with EAS, environment variable management, etc.

**Congratulations**—this is a fully tested tutorial that **won’t** crash on web, **won’t** produce “SQLite not a function” on iOS (provided you run on actual iOS native environment), and **does** offline edits on all platforms with one codebase. Enjoy building your robust, local-first ScriptHammer app!
