Below is **one** cohesive, **progressive** tutorial that includes everything—**no** “later” sections for the admin dashboard. Instead, we build it **from scratch** with the `role` column, “first user = admin” logic, and `(admin)` folder **right away**. We also insert **three** dummy users (Jane, Jon, James) after the first admin, so you can easily test user lists in your admin panel.

---

# ScriptHammer - Dexie on Web, SQLite + SecureStore on Native, **Plus** Admin Dashboard (Single Tutorial)

## Table of Contents

1. [Create a New Expo Project](#1-create-a-new-expo-project)  
2. [Install Dependencies](#2-install-dependencies)  
3. [Set Up `.env.local`](#3-set-up-envlocal)  
4. [Supabase Setup (Admin in Mind + 3 Dummy Users)](#4-supabase-setup-admin-in-mind--3-dummy-users)  
   - [Create or Confirm `profiles` Table (With `role`)](#create-or-confirm-profiles-table-with-role)  
   - [RLS & Policies (Admin Logic)](#rls--policies-admin-logic)  
   - [DB Trigger for First-User-Is-Admin](#db-trigger-for-first-user-is-admin)  
   - [Insert 3 Dummy Users](#insert-3-dummy-users)  
   - [Enable Realtime](#enable-realtime)  
5. [File & Folder Structure (Manual Creation)](#5-file--folder-structure-manual-creation)  
   - [Skip Manual Setup? Jump to Step 19 for the Script](#skip-manual-setup-jump-to-step-19-for-the-script)  
6. [Code: `localdb.native.ts` (Expo SQLite)](#6-code-localdbnativets-expo-sqlite)  
7. [Code: `localdb.web.ts` (Dexie)](#7-code-localdbwebts-dexie)  
8. [Code: `supabaseClient.ts` (Connection)](#8-code-supabaseclientts-connection)  
9. [Code: `auth.native.tsx` (Auth Context for iOS/Android)](#9-code-authnativetsx-auth-context-for-iosandroid)  
10. [Code: `auth.web.tsx` (Auth Context for Web)](#10-code-authwebtsx-auth-context-for-web)  
11. [Code: `context/offline.tsx` (Offline Context)](#11-code-contextofflinetsx-offline-context)  
12. [Code: `app/_layout.tsx` (Top-Level Layout)](#12-code-app_layouttsx-top-level-layout)  
13. [Code: `app/index.tsx` (Redirect on Launch)](#13-code-appindextsx-redirect-on-launch)  
14. [Code: `(auth)` Folder (Sign In & Sign Up)](#14-code-auth-folder-sign-in--sign-up)  
15. [Code: `(protected)` Folder (Profile + Edit)](#15-code-protected-folder-profile--edit)  
16. [Code: `(admin)` Folder (All-In-One Admin)](#16-code-admin-folder-all-in-one-admin)  
    - [`_layout.tsx`](#layouttsx-for-admin)  
    - [`index.tsx` (Dashboard Home)](#indextsx-dashboard-home)  
    - [`users.tsx` (User Management)](#userstsx-user-management)  
    - [`content.tsx` (Content Moderation)](#contenttsx-content-moderation)  
17. [Run & Test](#17-run--test)  
18. [Troubleshooting SecureStore or SQLite Issues](#18-troubleshooting-securestore-or-sqlite-issues)  
19. [Scaffolding Script (Generates All Files)](#19-scaffolding-script-generates-all-files)  
20. [Next Steps](#20-next-steps)  

---

## 1) Create a New Expo Project

```bash
npx create-expo-app ScriptHammer
cd ScriptHammer

# Optional: remove leftover example screens
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

- **expo-secure-store**: Used **only** on native (iOS/Android).  
- **expo-sqlite**: Used **only** on native for local DB.  
- **dexie**: Used **only** on web.  

---

## 3) Set Up `.env.local`

At your **project root**:

```bash
EXPO_PUBLIC_SUPABASE_URL=https://YOUR-PROJECT.supabase.co
EXPO_PUBLIC_SUPABASE_ANON_KEY=YOUR-ANON-KEY
```

Ignore it in git:

```bash
# .gitignore
.env.local
```

---

## 4) Supabase Setup (Admin in Mind + 3 Dummy Users)

### Create or Confirm `profiles` Table (With `role`)

```sql
create table if not exists profiles (
  id uuid primary key default uuid_generate_v4(),
  user_id uuid references auth.users not null,
  display_name text,
  role text not null default 'user',
  created_at timestamp default now()
);
```

### RLS & Policies (Admin Logic)

```sql
alter table public.profiles enable row level security;

create policy "Select own or admin"
on public.profiles
for select
using (
  auth.uid() = user_id
  or (select role from public.profiles where user_id = auth.uid()) = 'admin'
);

create policy "Update own or admin"
on public.profiles
for update
using (
  auth.uid() = user_id
  or (select role from public.profiles where user_id = auth.uid()) = 'admin'
);
```

### DB Trigger for First-User-Is-Admin

```sql
create or replace function handle_new_user()
returns trigger as $$
declare
  existing_count int;
begin
  select count(*) into existing_count 
    from public.profiles;

  if existing_count = 0 then
    -- The first user is admin
    insert into public.profiles (user_id, display_name, role)
    values (new.id, '', 'admin');
  else
    -- All subsequent users are user
    insert into public.profiles (user_id, display_name, role)
    values (new.id, '', 'user');
  end if;

  return new;
end;
$$ language plpgsql security definer;

create trigger on_auth_user_created
  after insert on auth.users
  for each row
  execute procedure handle_new_user();
```

### Insert 3 Dummy Users

Next, **optionally** insert three “dummy” users to test user listing in the admin. Since the **first** user to appear in `auth.users` was presumably your real sign-up account (the “admin”), these three become `'user'`:

```sql
insert into auth.users (email)
values
  ('jane@doe.com'),
  ('jon@doe.com'),
  ('james@doe.com');
```

**Note**: These dummy accounts have **no** passwords set, so they can’t sign in. They simply appear in `auth.users`—and via our trigger, in `profiles` (with `role='user'`). This is great for testing how your admin sees multiple accounts.

### Enable Realtime

In **Supabase UI** → Table Editor → `profiles` → enable **Realtime**. Done.

---

## 5) File & Folder Structure (Manual Creation)

We’ll create **everything** in one shot, including the `(admin)` folder, so the admin panel is integrated from day one:

```bash
mkdir -p context
touch context/offline.tsx
touch context/auth.native.tsx
touch context/auth.web.tsx

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

# Admin panel included from the start
mkdir -p app/\(admin\)
touch app/\(admin\)/_layout.tsx
touch app/\(admin\)/index.tsx
touch app/\(admin\)/users.tsx
touch app/\(admin\)/content.tsx

touch supabaseClient.ts
touch app/_layout.tsx
touch app/index.tsx
touch .env.local
```

**Want to skip this manual process?** See [Step 19](#19-scaffolding-script-generates-all-files) for a single Node script that writes every file.

---

## 6) Code: `localdb.native.ts` (Expo SQLite)

```ts
// localdb/localdb.native.ts
import * as SQLite from "expo-sqlite";

const db = SQLite.openDatabase("ScriptHammer.db");

// Create a local table for user profiles, storing display_name, role, etc.
export async function setupLocalDatabase() {
  return new Promise<void>((resolve, reject) => {
    db.transaction((tx) => {
      tx.executeSql(
        `CREATE TABLE IF NOT EXISTS local_profiles (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          user_id TEXT UNIQUE,
          display_name TEXT,
          role TEXT,
          updated_at TEXT
        );`,
        [],
        () => resolve(),
        (_, error) => {
          console.error("Error creating local_profiles table:", error);
          reject(error);
          return false;
        }
      );
    });
  });
}

// Insert or update a row in local_profiles
export async function upsertLocalProfile(
  userId: string,
  displayName: string,
  role: string,
  updatedAt: string
) {
  return new Promise<void>((resolve, reject) => {
    db.transaction((tx) => {
      tx.executeSql(
        `
        INSERT INTO local_profiles (user_id, display_name, role, updated_at)
        VALUES (?, ?, ?, ?)
        ON CONFLICT(user_id) DO UPDATE
          SET display_name=excluded.display_name,
              role=excluded.role,
              updated_at=excluded.updated_at
      `,
        [userId, displayName, role, updatedAt],
        () => resolve(),
        (_, error) => {
          console.error("Error upserting local profile:", error);
          reject(error);
          return false;
        }
      );
    });
  });
}

// Fetch a single local profile
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
          console.error("Error fetching local profile:", error);
          reject(error);
          return false;
        }
      );
    });
  });
}

// Update just the display_name in local_profiles
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
        (_, error) => {
          console.error("Error updating local profile:", error);
          reject(error);
          return false;
        }
      );
    });
  });
}
```

---

## 7) Code: `localdb.web.ts` (Dexie)

```ts
// localdb/localdb.web.ts
import Dexie from "dexie";

// Dexie DB for storing local_profiles on web
const db = new Dexie("ScriptHammerWebDB");
db.version(1).stores({
  local_profiles: "user_id, display_name, role, updated_at",
});

export async function setupLocalDatabase() {
  console.log("Dexie is ready on web for offline DB");
}

export async function upsertLocalProfile(
  userId: string,
  displayName: string,
  role: string,
  updatedAt: string
) {
  await db.table("local_profiles").put({
    user_id: userId,
    display_name: displayName,
    role,
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

## 8) Code: `supabaseClient.ts` (Connection)

```ts
// supabaseClient.ts
import { createClient } from "@supabase/supabase-js";

const SUPABASE_URL = process.env.EXPO_PUBLIC_SUPABASE_URL!;
const SUPABASE_ANON_KEY = process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY!;

export const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
```

---

## 9) Code: `auth.native.tsx` (Auth Context for iOS/Android)

```tsx
// context/auth.native.tsx
import React, { createContext, useContext, useEffect, useState } from "react";
import { supabase } from "../supabaseClient";
import { Session } from "@supabase/supabase-js";
import * as SecureStore from "expo-secure-store";

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

async function setItem(key: string, value: string) {
  await SecureStore.setItemAsync(key, value);
}
async function getItem(key: string) {
  return await SecureStore.getItemAsync(key);
}
async function deleteItem(key: string) {
  await SecureStore.deleteItemAsync(key);
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

    const {
      data: { subscription },
    } = supabase.auth.onAuthStateChange((_event, session) => {
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
      setError(err.message || "Sign-up failed");
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
      setError(err.message || "Sign-in failed");
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
      setError(err.message || "Sign-out failed");
      console.log("Sign-out error:", err);
    } finally {
      setLoading(false);
    }
  }

  return (
    <AuthContext.Provider value={{ user, loading, error, signUp, signIn, signOut }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  return useContext(AuthContext);
}
```

---

## 10) Code: `auth.web.tsx` (Auth Context for Web)

```tsx
// context/auth.web.tsx
import React, { createContext, useContext, useEffect, useState } from "react";
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

function setItem(key: string, value: string) {
  window.localStorage.setItem(key, value);
}
function getItem(key: string) {
  return window.localStorage.getItem(key) ?? null;
}
function deleteItem(key: string) {
  window.localStorage.removeItem(key);
}

export default function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<any>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  // Attempt to restore from localStorage
  useEffect(() => {
    (async () => {
      try {
        const accessToken = getItem(SESSION_KEY_ACCESS);
        const refreshToken = getItem(SESSION_KEY_REFRESH);
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

    const {
      data: { subscription },
    } = supabase.auth.onAuthStateChange((_event, session) => {
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
    if (access_token) setItem(SESSION_KEY_ACCESS, access_token);
    if (refresh_token) setItem(SESSION_KEY_REFRESH, refresh_token);
  }
  async function clearTokens() {
    deleteItem(SESSION_KEY_ACCESS);
    deleteItem(SESSION_KEY_REFRESH);
  }

  async function signUp(email: string, password: string) {
    setError(null);
    setLoading(true);
    try {
      const { error: signUpError } = await supabase.auth.signUp({ email, password });
      if (signUpError) throw new Error(signUpError.message);
    } catch (err: any) {
      setError(err.message || "Sign-up failed");
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
      setError(err.message || "Sign-in failed");
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
      setError(err.message || "Sign-out failed");
      console.log("Sign-out error:", err);
    } finally {
      setLoading(false);
    }
  }

  return (
    <AuthContext.Provider value={{ user, loading, error, signUp, signIn, signOut }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  return useContext(AuthContext);
}
```

---

## 11) Code: `context/offline.tsx` (Offline Context)

```tsx
// context/offline.tsx
import React, { createContext, useContext, useEffect, useState, useCallback } from "react";
import NetInfo from "@react-native-community/netinfo";
import { supabase } from "../supabaseClient";
import {
  setupLocalDatabase,
  getLocalProfile,
  upsertLocalProfile,
  updateLocalDisplayName,
} from "../localdb/localdb";
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

  // 2) NetInfo for connectivity
  useEffect(() => {
    const unsubscribe = NetInfo.addEventListener((state) => {
      setIsOnline(!!state.isConnected);
    });
    return () => unsubscribe();
  }, []);

  // 3) Load local data if user logs in
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

  // 4) Real-time subscription for profiles
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
            await fetchRemoteProfile();
          }
        }
      )
      .subscribe();

    return () => {
      supabase.removeChannel(channel);
    };
  }, [user, isOnline]);

  async function fetchRemoteProfile() {
    if (!user?.id || !isOnline) return;
    try {
      setSyncing(true);
      const { data, error } = await supabase
        .from("profiles")
        .select("*")
        .eq("user_id", user.id)
        .single();
      if (!error && data) {
        const updatedAt = new Date().toISOString();
        // We also store 'role' locally
        await upsertLocalProfile(data.user_id, data.display_name ?? "", data.role ?? "user", updatedAt);
        const localData = await getLocalProfile(data.user_id);
        setLocalProfile(localData);
      }
    } catch (err) {
      console.log("fetchRemoteProfile error:", err);
    } finally {
      setSyncing(false);
    }
  }

  // 5) For local edits: store offline, push if online
  const updateLocalAndQueueSync = useCallback(
    async (displayName: string) => {
      if (!user?.id) return;
      setSyncing(true);
      try {
        await updateLocalDisplayName(user.id, displayName);
        const localData = await getLocalProfile(user.id);
        setLocalProfile(localData);

        if (isOnline) {
          const { error } = await supabase
            .from("profiles")
            .update({ display_name: displayName })
            .eq("user_id", user.id);
          if (error) {
            console.log("Error updating remote profile:", error.message);
          }
        }
      } catch (err) {
        console.log("updateLocalAndQueueSync error:", err);
      } finally {
        setSyncing(false);
      }
    },
    [user, isOnline]
  );

  return (
    <OfflineContext.Provider
      value={{ localProfile, isOnline, syncing, updateLocalAndQueueSync }}
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

## 12) Code: `app/_layout.tsx` (Top-Level Layout)

```tsx
// app/_layout.tsx
import { Stack } from "expo-router";
import AuthProvider from "../context/auth"; // picks native or web
import { OfflineProvider } from "../context/offline";
import { StatusBar } from "expo-status-bar";

export default function RootLayout() {
  return (
    <AuthProvider>
      <OfflineProvider>
        <Stack screenOptions={{ headerShown: false }} />
        <StatusBar style="auto" />
      </OfflineProvider>
    </AuthProvider>
  );
}
```

---

## 13) Code: `app/index.tsx` (Redirect on Launch)

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

## 14) Code: `(auth)` Folder (Sign In & Sign Up)

### `(auth)/_layout.tsx`

```tsx
// app/(auth)/_layout.tsx
import { Stack } from "expo-router";

export default function AuthLayout() {
  return <Stack screenOptions={{ headerShown: true, headerTitle: "Auth Flow" }} />;
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
  const { user, signIn, loading, error } = useAuth();
  const router = useRouter();

  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [localError, setLocalError] = useState("");

  useEffect(() => {
    if (user) router.replace("(protected)/profile");
  }, [user]);

  async function handleSignIn() {
    if (!/\S+@\S+\.\S+/.test(email)) {
      setLocalError("Invalid email");
      return;
    }
    if (password.length < 8) {
      setLocalError("Must be at least 8 chars");
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
        onChangeText={setEmail}
        value={email}
        autoCapitalize="none"
      />
      <TextInput
        style={styles.input}
        placeholder="Password"
        onChangeText={setPassword}
        value={password}
        secureTextEntry
      />

      <Button title={loading ? "Signing In..." : "Sign In"} onPress={handleSignIn} />

      <Link href="/(auth)/sign-up" style={styles.link}>
        Don’t have an account? Sign Up
      </Link>
    </View>
  );
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
  const { user, signUp, loading, error } = useAuth();
  const router = useRouter();

  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [confirmPass, setConfirmPass] = useState("");
  const [localError, setLocalError] = useState("");

  useEffect(() => {
    if (user) router.replace("(protected)/profile");
  }, [user]);

  async function handleSignUp() {
    if (!/\S+@\S+\.\S+/.test(email)) {
      setLocalError("Invalid email");
      return;
    }
    if (password.length < 8) {
      setLocalError("At least 8 chars");
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
        onChangeText={setEmail}
        value={email}
        autoCapitalize="none"
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

## 15) Code: `(protected)` Folder (Profile + Edit)

### `(protected)/_layout.tsx`

```tsx
// app/(protected)/_layout.tsx
import { Stack, useRouter, useRootNavigationState } from "expo-router";
import { useEffect } from "react";
import { useAuth } from "../../context/auth";

export default function ProtectedLayout() {
  const router = useRouter();
  const navState = useRootNavigationState();
  const { user } = useAuth();

  if (!navState?.key) return null; // Wait for router

  useEffect(() => {
    if (!user) {
      router.replace("(auth)/sign-in");
    }
  }, [user]);

  return (
    <Stack screenOptions={{ headerShown: true, headerTitle: "Protected" }} />
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
        {isOnline ? "Online" : "Offline"} — Hello, {user.email}
      </Text>
      {localProfile ? (
        <>
          <Text>Display: {localProfile.display_name || "(none)"} </Text>
          <Text>Role: {localProfile.role}</Text>
          <Text>Updated: {localProfile.updated_at || "N/A"}</Text>
        </>
      ) : (
        <Text>Loading local profile...</Text>
      )}

      <View style={{ marginTop: 20 }}>
        <Button
          title="Edit Profile"
          onPress={() => router.push("(protected)/edit-profile")}
        />
      </View>

      {/* If you're admin, show link to admin dashboard */}
      {localProfile?.role === "admin" && (
        <View style={{ marginTop: 20 }}>
          <Button
            title="Go to Admin Dashboard"
            onPress={() => router.push("(admin)")}
          />
        </View>
      )}

      <View style={{ marginTop: 20 }}>
        <Button title="Sign Out" onPress={handleSignOut} />
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { paddingHorizontal: 20, paddingTop: 40 },
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

## 16) Code: `(admin)` Folder (All-In-One Admin)

### `_layout.tsx` for Admin

```tsx
// app/(admin)/_layout.tsx
import { Stack, useRouter, useRootNavigationState } from "expo-router";
import { useEffect } from "react";
import { useAuth } from "../../context/auth";
import { useOffline } from "../../context/offline";

export default function AdminLayout() {
  const router = useRouter();
  const navigationState = useRootNavigationState();
  const { user } = useAuth();
  const { localProfile } = useOffline();

  if (!navigationState?.key) return null; // Wait for router

  useEffect(() => {
    // If not logged in or user is not admin, go to normal profile
    if (!user) {
      router.replace("(auth)/sign-in");
      return;
    }
    if (localProfile?.role !== "admin") {
      router.replace("(protected)/profile");
    }
  }, [user, localProfile]);

  return <Stack screenOptions={{ headerShown: true, headerTitle: "Admin" }} />;
}
```

### `index.tsx` (Dashboard Home)

```tsx
// app/(admin)/index.tsx
import { View, Text, Button, StyleSheet } from "react-native";
import { useRouter } from "expo-router";

export default function AdminHome() {
  const router = useRouter();

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Admin Dashboard</Text>

      <Button
        title="Manage Users"
        onPress={() => router.push("/(admin)/users")}
      />
      <Button
        title="Moderate Content"
        onPress={() => router.push("/(admin)/content")}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { padding: 20, marginTop: 40 },
  title: { fontSize: 24, fontWeight: "bold", marginBottom: 20 },
});
```

### `users.tsx` (User Management)

```tsx
// app/(admin)/users.tsx
import { View, Text, StyleSheet, FlatList, Button } from "react-native";
import { useState, useEffect } from "react";
import { supabase } from "../../supabaseClient";
import { useRouter } from "expo-router";

interface Profile {
  user_id: string;
  display_name: string;
  role: string;
}

export default function ManageUsers() {
  const router = useRouter();
  const [profiles, setProfiles] = useState<Profile[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchProfiles();
  }, []);

  async function fetchProfiles() {
    setLoading(true);
    const { data, error } = await supabase.from("profiles").select("*");
    if (error) {
      console.log("Error fetching profiles:", error.message);
      setLoading(false);
      return;
    }
    setProfiles(data as Profile[]);
    setLoading(false);
  }

  async function promoteToAdmin(userId: string) {
    const { error } = await supabase
      .from("profiles")
      .update({ role: "admin" })
      .eq("user_id", userId);
    if (error) {
      console.log("Error promoting user:", error.message);
    } else {
      fetchProfiles();
    }
  }

  async function demoteToUser(userId: string) {
    const { error } = await supabase
      .from("profiles")
      .update({ role: "user" })
      .eq("user_id", userId);
    if (error) {
      console.log("Error demoting user:", error.message);
    } else {
      fetchProfiles();
    }
  }

  return (
    <View style={styles.container}>
      <Text style={styles.heading}>Manage Users</Text>
      {loading && <Text>Loading...</Text>}

      <FlatList
        data={profiles}
        keyExtractor={(item) => item.user_id}
        renderItem={({ item }) => {
          return (
            <View style={styles.userRow}>
              <View style={{ flex: 1 }}>
                <Text>{item.display_name || "(no name)"}</Text>
                <Text style={styles.role}>{item.role}</Text>
                <Text style={styles.userId}>{item.user_id}</Text>
              </View>
              <View style={styles.buttons}>
                {item.role === "admin" ? (
                  <Button
                    title="Demote"
                    onPress={() => demoteToUser(item.user_id)}
                  />
                ) : (
                  <Button
                    title="Promote"
                    onPress={() => promoteToAdmin(item.user_id)}
                  />
                )}
              </View>
            </View>
          );
        }}
      />

      <View style={{ marginTop: 20 }}>
        <Button title="Back to Admin Home" onPress={() => router.push("/(admin)")} />
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { padding: 20, marginTop: 40 },
  heading: { fontSize: 20, fontWeight: "bold", marginBottom: 10 },
  userRow: {
    flexDirection: "row",
    alignItems: "center",
    marginBottom: 10,
    borderBottomWidth: 1,
    borderBottomColor: "#ccc",
    paddingBottom: 8,
  },
  role: { color: "#666" },
  userId: { color: "#999", fontSize: 12 },
  buttons: { flexDirection: "row", gap: 10 },
});
```

### `content.tsx` (Content Moderation)

```tsx
// app/(admin)/content.tsx
import { View, Text, StyleSheet, FlatList, Button } from "react-native";
import { useState, useEffect } from "react";
import { supabase } from "../../supabaseClient";
import { useRouter } from "expo-router";

interface Post {
  id: string;
  author_id: string;
  title: string;
  body: string;
  status: string;
}

export default function ContentModeration() {
  const router = useRouter();
  const [posts, setPosts] = useState<Post[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchPosts();
  }, []);

  async function fetchPosts() {
    setLoading(true);
    const { data, error } = await supabase.from("posts").select("*");
    if (error) {
      console.log("Error fetching posts:", error.message);
      setLoading(false);
      return;
    }
    setPosts(data as Post[]);
    setLoading(false);
  }

  async function setStatus(id: string, status: string) {
    const { error } = await supabase
      .from("posts")
      .update({ status })
      .eq("id", id);
    if (error) {
      console.log("Error updating post:", error.message);
    } else {
      fetchPosts();
    }
  }

  return (
    <View style={styles.container}>
      <Text style={styles.heading}>Content Moderation</Text>
      {loading && <Text>Loading...</Text>}

      <FlatList
        data={posts}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => {
          return (
            <View style={styles.postRow}>
              <View style={{ flex: 1 }}>
                <Text style={styles.title}>{item.title}</Text>
                <Text>{item.body}</Text>
                <Text style={styles.status}>Status: {item.status}</Text>
              </View>
              <View style={styles.buttons}>
                <Button
                  title="Publish"
                  onPress={() => setStatus(item.id, "published")}
                />
                <Button
                  title="Draft"
                  onPress={() => setStatus(item.id, "draft")}
                />
              </View>
            </View>
          );
        }}
      />

      <View style={{ marginTop: 20 }}>
        <Button title="Back to Admin Home" onPress={() => router.push("/(admin)")} />
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { padding: 20, marginTop: 40 },
  heading: { fontSize: 20, fontWeight: "bold", marginBottom: 10 },
  postRow: {
    borderBottomColor: "#ccc",
    borderBottomWidth: 1,
    marginBottom: 10,
    paddingBottom: 8,
  },
  title: { fontWeight: "bold" },
  status: { color: "#666" },
  buttons: { flexDirection: "row", gap: 10, marginTop: 5 },
});
```

---

## 17) Run & Test

```bash
npx expo start --clear
```

- **On iOS/Android**: The code automatically uses `auth.native.tsx` + `localdb.native.ts` with SecureStore + SQLite.  
- **On Web**: The code automatically uses `auth.web.tsx` + `localdb.web.ts` with localStorage + Dexie.  

**Walkthrough**:

1. **Sign Up**: If you sign up and no prior profile exists, you become **admin**.  
2. Supabase automatically inserts three dummy users (`jane@doe.com`, `jon@doe.com`, `james@doe.com`) with role = `'user'`.  
3. **View Profile**: If your `role='admin'`, you’ll see a “Go to Admin Dashboard” button.  
4. **Admin Panel** (`(admin)/index.tsx`): Manage other users or moderate content.  
5. Normal users see no mention of admin.  

---

## 18) Troubleshooting SecureStore or SQLite Issues

1. If **web** tries to import `expo-secure-store`, double-check:
   - `auth.native.tsx` → iOS/Android  
   - `auth.web.tsx` → Web  
2. If you never installed them, do:
   ```bash
   npx expo install expo-secure-store expo-sqlite
   ```
3. If you’re in a bare or prebuild workflow, run `npx expo prebuild && npx expo run:ios` to link iOS.  
4. If you have an older Expo SDK, consider upgrading.

---

## 19) Scaffolding Script (Generates All Files)

If you want to skip Steps 5–16, copy no code manually, and have one script that overwrites/creates **every** file (including `(admin)`, Dexie, and so on):

1. Make a `scripts/` folder:
   ```bash
   mkdir -p scripts
   ```
2. Create `scaffold-ScriptHammer-Admin.js`:
   ```bash
   touch scripts/scaffold-ScriptHammer-Admin.js
   chmod +x scripts/scaffold-ScriptHammer-Admin.js
   ```
3. In `package.json`, add:
   ```json
   {
     "scripts": {
       "scaffold-ScriptHammer-Admin": "node ./scripts/scaffold-ScriptHammer-Admin.js"
     }
   }
   ```
4. Paste the code below (the big array includes all files from Steps 6–16, plus admin):

```js
#!/usr/bin/env node

/**
 * scaffold-ScriptHammer-Admin.js
 *
 * A Node.js script that overwrites/creates all files from the
 * "ScriptHammer + Admin Dashboard" single-tutorial approach
 * (Dexie on Web, SQLite + SecureStore on Native, plus integrated admin).
 *
 * USAGE:
 *   npm run scaffold-ScriptHammer-Admin
 *
 * WARNING:
 *   - Overwrites existing files if paths match.
 *   - Make sure you commit or backup first.
 */

const fs = require("fs");
const path = require("path");
const readline = require("readline");

// Below is the full array of file objects for Steps 6–16 in the final tutorial,
// using ASCII-only strings so there are no "invalid token" errors.

const FILES = [
  {
    filePath: "localdb/localdb.native.ts",
    content: `import * as SQLite from "expo-sqlite";

const db = SQLite.openDatabase("ScriptHammer.db");

// Create a local table for user profiles, storing display_name, role, etc.
export async function setupLocalDatabase() {
  return new Promise<void>((resolve, reject) => {
    db.transaction((tx) => {
      tx.executeSql(
        "CREATE TABLE IF NOT EXISTS local_profiles (\\n          id INTEGER PRIMARY KEY AUTOINCREMENT,\\n          user_id TEXT UNIQUE,\\n          display_name TEXT,\\n          role TEXT,\\n          updated_at TEXT\\n        );",
        [],
        () => resolve(),
        (_, error) => {
          console.error("Error creating local_profiles table:", error);
          reject(error);
          return false;
        }
      );
    });
  });
}

// Insert or update a row in local_profiles
export async function upsertLocalProfile(
  userId: string,
  displayName: string,
  role: string,
  updatedAt: string
) {
  return new Promise<void>((resolve, reject) => {
    db.transaction((tx) => {
      tx.executeSql(
        "INSERT INTO local_profiles (user_id, display_name, role, updated_at)\\n        VALUES (?, ?, ?, ?)\\n        ON CONFLICT(user_id) DO UPDATE\\n          SET display_name=excluded.display_name,\\n              role=excluded.role,\\n              updated_at=excluded.updated_at\\n      ",
        [userId, displayName, role, updatedAt],
        () => resolve(),
        (_, error) => {
          console.error("Error upserting local profile:", error);
          reject(error);
          return false;
        }
      );
    });
  });
}

// Fetch a single local profile
export async function getLocalProfile(userId: string) {
  return new Promise<any>((resolve, reject) => {
    db.transaction((tx) => {
      tx.executeSql(
        "SELECT * FROM local_profiles WHERE user_id=? LIMIT 1",
        [userId],
        (_, result) => {
          if (result.rows.length > 0) {
            resolve(result.rows.item(0));
          } else {
            resolve(null);
          }
        },
        (_, error) => {
          console.error("Error fetching local profile:", error);
          reject(error);
          return false;
        }
      );
    });
  });
}

// Update just the display_name in local_profiles
export async function updateLocalDisplayName(userId: string, displayName: string) {
  const now = new Date().toISOString();
  return new Promise<void>((resolve, reject) => {
    db.transaction((tx) => {
      tx.executeSql(
        "UPDATE local_profiles\\n         SET display_name=?, updated_at=?\\n         WHERE user_id=?",
        [displayName, now, userId],
        () => resolve(),
        (_, error) => {
          console.error("Error updating local profile:", error);
          reject(error);
          return false;
        }
      );
    });
  });
}
`
  },
  {
    filePath: "localdb/localdb.web.ts",
    content: `import Dexie from "dexie";

// Dexie DB for storing local_profiles on web
const db = new Dexie("ScriptHammerWebDB");
db.version(1).stores({
  local_profiles: "user_id, display_name, role, updated_at",
});

export async function setupLocalDatabase() {
  console.log("Dexie is ready on web for offline DB");
}

export async function upsertLocalProfile(
  userId: string,
  displayName: string,
  role: string,
  updatedAt: string
) {
  await db.table("local_profiles").put({
    user_id: userId,
    display_name: displayName,
    role,
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
`
  },
  {
    filePath: "supabaseClient.ts",
    content: `import { createClient } from "@supabase/supabase-js";

const SUPABASE_URL = process.env.EXPO_PUBLIC_SUPABASE_URL!;
const SUPABASE_ANON_KEY = process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY!;

export const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
`
  },
  {
    filePath: "context/auth.native.tsx",
    content: `import React, { createContext, useContext, useEffect, useState } from "react";
import { supabase } from "../supabaseClient";
import { Session } from "@supabase/supabase-js";
import * as SecureStore from "expo-secure-store";

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

async function setItem(key: string, value: string) {
  await SecureStore.setItemAsync(key, value);
}
async function getItem(key: string) {
  return await SecureStore.getItemAsync(key);
}
async function deleteItem(key: string) {
  await SecureStore.deleteItemAsync(key);
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

    const {
      data: { subscription },
    } = supabase.auth.onAuthStateChange((_event, session) => {
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
      setError(err.message || "Sign-up failed");
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
      setError(err.message || "Sign-in failed");
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
      setError(err.message || "Sign-out failed");
      console.log("Sign-out error:", err);
    } finally {
      setLoading(false);
    }
  }

  return (
    <AuthContext.Provider value={{ user, loading, error, signUp, signIn, signOut }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  return useContext(AuthContext);
}
`
  },
  {
    filePath: "context/auth.web.tsx",
    content: `import React, { createContext, useContext, useEffect, useState } from "react";
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

function setItem(key: string, value: string) {
  window.localStorage.setItem(key, value);
}
function getItem(key: string) {
  return window.localStorage.getItem(key) ?? null;
}
function deleteItem(key: string) {
  window.localStorage.removeItem(key);
}

export default function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<any>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  // Attempt to restore from localStorage
  useEffect(() => {
    (async () => {
      try {
        const accessToken = getItem(SESSION_KEY_ACCESS);
        const refreshToken = getItem(SESSION_KEY_REFRESH);
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

    const {
      data: { subscription },
    } = supabase.auth.onAuthStateChange((_event, session) => {
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
    if (access_token) setItem(SESSION_KEY_ACCESS, access_token);
    if (refresh_token) setItem(SESSION_KEY_REFRESH, refresh_token);
  }
  async function clearTokens() {
    deleteItem(SESSION_KEY_ACCESS);
    deleteItem(SESSION_KEY_REFRESH);
  }

  async function signUp(email: string, password: string) {
    setError(null);
    setLoading(true);
    try {
      const { error: signUpError } = await supabase.auth.signUp({ email, password });
      if (signUpError) throw new Error(signUpError.message);
    } catch (err: any) {
      setError(err.message || "Sign-up failed");
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
      setError(err.message || "Sign-in failed");
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
      setError(err.message || "Sign-out failed");
      console.log("Sign-out error:", err);
    } finally {
      setLoading(false);
    }
  }

  return (
    <AuthContext.Provider value={{ user, loading, error, signUp, signIn, signOut }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  return useContext(AuthContext);
}
`
  },
  {
    filePath: "context/offline.tsx",
    content: `import React, { createContext, useContext, useEffect, useState, useCallback } from "react";
import NetInfo from "@react-native-community/netinfo";
import { supabase } from "../supabaseClient";
import {
  setupLocalDatabase,
  getLocalProfile,
  upsertLocalProfile,
  updateLocalDisplayName,
} from "../localdb/localdb";
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

  // 2) NetInfo for connectivity
  useEffect(() => {
    const unsubscribe = NetInfo.addEventListener((state) => {
      setIsOnline(!!state.isConnected);
    });
    return () => unsubscribe();
  }, []);

  // 3) Load local data if user logs in
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

  // 4) Real-time subscription for profiles
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
          filter: \`user_id=eq.\${user.id}\`,
        },
        async () => {
          if (isOnline) {
            await fetchRemoteProfile();
          }
        }
      )
      .subscribe();

    return () => {
      supabase.removeChannel(channel);
    };
  }, [user, isOnline]);

  async function fetchRemoteProfile() {
    if (!user?.id || !isOnline) return;
    try {
      setSyncing(true);
      const { data, error } = await supabase
        .from("profiles")
        .select("*")
        .eq("user_id", user.id)
        .single();
      if (!error && data) {
        const updatedAt = new Date().toISOString();
        // We also store 'role' locally
        await upsertLocalProfile(data.user_id, data.display_name ?? "", data.role ?? "user", updatedAt);
        const localData = await getLocalProfile(data.user_id);
        setLocalProfile(localData);
      }
    } catch (err) {
      console.log("fetchRemoteProfile error:", err);
    } finally {
      setSyncing(false);
    }
  }

  // 5) For local edits: store offline, push if online
  const updateLocalAndQueueSync = useCallback(
    async (displayName: string) => {
      if (!user?.id) return;
      setSyncing(true);
      try {
        await updateLocalDisplayName(user.id, displayName);
        const localData = await getLocalProfile(user.id);
        setLocalProfile(localData);

        if (isOnline) {
          const { error } = await supabase
            .from("profiles")
            .update({ display_name: displayName })
            .eq("user_id", user.id);
          if (error) {
            console.log("Error updating remote profile:", error.message);
          }
        }
      } catch (err) {
        console.log("updateLocalAndQueueSync error:", err);
      } finally {
        setSyncing(false);
      }
    },
    [user, isOnline]
  );

  return (
    <OfflineContext.Provider
      value={{ localProfile, isOnline, syncing, updateLocalAndQueueSync }}
    >
      {children}
    </OfflineContext.Provider>
  );
}

export function useOffline() {
  return useContext(OfflineContext);
}
`
  },
  {
    filePath: "app/_layout.tsx",
    content: `import { Stack } from "expo-router";
import AuthProvider from "../context/auth"; // picks native or web
import { OfflineProvider } from "../context/offline";
import { StatusBar } from "expo-status-bar";

export default function RootLayout() {
  return (
    <AuthProvider>
      <OfflineProvider>
        <Stack screenOptions={{ headerShown: false }} />
        <StatusBar style="auto" />
      </OfflineProvider>
    </AuthProvider>
  );
}
`
  },
  {
    filePath: "app/index.tsx",
    content: `import { Redirect } from "expo-router";
import { useAuth } from "../context/auth";

export default function IndexScreen() {
  const { user } = useAuth();
  return user ? (
    <Redirect href="/(protected)/profile" />
  ) : (
    <Redirect href="/(auth)/sign-in" />
  );
}
`
  },
  {
    filePath: "app/(auth)/_layout.tsx",
    content: `import { Stack } from "expo-router";

export default function AuthLayout() {
  return <Stack screenOptions={{ headerShown: true, headerTitle: "Auth Flow" }} />;
}
`
  },
  {
    filePath: "app/(auth)/sign-in.tsx",
    content: `import { View, Text, Button, TextInput, StyleSheet } from "react-native";
import { useState, useEffect } from "react";
import { Link, useRouter } from "expo-router";
import { useAuth } from "../../context/auth";

export default function SignInScreen() {
  const { user, signIn, loading, error } = useAuth();
  const router = useRouter();

  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [localError, setLocalError] = useState("");

  useEffect(() => {
    if (user) router.replace("(protected)/profile");
  }, [user]);

  async function handleSignIn() {
    if (!/\\S+@\\S+\\.\\S+/.test(email)) {
      setLocalError("Invalid email");
      return;
    }
    if (password.length < 8) {
      setLocalError("Must be at least 8 chars");
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
        onChangeText={setEmail}
        value={email}
        autoCapitalize="none"
      />
      <TextInput
        style={styles.input}
        placeholder="Password"
        onChangeText={setPassword}
        value={password}
        secureTextEntry
      />

      <Button title={loading ? "Signing In..." : "Sign In"} onPress={handleSignIn} />

      <Link href="/(auth)/sign-up" style={styles.link}>
        Dont have an account? Sign Up
      </Link>
    </View>
  );
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
`
  },
  {
    filePath: "app/(auth)/sign-up.tsx",
    content: `import { View, Text, Button, TextInput, StyleSheet } from "react-native";
import { useState, useEffect } from "react";
import { Link, useRouter } from "expo-router";
import { useAuth } from "../../context/auth";

export default function SignUpScreen() {
  const { user, signUp, loading, error } = useAuth();
  const router = useRouter();

  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [confirmPass, setConfirmPass] = useState("");
  const [localError, setLocalError] = useState("");

  useEffect(() => {
    if (user) router.replace("(protected)/profile");
  }, [user]);

  async function handleSignUp() {
    if (!/\\S+@\\S+\\.\\S+/.test(email)) {
      setLocalError("Invalid email");
      return;
    }
    if (password.length < 8) {
      setLocalError("At least 8 chars");
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
        onChangeText={setEmail}
        value={email}
        autoCapitalize="none"
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
`
  },
  {
    filePath: "app/(protected)/_layout.tsx",
    content: `import { Stack, useRouter, useRootNavigationState } from "expo-router";
import { useEffect } from "react";
import { useAuth } from "../../context/auth";

export default function ProtectedLayout() {
  const router = useRouter();
  const navState = useRootNavigationState();
  const { user } = useAuth();

  if (!navState?.key) return null; // Wait for router

  useEffect(() => {
    if (!user) {
      router.replace("(auth)/sign-in");
    }
  }, [user]);

  return (
    <Stack screenOptions={{ headerShown: true, headerTitle: "Protected" }} />
  );
}
`
  },
  {
    filePath: "app/(protected)/profile.tsx",
    content: `import { View, Text, Button, StyleSheet } from "react-native";
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
        {isOnline ? "Online" : "Offline"} — Hello, {user.email}
      </Text>
      {localProfile ? (
        <>
          <Text>Display: {localProfile.display_name || "(none)"} </Text>
          <Text>Role: {localProfile.role}</Text>
          <Text>Updated: {localProfile.updated_at || "N/A"}</Text>
        </>
      ) : (
        <Text>Loading local profile...</Text>
      )}

      <View style={{ marginTop: 20 }}>
        <Button
          title="Edit Profile"
          onPress={() => router.push("(protected)/edit-profile")}
        />
      </View>

      {/* If you're admin, show link to admin dashboard */}
      {localProfile?.role === "admin" && (
        <View style={{ marginTop: 20 }}>
          <Button
            title="Go to Admin Dashboard"
            onPress={() => router.push("(admin)")}
          />
        </View>
      )}

      <View style={{ marginTop: 20 }}>
        <Button title="Sign Out" onPress={handleSignOut} />
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { paddingHorizontal: 20, paddingTop: 40 },
  heading: { fontSize: 18, fontWeight: "bold", marginBottom: 10 },
});
`
  },
  {
    filePath: "app/(protected)/edit-profile.tsx",
    content: `import { View, Text, TextInput, Button, StyleSheet } from "react-native";
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
`
  },
  {
    filePath: "app/(admin)/_layout.tsx",
    content: `import { Stack, useRouter, useRootNavigationState } from "expo-router";
import { useEffect } from "react";
import { useAuth } from "../../context/auth";
import { useOffline } from "../../context/offline";

export default function AdminLayout() {
  const router = useRouter();
  const navigationState = useRootNavigationState();
  const { user } = useAuth();
  const { localProfile } = useOffline();

  if (!navigationState?.key) return null; // Wait for router

  useEffect(() => {
    // If not logged in or user is not admin, go to normal profile
    if (!user) {
      router.replace("(auth)/sign-in");
      return;
    }
    if (localProfile?.role !== "admin") {
      router.replace("(protected)/profile");
    }
  }, [user, localProfile]);

  return <Stack screenOptions={{ headerShown: true, headerTitle: "Admin" }} />;
}
`
  },
  {
    filePath: "app/(admin)/index.tsx",
    content: `import { View, Text, Button, StyleSheet } from "react-native";
import { useRouter } from "expo-router";

export default function AdminHome() {
  const router = useRouter();

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Admin Dashboard</Text>

      <Button
        title="Manage Users"
        onPress={() => router.push("/(admin)/users")}
      />
      <Button
        title="Moderate Content"
        onPress={() => router.push("/(admin)/content")}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { padding: 20, marginTop: 40 },
  title: { fontSize: 24, fontWeight: "bold", marginBottom: 20 },
});
`
  },
  {
    filePath: "app/(admin)/users.tsx",
    content: `import { View, Text, StyleSheet, FlatList, Button } from "react-native";
import { useState, useEffect } from "react";
import { supabase } from "../../supabaseClient";
import { useRouter } from "expo-router";

interface Profile {
  user_id: string;
  display_name: string;
  role: string;
}

export default function ManageUsers() {
  const router = useRouter();
  const [profiles, setProfiles] = useState<Profile[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchProfiles();
  }, []);

  async function fetchProfiles() {
    setLoading(true);
    const { data, error } = await supabase.from("profiles").select("*");
    if (error) {
      console.log("Error fetching profiles:", error.message);
      setLoading(false);
      return;
    }
    setProfiles(data as Profile[]);
    setLoading(false);
  }

  async function promoteToAdmin(userId: string) {
    const { error } = await supabase
      .from("profiles")
      .update({ role: "admin" })
      .eq("user_id", userId);
    if (error) {
      console.log("Error promoting user:", error.message);
    } else {
      fetchProfiles();
    }
  }

  async function demoteToUser(userId: string) {
    const { error } = await supabase
      .from("profiles")
      .update({ role: "user" })
      .eq("user_id", userId);
    if (error) {
      console.log("Error demoting user:", error.message);
    } else {
      fetchProfiles();
    }
  }

  return (
    <View style={styles.container}>
      <Text style={styles.heading}>Manage Users</Text>
      {loading && <Text>Loading...</Text>}

      <FlatList
        data={profiles}
        keyExtractor={(item) => item.user_id}
        renderItem={({ item }) => {
          return (
            <View style={styles.userRow}>
              <View style={{ flex: 1 }}>
                <Text>{item.display_name || "(no name)"}</Text>
                <Text style={styles.role}>{item.role}</Text>
                <Text style={styles.userId}>{item.user_id}</Text>
              </View>
              <View style={styles.buttons}>
                {item.role === "admin" ? (
                  <Button
                    title="Demote"
                    onPress={() => demoteToUser(item.user_id)}
                  />
                ) : (
                  <Button
                    title="Promote"
                    onPress={() => promoteToAdmin(item.user_id)}
                  />
                )}
              </View>
            </View>
          );
        }}
      />

      <View style={{ marginTop: 20 }}>
        <Button title="Back to Admin Home" onPress={() => router.push("/(admin)")} />
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { padding: 20, marginTop: 40 },
  heading: { fontSize: 20, fontWeight: "bold", marginBottom: 10 },
  userRow: {
    flexDirection: "row",
    alignItems: "center",
    marginBottom: 10,
    borderBottomWidth: 1,
    borderBottomColor: "#ccc",
    paddingBottom: 8,
  },
  role: { color: "#666" },
  userId: { color: "#999", fontSize: 12 },
  buttons: { flexDirection: "row", gap: 10 },
});
`
  },
  {
    filePath: "app/(admin)/content.tsx",
    content: `import { View, Text, StyleSheet, FlatList, Button } from "react-native";
import { useState, useEffect } from "react";
import { supabase } from "../../supabaseClient";
import { useRouter } from "expo-router";

interface Post {
  id: string;
  author_id: string;
  title: string;
  body: string;
  status: string;
}

export default function ContentModeration() {
  const router = useRouter();
  const [posts, setPosts] = useState<Post[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchPosts();
  }, []);

  async function fetchPosts() {
    setLoading(true);
    const { data, error } = await supabase.from("posts").select("*");
    if (error) {
      console.log("Error fetching posts:", error.message);
      setLoading(false);
      return;
    }
    setPosts(data as Post[]);
    setLoading(false);
  }

  async function setStatus(id: string, status: string) {
    const { error } = await supabase
      .from("posts")
      .update({ status })
      .eq("id", id);
    if (error) {
      console.log("Error updating post:", error.message);
    } else {
      fetchPosts();
    }
  }

  return (
    <View style={styles.container}>
      <Text style={styles.heading}>Content Moderation</Text>
      {loading && <Text>Loading...</Text>}

      <FlatList
        data={posts}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => {
          return (
            <View style={styles.postRow}>
              <View style={{ flex: 1 }}>
                <Text style={styles.title}>{item.title}</Text>
                <Text>{item.body}</Text>
                <Text style={styles.status}>Status: {item.status}</Text>
              </View>
              <View style={styles.buttons}>
                <Button
                  title="Publish"
                  onPress={() => setStatus(item.id, "published")}
                />
                <Button
                  title="Draft"
                  onPress={() => setStatus(item.id, "draft")}
                />
              </View>
            </View>
          );
        }}
      />

      <View style={{ marginTop: 20 }}>
        <Button title="Back to Admin Home" onPress={() => router.push("/(admin)")} />
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { padding: 20, marginTop: 40 },
  heading: { fontSize: 20, fontWeight: "bold", marginBottom: 10 },
  postRow: {
    borderBottomColor: "#ccc",
    borderBottomWidth: 1,
    marginBottom: 10,
    paddingBottom: 8,
  },
  title: { fontWeight: "bold" },
  status: { color: "#666" },
  buttons: { flexDirection: "row", gap: 10, marginTop: 5 },
});
`
  }
];

(async function main() {
  console.log("=== ScriptHammer-Admin Scaffold ===");
  console.log("This script overwrites/creates all files from the integrated admin tutorial.\n");

  const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout,
  });

  rl.question("Proceed with file creation/overwrites? (y/N) ", (answer) => {
    rl.close();
    if (answer.toLowerCase() !== "y") {
      console.log("Aborted.");
      process.exit(0);
    }

    function writeFileRecursive(filePath, content) {
      const dir = path.dirname(filePath);
      if (!fs.existsSync(dir)) {
        fs.mkdirSync(dir, { recursive: true });
      }
      fs.writeFileSync(filePath, content, "utf8");
      console.log("Created/updated: " + filePath);
    }

    try {
      FILES.forEach(({ filePath, content }) => {
        const outPath = path.join(process.cwd(), filePath);
        writeFileRecursive(outPath, content);
      });

      console.log("\nAll files created or updated successfully!");
      console.log("Run 'npx expo start --clear' to test your Admin Dashboard now!");
    } catch (err) {
      console.error("Error scaffolding files:", err);
      process.exit(1);
    }
  });
})();
```

Then:

```bash
npm run scaffold-ScriptHammer-Admin
```
Confirm with “y” to create all files. Perfect for skipping manual steps.

---

## 20) Next Steps

You now have:

1. **Dexie + localStorage** on web, **SQLite + SecureStore** on native.  
2. **First user** to sign up becomes admin, subsequent (and dummy) users are normal.  
3. A **fully integrated** `(admin)` panel, no separate extension.  
4. **One** scaffolding script to generate all code at once.

From here:

- Expand your admin to show analytics or other tables.  
- Add [Supabase Storage](https://supabase.com/docs/guides/storage) for file uploads.  
- Implement push notifications or advanced roles beyond `'admin'/'user'`.  
- Finally, move on to production with Expo EAS, environment variables, logging, etc.

No separate “later” step needed—**the admin is already built in**. Enjoy building ScriptHammer!
