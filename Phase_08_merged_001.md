# ScriptHammer – Dexie on Web, SQLite + SecureStore on Native (Complete Tutorial + Script)

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

> **Note:**  
> • `expo-secure-store` is used only on native (iOS/Android).  
> • `expo-sqlite` is used only on native for the local DB.  
> • `dexie` is used only on Web.

---

## 3) Set Up `.env.local`

At your **project root**:

```bash
EXPO_PUBLIC_SUPABASE_URL=https://YOUR-PROJECT.supabase.co
EXPO_PUBLIC_SUPABASE_ANON_KEY=YOUR-ANON-KEY
```

Also, add it to your `.gitignore`:

```bash
# .gitignore
.env.local
```

---

## 4) Supabase Setup

### Create or Confirm `profiles` Table  
This table now has a new **role** column. The column defaults to `"user"`, but the very first account inserted into the table will be set to `"admin"` by our trigger.

```sql
create table if not exists profiles (
  id uuid primary key default uuid_generate_v4(),
  user_id uuid references auth.users not null,
  display_name text,
  role text default 'user',
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

### DB Trigger for Auto-Inserting `profiles`

This trigger function automatically inserts a profile record when a new auth user is created. If no profile exists yet, the new account is set to `"admin"`; otherwise it defaults to `"user"`. (Dummy account insertion has been removed here to avoid foreign key errors.)

```sql
create or replace function handle_new_user()
returns trigger as $$
declare
  new_role text;
begin
  -- Determine role: if no profiles exist, assign admin; else default to 'user'
  if (select count(*) from public.profiles) = 0 then
    new_role := 'admin';
  else
    new_role := 'user';
  end if;

  insert into public.profiles (user_id, display_name, role)
  values (new.id, '', new_role);

  return new;
end;
$$ language plpgsql security definer;

create trigger on_auth_user_created
after insert on auth.users
for each row
execute procedure handle_new_user();
```

### Separate SQL Command for Dummy Accounts  
(Optional) Run this SQL command manually (in the Supabase SQL Editor) after you have a valid admin user to seed dummy profiles:

```sql
insert into public.profiles (user_id, display_name, role)
values (gen_random_uuid(), 'Jane', 'user'),
       (gen_random_uuid(), 'Jon', 'user'),
       (gen_random_uuid(), 'James Doe', 'user');
```

> **Note:** Ensure the `pgcrypto` extension is enabled.

### Enable Realtime for `profiles`  
In the Supabase UI, open the **Table Editor** for the `profiles` table and enable **Realtime**.

---

## 5) File & Folder Structure (Manual Creation)

Create the following folders and files (this now includes the admin panel):

```bash
# Context and local DB files
mkdir -p context
touch context/offline.tsx
touch context/auth.native.tsx
touch context/auth.web.tsx

mkdir -p localdb
touch localdb/localdb.native.ts
touch localdb/localdb.web.ts

# Auth and protected folders
mkdir -p app/\(auth\)
touch app/\(auth\)/_layout.tsx
touch app/\(auth\)/sign-in.tsx
touch app/\(auth\)/sign-up.tsx

mkdir -p app/\(protected\)
touch app/\(protected\)/_layout.tsx
touch app/\(protected\)/profile.tsx
touch app/\(protected\)/edit-profile.tsx

# Admin Dashboard folder
mkdir -p app/\(admin\)
touch app/\(admin\)/_layout.tsx
touch app/\(admin\)/dashboard.tsx
touch app/\(admin\)/user-management.tsx
touch app/\(admin\)/content-moderation.tsx
# Not Authorized page for restricted access
touch app/not-authorized.tsx

# Other files
touch supabaseClient.ts
touch app/_layout.tsx
touch app/index.tsx
touch .env.local
code .
```

If you prefer not to create files manually, jump to [Step 19](#19-scaffolding-script-generates-all-files).

---

## 6) Code: `localdb.native.ts` (Expo SQLite)

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
        (_, error) => {
          console.error("Error creating local_profiles table:", error);
          reject(error);
          return false;
        }
      );
    });
  });
}

export async function upsertLocalProfile(
  userId: string,
  display_name: string,
  updated_at: string
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
        [userId, display_name, updated_at],
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

export async function updateLocalDisplayName(userId: string, display_name: string) {
  const now = new Date().toISOString();
  return new Promise<void>((resolve, reject) => {
    db.transaction((tx) => {
      tx.executeSql(
        `UPDATE local_profiles SET display_name=?, updated_at=? WHERE user_id=?`,
        [display_name, now, userId],
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

const db = new Dexie("ScriptHammerWebDB");
db.version(1).stores({
  local_profiles: "user_id, display_name, updated_at",
});

export async function setupLocalDatabase() {
  console.log("Dexie is ready on web for offline DB");
}

export async function upsertLocalProfile(
  userId: string,
  display_name: string,
  updated_at: string
) {
  await db.table("local_profiles").put({
    user_id: userId,
    display_name,
    updated_at,
  });
}

export async function getLocalProfile(userId: string) {
  const row = await db.table("local_profiles").get(userId);
  return row || null;
}

export async function updateLocalDisplayName(userId: string, display_name: string) {
  const now = new Date().toISOString();
  await db.table("local_profiles").update(userId, {
    display_name,
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
import { Platform } from "react-native";

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

// SecureStore-based get/set
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

async function setItem(key: string, value: string) {
  window.localStorage.setItem(key, value);
}
async function getItem(key: string) {
  return window.localStorage.getItem(key) ?? null;
}
async function deleteItem(key: string) {
  window.localStorage.removeItem(key);
}

export default function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<any>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

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

This version now ensures that if no profile record is found for the logged‑in user, a default record is inserted. (Also note that if your display name is blank, it remains blank.)

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

  // 4) Real-time subscription
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
      let { data, error } = await supabase
        .from("profiles")
        .select("*")
        .eq("user_id", user.id)
        .single();
      if (error) {
        // If no profile record exists, insert a default record
        if (error.message.includes("No rows found")) {
          const { data: insertedData, error: insertError } = await supabase
            .from("profiles")
            .insert({ user_id: user.id, display_name: "", role: "user" })
            .select();
          if (insertError) {
            console.error("Error inserting profile:", insertError.message);
          } else {
            data = insertedData[0];
          }
        } else {
          console.error("Error fetching profile:", error.message);
        }
      }
      if (data) {
        const updatedAt = new Date().toISOString();
        await upsertLocalProfile(data.user_id, data.display_name ?? "", updatedAt);
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

> **Important Update:**  
> To avoid the “Attempted to navigate before mounting the Root Layout component” error, the root layout now renders a `<Slot />` inside the `<Stack>`. This ensures that navigation can occur only after the layout is fully mounted.

```tsx
// app/_layout.tsx
import { Stack, Slot } from "expo-router";
import AuthProvider from "../context/auth"; // picks native or web
import { OfflineProvider } from "../context/offline";
import { StatusBar } from "expo-status-bar";

export default function RootLayout() {
  return (
    <AuthProvider>
      <OfflineProvider>
        <Stack screenOptions={{ headerShown: false }}>
          <Slot />
        </Stack>
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
        <View>
          <Text>Display: {localProfile.display_name || "(none)"} </Text>
          <Text>Updated: {localProfile.updated_at || "N/A"}</Text>
        </View>
      ) : (
        <Text>Loading local profile...</Text>
      )}
      <View style={{ marginTop: 20 }}>
        <Button title="Edit Profile" onPress={() => router.push("(protected)/edit-profile")} />
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
    // If display_name is null, leave it blank.
    setDisplayName(localProfile.display_name ?? "");
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
      <Text style={styles.label}>Display Name (optional):</Text>
      <TextInput
        style={styles.input}
        value={displayName}
        onChangeText={setDisplayName}
        placeholder="Leave blank to remain empty"
      />
      <Button title={syncing ? "Saving..." : "Save Changes"} onPress={handleSave} disabled={syncing} />
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

## 16) Code: `(admin)` Folder (Admin Dashboard)

### `(admin)/_layout.tsx`

```tsx
// app/(admin)/_layout.tsx
import React, { useEffect, useState } from "react";
import { Stack, useRouter } from "expo-router";
import { useAuth } from "../../context/auth";
import { supabase } from "../../supabaseClient";
import { View, ActivityIndicator } from "react-native";

export default function AdminLayout() {
  const { user } = useAuth();
  const router = useRouter();
  const [isAdmin, setIsAdmin] = useState<boolean | null>(null);

  useEffect(() => {
    if (!user) {
      router.replace("/(auth)/sign-in");
      return;
    }
    async function checkAdmin() {
      const { data, error } = await supabase
        .from("profiles")
        .select("role")
        .eq("user_id", user.id)
        .single();
      if (error || !data || data.role !== 'admin') {
        router.replace("/not-authorized");
      } else {
        setIsAdmin(true);
      }
    }
    checkAdmin();
  }, [user]);

  if (isAdmin === null) {
    return (
      <View style={{ flex: 1, justifyContent: "center", alignItems: "center" }}>
        <ActivityIndicator size="large" />
      </View>
    );
  }

  return (
    <Stack screenOptions={{ headerShown: true, headerTitle: "Admin Panel" }} />
  );
}
```

### `(admin)/dashboard.tsx`

```tsx
// app/(admin)/dashboard.tsx
import React from "react";
import { View, Text, Button, StyleSheet } from "react-native";
import { Link, useRouter } from "expo-router";
import { useAuth } from "../../context/auth";

export default function AdminDashboard() {
  const router = useRouter();
  const { user } = useAuth();

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Admin Dashboard</Text>
      <Text>Welcome, {user?.email}</Text>
      <View style={styles.buttonContainer}>
        <Button title="User Management" onPress={() => router.push("/(admin)/user-management")} />
      </View>
      <View style={styles.buttonContainer}>
        <Button title="Content Moderation" onPress={() => router.push("/(admin)/content-moderation")} />
      </View>
      <View style={styles.buttonContainer}>
        <Link href="/" style={styles.link}>
          Back to Home
        </Link>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 20, justifyContent: "center", alignItems: "center" },
  title: { fontSize: 24, fontWeight: "bold", marginBottom: 20 },
  buttonContainer: { marginVertical: 10, width: "80%" },
  link: { marginTop: 20, color: "blue" },
});
```

### `(admin)/user-management.tsx`

```tsx
// app/(admin)/user-management.tsx
import React, { useEffect, useState } from "react";
import { View, Text, Button, FlatList, StyleSheet, ActivityIndicator, Alert } from "react-native";
import { supabase } from "../../supabaseClient";

interface UserProfile {
  user_id: string;
  display_name: string;
  role: string;
  created_at: string;
}

export default function UserManagement() {
  const [profiles, setProfiles] = useState<UserProfile[]>([]);
  const [loading, setLoading] = useState<boolean>(true);

  useEffect(() => {
    fetchProfiles();
  }, []);

  async function fetchProfiles() {
    setLoading(true);
    const { data, error } = await supabase.from("profiles").select("*");
    if (error) {
      Alert.alert("Error", error.message);
    } else {
      setProfiles(data as UserProfile[]);
    }
    setLoading(false);
  }

  async function toggleRole(userId: string, currentRole: string) {
    const newRole = currentRole === 'admin' ? 'user' : 'admin';
    const { error } = await supabase
      .from("profiles")
      .update({ role: newRole })
      .eq("user_id", userId);
    if (error) {
      Alert.alert("Error", error.message);
    } else {
      Alert.alert("Success", "User role updated.");
      fetchProfiles();
    }
  }

  if (loading) {
    return (
      <View style={styles.center}>
        <ActivityIndicator size="large" />
      </View>
    );
  }

  const renderItem = ({ item }: { item: UserProfile }) => (
    <View style={styles.item}>
      <Text style={styles.itemText}>{item.display_name || item.user_id}</Text>
      <Text>Role: {item.role}</Text>
      <Button title={\`Make \${item.role === 'admin' ? 'User' : 'Admin'}\`} onPress={() => toggleRole(item.user_id, item.role)} />
    </View>
  );

  return (
    <View style={styles.container}>
      <Text style={styles.title}>User Management</Text>
      <FlatList
        data={profiles}
        keyExtractor={(item) => item.user_id}
        renderItem={renderItem}
        contentContainerStyle={styles.list}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 20 },
  center: { flex: 1, justifyContent: "center", alignItems: "center" },
  title: { fontSize: 24, fontWeight: "bold", marginBottom: 20 },
  list: { paddingBottom: 20 },
  item: { padding: 15, marginBottom: 10, backgroundColor: "#f2f2f2", borderRadius: 5 },
  itemText: { fontSize: 18, marginBottom: 5 },
});
```

### `(admin)/content-moderation.tsx`

```tsx
// app/(admin)/content-moderation.tsx
import React, { useEffect, useState } from "react";
import { View, Text, Button, FlatList, StyleSheet, ActivityIndicator, Alert } from "react-native";
import { supabase } from "../../supabaseClient";

interface Post {
  id: number;
  user_id: string;
  content: string;
  is_flagged: boolean;
  created_at: string;
}

export default function ContentModeration() {
  const [posts, setPosts] = useState<Post[]>([]);
  const [loading, setLoading] = useState<boolean>(true);

  useEffect(() => {
    fetchPosts();
  }, []);

  async function fetchPosts() {
    setLoading(true);
    const { data, error } = await supabase.from("posts").select("*").order("created_at", { ascending: false });
    if (error) {
      Alert.alert("Error", error.message);
    } else {
      setPosts(data as Post[]);
    }
    setLoading(false);
  }

  async function toggleFlag(postId: number, currentFlag: boolean) {
    const { error } = await supabase
      .from("posts")
      .update({ is_flagged: !currentFlag })
      .eq("id", postId);
    if (error) {
      Alert.alert("Error", error.message);
    } else {
      Alert.alert("Success", "Post flag status updated.");
      fetchPosts();
    }
  }

  async function deletePost(postId: number) {
    const { error } = await supabase.from("posts").delete().eq("id", postId);
    if (error) {
      Alert.alert("Error", error.message);
    } else {
      Alert.alert("Deleted", "Post deleted successfully.");
      fetchPosts();
    }
  }

  if (loading) {
    return (
      <View style={styles.center}>
        <ActivityIndicator size="large" />
      </View>
    );
  }

  const renderItem = ({ item }: { item: Post }) => (
    <View style={styles.item}>
      <Text style={styles.itemText}>Post ID: {item.id}</Text>
      <Text>User ID: {item.user_id}</Text>
      <Text>{item.content}</Text>
      <Text>Flagged: {item.is_flagged ? "Yes" : "No"}</Text>
      <View style={styles.buttonRow}>
        <Button title={item.is_flagged ? "Unflag" : "Flag"} onPress={() => toggleFlag(item.id, item.is_flagged)} />
        <View style={{ width: 10 }} />
        <Button title="Delete" onPress={() => deletePost(item.id)} color="red" />
      </View>
    </View>
  );

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Content Moderation</Text>
      <FlatList
        data={posts}
        keyExtractor={(item) => item.id.toString()}
        renderItem={renderItem}
        contentContainerStyle={styles.list}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 20 },
  center: { flex: 1, justifyContent: "center", alignItems: "center" },
  title: { fontSize: 24, fontWeight: "bold", marginBottom: 20 },
  list: { paddingBottom: 20 },
  item: { padding: 15, marginBottom: 10, backgroundColor: "#e6e6e6", borderRadius: 5 },
  itemText: { fontSize: 18, marginBottom: 5 },
  buttonRow: { flexDirection: "row", marginTop: 10 },
});
```

### `not-authorized.tsx`

```tsx
// app/not-authorized.tsx
import React from "react";
import { View, Text, StyleSheet, Button } from "react-native";
import { useRouter } from "expo-router";

export default function NotAuthorized() {
  const router = useRouter();

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Not Authorized</Text>
      <Text>You do not have permission to access this page.</Text>
      <Button title="Go Back" onPress={() => router.back()} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: "center", alignItems: "center", padding: 20 },
  title: { fontSize: 24, fontWeight: "bold", marginBottom: 20 },
});
```

---

## 17) Run & Test

Start your project with:

```bash
npx expo start --clear
```

> **Testing Notes:**  
> • On iOS/Android the code uses `auth.native.tsx` and `localdb.native.ts` (SecureStore and SQLite).  
> • On Web the code uses `auth.web.tsx` and `localdb.web.ts` (localStorage and Dexie).  
> • When you sign up, the very first user is automatically set to `"admin"` (via the Supabase trigger).  
> • If no profile record exists, the client (in `fetchRemoteProfile()`) inserts a default profile record (with a blank display name) so that your profile will load instead of “Loading local profile…” forever.  
> • To test admin functionality, navigate to `/admin/dashboard`. (If you’d like an admin button on your profile page, you can add one conditionally based on a separate Supabase call.)  
> 
> **Important:**  
> • The Root Layout now properly renders a `<Slot />` inside a `<Stack>`, preventing the navigation error.

---

## 18) Troubleshooting SecureStore or SQLite Issues

1. If on Web your project still imports `expo-secure-store`, ensure you have named your files correctly:
   - Use **auth.native.tsx** for iOS/Android  
   - Use **auth.web.tsx** for Web  
2. If you did not install them, run:
   ```bash
   npx expo install expo-secure-store expo-sqlite
   ```
3. In a bare or prebuild workflow, run:
   ```bash
   npx expo prebuild && npx expo run:ios
   ```
4. Consider upgrading your Expo SDK if needed.

---

## 19) Scaffolding Script (Generates All Files)

If you prefer to automatically create or overwrite all files (including the admin dashboard), use the following Node script.

1. Create a `scripts/` folder:
2. Create the scaffolding script:
   ```bash
   mkdir -p scripts
   touch scripts/scaffold-ScriptHammer.js
   chmod +x scripts/scaffold-ScriptHammer.js
   code .
   ```
3. In your `package.json`, add:
   ```json
       "scaffold-ScriptHammer": "node ./scripts/scaffold-ScriptHammer.js",
   ```
4. Paste the complete script below:

```js
#!/usr/bin/env node

/**
 * scaffold-ScriptHammer.js
 *
 * A Node.js script that overwrites/creates all files from the 
 * “ScriptHammer: Dexie on Web + SQLite & SecureStore on Native (with Admin Dashboard)” tutorial.
 *
 * USAGE:
 *   npm run scaffold-ScriptHammer
 *
 * WARNING:
 *   - This overwrites existing files with the same paths.
 *   - Please commit or backup your work first.
 */

const fs = require("fs");
const path = require("path");
const readline = require("readline");

const FILES = [
  {
    filePath: "supabaseClient.ts",
    content: `import { createClient } from "@supabase/supabase-js";

const SUPABASE_URL = process.env.EXPO_PUBLIC_SUPABASE_URL!;
const SUPABASE_ANON_KEY = process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY!;

export const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
`
  },
  {
    filePath: "localdb/localdb.native.ts",
    content: `import * as SQLite from "expo-sqlite";

const db = SQLite.openDatabase("ScriptHammer.db");

export async function setupLocalDatabase() {
  return new Promise<void>((resolve, reject) => {
    db.transaction((tx) => {
      tx.executeSql(
        \`CREATE TABLE IF NOT EXISTS local_profiles (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          user_id TEXT UNIQUE,
          display_name TEXT,
          updated_at TEXT
        );\`,
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

export async function upsertLocalProfile(
  userId: string,
  display_name: string,
  updated_at: string
) {
  return new Promise<void>((resolve, reject) => {
    db.transaction((tx) => {
      tx.executeSql(
        \`
        INSERT INTO local_profiles (user_id, display_name, updated_at)
        VALUES (?, ?, ?)
        ON CONFLICT(user_id) DO UPDATE 
          SET display_name=excluded.display_name,
              updated_at=excluded.updated_at
        \`,
        [userId, display_name, updated_at],
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

export async function getLocalProfile(userId: string) {
  return new Promise<any>((resolve, reject) => {
    db.transaction((tx) => {
      tx.executeSql(
        \`SELECT * FROM local_profiles WHERE user_id=? LIMIT 1\`,
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

export async function updateLocalDisplayName(userId: string, display_name: string) {
  const now = new Date().toISOString();
  return new Promise<void>((resolve, reject) => {
    db.transaction((tx) => {
      tx.executeSql(
        \`UPDATE local_profiles SET display_name=?, updated_at=? WHERE user_id=?\`,
        [display_name, now, userId],
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

const db = new Dexie("ScriptHammerWebDB");
db.version(1).stores({
  local_profiles: "user_id, display_name, updated_at",
});

export async function setupLocalDatabase() {
  console.log("Dexie is ready on web for offline DB");
}

export async function upsertLocalProfile(
  userId: string,
  display_name: string,
  updated_at: string
) {
  await db.table("local_profiles").put({
    user_id: userId,
    display_name,
    updated_at,
  });
}

export async function getLocalProfile(userId: string) {
  const row = await db.table("local_profiles").get(userId);
  return row || null;
}

export async function updateLocalDisplayName(userId: string, display_name: string) {
  const now = new Date().toISOString();
  await db.table("local_profiles").update(userId, {
    display_name,
    updated_at: now,
  });
}
`
  },
  {
    filePath: "context/auth.native.tsx",
    content: `import React, { createContext, useContext, useEffect, useState } from "react";
import { supabase } from "../supabaseClient";
import { Session } from "@supabase/supabase-js";
import * as SecureStore from "expo-secure-store";
import { Platform } from "react-native";

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

// SecureStore-based get/set
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

async function setItem(key: string, value: string) {
  window.localStorage.setItem(key, value);
}
async function getItem(key: string) {
  return window.localStorage.getItem(key) ?? null;
}
async function deleteItem(key: string) {
  window.localStorage.removeItem(key);
}

export default function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<any>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

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

  useEffect(() => {
    (async () => {
      await setupLocalDatabase();
    })();
  }, []);

  useEffect(() => {
    const unsubscribe = NetInfo.addEventListener((state) => {
      setIsOnline(!!state.isConnected);
    });
    return () => unsubscribe();
  }, []);

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
      let { data, error } = await supabase
        .from("profiles")
        .select("*")
        .eq("user_id", user.id)
        .single();
      if (error) {
        if (error.message.includes("No rows found")) {
          const { data: insertedData, error: insertError } = await supabase
            .from("profiles")
            .insert({ user_id: user.id, display_name: "", role: "user" })
            .select();
          if (insertError) {
            console.error("Error inserting profile:", insertError.message);
          } else {
            data = insertedData[0];
          }
        } else {
          console.error("Error fetching profile:", error.message);
        }
      }
      if (data) {
        const updatedAt = new Date().toISOString();
        await upsertLocalProfile(data.user_id, data.display_name ?? "", updatedAt);
        const localData = await getLocalProfile(data.user_id);
        setLocalProfile(localData);
      }
    } catch (err) {
      console.log("fetchRemoteProfile error:", err);
    } finally {
      setSyncing(false);
    }
  }

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
    content: `import { Stack, Slot } from "expo-router";
import AuthProvider from "../context/auth";
import { OfflineProvider } from "../context/offline";
import { StatusBar } from "expo-status-bar";

export default function RootLayout() {
  return (
    <AuthProvider>
      <OfflineProvider>
        <Stack screenOptions={{ headerShown: false }}>
          <Slot />
        </Stack>
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
        <View>
          <Text>Display: {localProfile.display_name || "(none)"} </Text>
          <Text>Updated: {localProfile.updated_at || "N/A"}</Text>
        </View>
      ) : (
        <Text>Loading local profile...</Text>
      )}
      <View style={{ marginTop: 20 }}>
        <Button title="Edit Profile" onPress={() => router.push("(protected)/edit-profile")} />
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
    // If display_name is null, leave it blank.
    setDisplayName(localProfile.display_name ?? "");
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
      <Text style={styles.label}>Display Name (optional):</Text>
      <TextInput
        style={styles.input}
        value={displayName}
        onChangeText={setDisplayName}
        placeholder="Leave blank to remain empty"
      />
      <Button title={syncing ? "Saving..." : "Save Changes"} onPress={handleSave} disabled={syncing} />
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
    content: `import React, { useEffect, useState } from "react";
import { Stack, useRouter } from "expo-router";
import { useAuth } from "../../context/auth";
import { supabase } from "../../supabaseClient";
import { View, ActivityIndicator } from "react-native";

export default function AdminLayout() {
  const { user } = useAuth();
  const router = useRouter();
  const [isAdmin, setIsAdmin] = useState<boolean | null>(null);

  useEffect(() => {
    if (!user) {
      router.replace("/(auth)/sign-in");
      return;
    }
    async function checkAdmin() {
      const { data, error } = await supabase
        .from("profiles")
        .select("role")
        .eq("user_id", user.id)
        .single();
      if (error || !data || data.role !== 'admin') {
        router.replace("/not-authorized");
      } else {
        setIsAdmin(true);
      }
    }
    checkAdmin();
  }, [user]);

  if (isAdmin === null) {
    return (
      <View style={{ flex: 1, justifyContent: "center", alignItems: "center" }}>
        <ActivityIndicator size="large" />
      </View>
    );
  }

  return (
    <Stack screenOptions={{ headerShown: true, headerTitle: "Admin Panel" }} />
  );
}
`
  },
  {
    filePath: "app/(admin)/dashboard.tsx",
    content: `import React from "react";
import { View, Text, Button, StyleSheet } from "react-native";
import { Link, useRouter } from "expo-router";
import { useAuth } from "../../context/auth";

export default function AdminDashboard() {
  const router = useRouter();
  const { user } = useAuth();

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Admin Dashboard</Text>
      <Text>Welcome, {user?.email}</Text>
      <View style={styles.buttonContainer}>
        <Button title="User Management" onPress={() => router.push("/(admin)/user-management")} />
      </View>
      <View style={styles.buttonContainer}>
        <Button title="Content Moderation" onPress={() => router.push("/(admin)/content-moderation")} />
      </View>
      <View style={styles.buttonContainer}>
        <Link href="/" style={styles.link}>
          Back to Home
        </Link>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 20, justifyContent: "center", alignItems: "center" },
  title: { fontSize: 24, fontWeight: "bold", marginBottom: 20 },
  buttonContainer: { marginVertical: 10, width: "80%" },
  link: { marginTop: 20, color: "blue" },
});
`
  },
  {
    filePath: "app/(admin)/user-management.tsx",
    content: `import React, { useEffect, useState } from "react";
import { View, Text, Button, FlatList, StyleSheet, ActivityIndicator, Alert } from "react-native";
import { supabase } from "../../supabaseClient";

interface UserProfile {
  user_id: string;
  display_name: string;
  role: string;
  created_at: string;
}

export default function UserManagement() {
  const [profiles, setProfiles] = useState<UserProfile[]>([]);
  const [loading, setLoading] = useState<boolean>(true);

  useEffect(() => {
    fetchProfiles();
  }, []);

  async function fetchProfiles() {
    setLoading(true);
    const { data, error } = await supabase.from("profiles").select("*");
    if (error) {
      Alert.alert("Error", error.message);
    } else {
      setProfiles(data as UserProfile[]);
    }
    setLoading(false);
  }

  async function toggleRole(userId: string, currentRole: string) {
    const newRole = currentRole === 'admin' ? 'user' : 'admin';
    const { error } = await supabase
      .from("profiles")
      .update({ role: newRole })
      .eq("user_id", userId);
    if (error) {
      Alert.alert("Error", error.message);
    } else {
      Alert.alert("Success", "User role updated.");
      fetchProfiles();
    }
  }

  if (loading) {
    return (
      <View style={styles.center}>
        <ActivityIndicator size="large" />
      </View>
    );
  }

  const renderItem = ({ item }: { item: UserProfile }) => (
    <View style={styles.item}>
      <Text style={styles.itemText}>{item.display_name || item.user_id}</Text>
      <Text>Role: {item.role}</Text>
      <Button title={\`Make \${item.role === 'admin' ? 'User' : 'Admin'}\`} onPress={() => toggleRole(item.user_id, item.role)} />
    </View>
  );

  return (
    <View style={styles.container}>
      <Text style={styles.title}>User Management</Text>
      <FlatList
        data={profiles}
        keyExtractor={(item) => item.user_id}
        renderItem={renderItem}
        contentContainerStyle={styles.list}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 20 },
  center: { flex: 1, justifyContent: "center", alignItems: "center" },
  title: { fontSize: 24, fontWeight: "bold", marginBottom: 20 },
  list: { paddingBottom: 20 },
  item: { padding: 15, marginBottom: 10, backgroundColor: "#f2f2f2", borderRadius: 5 },
  itemText: { fontSize: 18, marginBottom: 5 },
});
`
  },
  {
    filePath: "app/(admin)/content-moderation.tsx",
    content: `import React, { useEffect, useState } from "react";
import { View, Text, Button, FlatList, StyleSheet, ActivityIndicator, Alert } from "react-native";
import { supabase } from "../../supabaseClient";

interface Post {
  id: number;
  user_id: string;
  content: string;
  is_flagged: boolean;
  created_at: string;
}

export default function ContentModeration() {
  const [posts, setPosts] = useState<Post[]>([]);
  const [loading, setLoading] = useState<boolean>(true);

  useEffect(() => {
    fetchPosts();
  }, []);

  async function fetchPosts() {
    setLoading(true);
    const { data, error } = await supabase.from("posts").select("*").order("created_at", { ascending: false });
    if (error) {
      Alert.alert("Error", error.message);
    } else {
      setPosts(data as Post[]);
    }
    setLoading(false);
  }

  async function toggleFlag(postId: number, currentFlag: boolean) {
    const { error } = await supabase
      .from("posts")
      .update({ is_flagged: !currentFlag })
      .eq("id", postId);
    if (error) {
      Alert.alert("Error", error.message);
    } else {
      Alert.alert("Success", "Post flag status updated.");
      fetchPosts();
    }
  }

  async function deletePost(postId: number) {
    const { error } = await supabase.from("posts").delete().eq("id", postId);
    if (error) {
      Alert.alert("Error", error.message);
    } else {
      Alert.alert("Deleted", "Post deleted successfully.");
      fetchPosts();
    }
  }

  if (loading) {
    return (
      <View style={styles.center}>
        <ActivityIndicator size="large" />
      </View>
    );
  }

  const renderItem = ({ item }: { item: Post }) => (
    <View style={styles.item}>
      <Text style={styles.itemText}>Post ID: {item.id}</Text>
      <Text>User ID: {item.user_id}</Text>
      <Text>{item.content}</Text>
      <Text>Flagged: {item.is_flagged ? "Yes" : "No"}</Text>
      <View style={styles.buttonRow}>
        <Button title={item.is_flagged ? "Unflag" : "Flag"} onPress={() => toggleFlag(item.id, item.is_flagged)} />
        <View style={{ width: 10 }} />
        <Button title="Delete" onPress={() => deletePost(item.id)} color="red" />
      </View>
    </View>
  );

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Content Moderation</Text>
      <FlatList
        data={posts}
        keyExtractor={(item) => item.id.toString()}
        renderItem={renderItem}
        contentContainerStyle={styles.list}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 20 },
  center: { flex: 1, justifyContent: "center", alignItems: "center" },
  title: { fontSize: 24, fontWeight: "bold", marginBottom: 20 },
  list: { paddingBottom: 20 },
  item: { padding: 15, marginBottom: 10, backgroundColor: "#e6e6e6", borderRadius: 5 },
  itemText: { fontSize: 18, marginBottom: 5 },
  buttonRow: { flexDirection: "row", marginTop: 10 },
});
`
  },
  {
    filePath: "app/not-authorized.tsx",
    content: `import React from "react";
import { View, Text, StyleSheet, Button } from "react-native";
import { useRouter } from "expo-router";

export default function NotAuthorized() {
  const router = useRouter();

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Not Authorized</Text>
      <Text>You do not have permission to access this page.</Text>
      <Button title="Go Back" onPress={() => router.back()} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: "center", alignItems: "center", padding: 20 },
  title: { fontSize: 24, fontWeight: "bold", marginBottom: 20 },
});
`
  },
];

(async function main() {
  console.log("=== ScriptHammer Scaffold ===");
  console.log("This script will create or overwrite the tutorial files.");

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
      console.log("All files created or updated successfully!");
      console.log("You can now run 'npx expo start --clear' to test.");
    } catch (err) {
      console.error("Error scaffolding files:", err);
      process.exit(1);
    }
  });
})();
```

**Usage:**

```bash
npm run scaffold-ScriptHammer
```

Confirm the prompt, and the script will generate (or overwrite) all files.

---

## 20) Next Steps

You now have a complete ScriptHammer codebase that includes:

- **Web:** Dexie + localStorage (with no references to `expo-secure-store`).  
- **Native:** SQLite + `expo-secure-store`.  
- Auto‑sync via NetInfo and real‑time updates from Supabase with RLS in place.  
- An integrated **Admin Dashboard (Phase 7)** that uses a new role column in the profiles table. The very first user who signs up becomes an admin, and if no profile record exists for the user the client inserts a default record (with a blank display name). You may also run the separate SQL command (Step 4, “Separate SQL Command for Dummy Accounts”) if you wish to seed dummy profiles.
- **Important:** The Root Layout now includes a `<Slot />` to ensure navigation occurs only after mounting. This fixes the navigation error you saw.
- (Optionally) To make admin navigation easier, consider adding an “Admin Dashboard” button to your profile screen if desired.

**Possible Expansions:**

- Create corresponding auth.users records for dummy accounts (if needed).
- Add additional tables (e.g., posts, comments) with advanced conflict resolution.
- Enhance role/permission management and refine the Admin Dashboard UI.
- Prepare production builds using EAS, manage environment variables more robustly, add push notifications, etc.

**Congratulations!**  
This tutorial is fully self-contained, with no missing imports or placeholders, and integrates the Admin Dashboard as an inherent part of the ScriptHammer installation. Enjoy building and iterating on your application while maintaining code quality and a unified codebase!

---

*Happy coding!*
