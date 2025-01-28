Below is the **original tutorial** with **two** best-practice changes:

1. **Auto-insert** into `profiles` after successful sign-up, so every new user has a matching row.  
2. Store **Supabase connection variables** in a **`.env.local`** file (using `EXPO_PUBLIC_` prefixes) rather than hardcoding them.

Everything else remains the same. This preserves the original structure but fixes the two biggest pain points—no empty `profiles` table and no exposed keys.

---

# Single, Complete Tutorial (With `.env.local` & Auto-Inserted Profiles)

By the end, you’ll have:

1. **Sign-up & sign-in** with real Supabase credentials.  
2. **Secure** token storage using **Expo SecureStore**.  
3. **Protected** routes in Expo Router (`(protected)`) ensuring only logged-in users can access certain screens.  
4. A **`profiles`** table in Supabase (with Row Level Security) so each user can only see/update **their own** row.  
5. A **Profile** screen with the user’s display name + “Edit Profile” flow to update that data.  
6. **Auto-created** `profiles` rows on sign-up & environment variables in `.env.local`.

---

## 1) Set Up Your Expo Project

```bash
npx create-expo-app rn_SupabaseDemo
cd rn_SupabaseDemo
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

> `expo-secure-store` provides secure session storage on iOS/Android.  
> `dotenv` allows us to use `.env.local` variables (prefixed with `EXPO_PUBLIC_`) in Expo.

---

## 3) Create & Configure `.env.local`

At the **root** of your project (same level as `package.json`), create a file named **`.env.local`**:

```bash
EXPO_PUBLIC_SUPABASE_URL=https://cjzysiuadvzgrgcgrvct.supabase.co
EXPO_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

Replace with **your** real Supabase Project URL & anon key. If you don’t want to expose them in a public repo:

```bash
# .gitignore
.env.local
```

Because these variables start with `EXPO_PUBLIC_`, Expo automatically includes them at build time (`process.env.EXPO_PUBLIC_...`).

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

1. Log in to [Supabase](https://app.supabase.com/).  
2. **Project Settings** -> **API**: Copy your **Project URL** and **anon key**.  
3. Create a `profiles` table or confirm it exists:

### Create or Confirm `profiles` Table

In your Supabase project, run:

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

This ensures each logged-in user can only see/update their own row in `profiles`.

---

## 6) Code: `supabaseClient.ts` (Connecting to Supabase via `.env.local`)

**File: `supabaseClient.ts`**

```ts
// supabaseClient.ts
import { createClient } from "@supabase/supabase-js";

// We rely on .env.local with "EXPO_PUBLIC_"
const SUPABASE_URL = process.env.EXPO_PUBLIC_SUPABASE_URL!;
const SUPABASE_ANON_KEY = process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY!;

export const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
```

> Note the exclamation marks (`!`)—telling TypeScript “we’re sure these exist.”  

---

## 7) Code: `app/_layout.tsx` (Single Top-Level Layout)

**File: `app/_layout.tsx`**

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

## 8) Code: `context/auth.tsx` (Auth Context with SecureStore + **Profiles Insert**)

> **Key Improvement**: We **insert** a row into `profiles` for each new user in `signUp`.

**File: `context/auth.tsx`**

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

  // Sign Up => also insert row in 'profiles'
  async function signUp(email: string, password: string) {
    setError(null);
    setLoading(true);
    try {
      // 1) Create user in supabase.auth.users
      const { data: signUpData, error: signUpError } = await supabase.auth.signUp({
        email,
        password,
      });
      if (signUpError) throw new Error(signUpError.message);

      // 2) If user object is returned, insert a 'profiles' row
      if (signUpData?.user) {
        const userId = signUpData.user.id;
        const { error: profileError } = await supabase
          .from("profiles")
          .insert({ user_id: userId, display_name: "" });
        if (profileError) throw new Error(profileError.message);
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

### **File**: `app/(auth)/sign-in.tsx`

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
    // if signIn successful, user is set by onAuthStateChange
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

### **File**: `app/(auth)/sign-up.tsx`

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
    // if signUp successful, user is set by onAuthStateChange, row is inserted in 'profiles'
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

  // If the router isn't ready, don't do anything
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
      <Text style={{ fontSize: 20, marginBottom: 10 }}>Welcome, {user.email}!</Text>

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

If you want `/` to redirect to `(auth)/sign-in` or `(protected)/profile`:

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

Open in a simulator or device:

1. **Sign Up**. Your `.env.local` credentials let you talk to your Supabase project. If the project requires email confirmation, check your inbox.  
2. **Auto-Created Row**: A row in `profiles` is inserted for each new user, so you’ll never have an empty table.  
3. **Sign In**: Lands on `(protected)/profile`.  
4. **Display Name**: Edit in `(protected)/edit-profile`.  
5. **RLS** ensures each user only sees/updates their own row.  
6. **Sign Out**—clears session from SecureStore or localStorage (on web).

---

## 13) Recap & Next Steps

You now have:

- Real Supabase Auth, storing sessions in **Expo SecureStore** and restricting `(protected)` routes in Expo Router.  
- A `profiles` table with Row Level Security.  
- **Auto-insert** in `profiles` after sign-up, so it’s never empty.  
- Connection credentials in `.env.local` with `EXPO_PUBLIC_`—no more hardcoding in code.  

### Next Steps

1. **DB Trigger**: Instead of inserting `profiles` in client code, use a [Supabase DB Trigger](https://supabase.com/docs/guides/auth/managing-user-data#database-triggers) for even cleaner separation.  
2. **Avatars**: Use Supabase Storage to store profile pictures, referencing the image URL in `profiles.avatar_url`.  
3. **Real-Time**: Subscribe to changes with `supabase.channel(...)` or `supabase.on(...)`.  
4. **Social Features**: Create a “posts” table with RLS, let users post and interact.

By incorporating `.env.local` and the auto-insert snippet in `signUp`, you’ll avoid any “(none)” or missing row issues and keep your credentials out of your code. Enjoy!
