Understood! I will follow best practices for setting up ScriptHammer, ensuring:

- **Expo Managed Workflow** (leveraging Expo Router's file-based navigation inside `app/`).
- **A clean project structure** that avoids conflicts from putting everything inside `app/`.
- **Best practices for organizing modules**, whether inside `app/` or `src/` to keep the codebase maintainable.
- **Role-based authentication with admin dashboards** to properly implement user roles (admin, editor, moderator, etc.).
- **A step-by-step roadmap that reads like a tutorial**, starting with precise setup commands and clear explanations.
- **A more engaging tone with emojis** to keep it fun but professional!

I'll provide the full roadmap soon! 🚀

# ScriptHammer Development Roadmap 📜

Let's break the journey into **three phases** – **MVP**, **Beta**, and **Full Release** – with clear steps in each. We’ll use **Expo’s Managed Workflow** throughout (no ejecting!), leveraging **Expo Router**, **NativeWind**, **Supabase**, and **geolocation**. Each step lists the **command(s) first** (💻), followed by an explanation. The tone is fun and engaging (notice the emojis 😄), but still professional and detailed. 

## Phase 1: MVP 🏁 – Build the Foundation

In the MVP phase, we set up the fundamental architecture: Expo + Router navigation, NativeWind for UI, Supabase auth with basic roles, and geolocation. We’ll also establish a clean project structure and version control to start off right.

### 1. Initialize the Expo project 🤖

**Command:**  
```bash
npx create-expo-app@latest ScriptHammer
```

This creates a new Expo project named **ScriptHammer** using the latest managed workflow. Expo’s CLI will scaffold the project with all the essentials. By default, you’ll get an `App.js` (or `App.tsx`) and an `app/` directory if using the Expo Router template. (If you used the **tabs** template or recent Expo SDK, Expo Router might already be configured in the new app. If not, we’ll add it next.)

**Best Practice:** Keep using the **Managed Workflow** – you don't need to eject to Xcode/Android Studio. Expo’s managed approach lets you handle native features (like camera, location) via Expo SDK packages, keeping your life easier. All configuration will be via JavaScript/JSON (no native code editing), which fits our needs.

### 2. Set up Expo Router navigation 🗺️

**Command:**  
```bash
npx expo install expo-router react-native-safe-area-context
``` 

This installs Expo’s file-based router and a safe-area library. Expo Router enables **Next.js-like page routing** in React Native, where screens are files in the `app/` directory. The safe-area context ensures content isn’t hidden under notches or status bars.

Next, open your `package.json` and **set the entry point** to Expo Router:

```json
"main": "expo-router/entry"
``` 

This tells Expo to load the router’s entry file. (If you created the project with Expo Router from the start, this is likely done for you; otherwise, add it now.) 

Now, organize the navigation structure in the `app/` directory:

- **Create a file** `app/_layout.tsx` with a default layout component. This will wrap all your screens. For example: 

  ```tsx
  import { Stack } from "expo-router";
  export default function Layout() {
    return <Stack />;
  }
  ``` 

  This uses Expo Router’s Stack navigator by default for all child routes. The `_layout.tsx` serves as a layout wrapper for screens in the same folder (here, the root of `app/`). You can think of it like a global navigation container.

- **Create a screen** file, e.g. `app/index.tsx`, for your home screen. For now, it can return a simple `<Text>Home</Text>` component. Every file in `app/` becomes a route automatically. So `index.tsx` becomes the default **home** route, and other files (e.g. `profile.tsx`) become other screens. Expo Router will map these to navigation paths.

🚦 **Routing Best Practices:** Keep all screens in the `app/` directory for Expo Router to auto-detect. If you ever decide to group your source code in a `src/` folder (for a cleaner project root), you can move `app` into `src/app`. Expo Router supports a top-level `src/` directory out of the box. **Important:** Don’t have both an `app/` and `src/app` – if both exist, **only** `src/app` is used (it takes precedence). In other words, choose one approach to avoid conflicts. For now, you can keep using the root `app/` directory, or if you prefer a dedicated src folder, move it now (and leave your config files like **app.json**, **babel.config.js** at the root).

Also, to avoid unwanted routes, **do not put non-screen components inside `app/`** unless they follow Expo Router conventions. For example, if you add `app/components/Header.tsx`, Expo Router will think “Header” is a route (`/components/header`) – not what we want. To prevent this, keep such components in a separate folder (e.g. a top-level `components/` or within `src/`) so the router ignores them. This keeps the navigation clean and only mapping actual screens.

### 3. Install and configure NativeWind (Tailwind CSS for React Native) 🎨

We’ll use **NativeWind** to style our app with Tailwind CSS utility classes, giving us rapid and consistent UI styling.

**Commands:**  
```bash
npx expo install nativewind tailwindcss react-native-reanimated react-native-safe-area-context
```  
```bash
npx tailwindcss init
``` 

The first command installs NativeWind and its peer dependencies: Tailwind CSS (for styling), Reanimated (used by NativeWind for animations), and safe-area-context (likely already installed, but included just in case). The second command creates a default **`tailwind.config.js`** in your project.

Now, open **tailwind.config.js** and set up the config:

- **Specify content paths** so Tailwind can tree-shake unused styles. Include all your app directories and files. For example: 

  ```js
  // tailwind.config.js
  module.exports = {
    content: ["./app/**/*.{ts,tsx}", "./components/**/*.{ts,tsx}"],
    presets: [require("nativewind/preset")],
    theme: {
      extend: {}
    },
    plugins: []
  };
  ``` 

  We include the `app/` folder (since that’s where screens live) and any other folders where you use Tailwind classes. The `nativewind/preset` is added to integrate NativeWind’s defaults.

- **Create a global stylesheet** (e.g., **global.css** in the project root or in `app/`). Add the Tailwind directives: 

  ```css
  @tailwind base;
  @tailwind components;
  @tailwind utilities;
  ``` 

  This will include Tailwind’s base styles. (Even though we’re in React Native, these are needed for web and consistent styling.)

- **Import the CSS in your app** so it’s applied. A good place is the root layout. In `app/_layout.tsx`, import the global.css at the top: 

  ```tsx
  import "../global.css";
  import { Stack } from "expo-router";
  // ... rest of imports
  ``` 

  This ensures Tailwind’s styles are loaded (especially if you later build for web). 

- **Configure Babel for NativeWind and Expo Router:** Open **babel.config.js**. We need to add NativeWind’s preset and Expo Router’s plugin. Adjust your Babel config to include:

  ```js
  return {
    presets: [
      ["babel-preset-expo", { jsxImportSource: "nativewind" }],
      "nativewind/babel"
    ],
    plugins: [
      "expo-router/babel",
      "react-native-reanimated/plugin"
    ]
  };
  ``` 

  This does a few things: it allows using JSX with NativeWind (the `jsxImportSource`), adds NativeWind’s Babel plugin, and includes Expo Router’s Babel plugin and Reanimated’s plugin. The router’s plugin is required for file-based navigation to work, and Reanimated’s plugin is required because we installed `react-native-reanimated`. **Tip:** The order is important – make sure `react-native-reanimated/plugin` is last in the plugins array (as per Reanimated docs).

- **(Optional) Metro config:** If you run into issues with the Metro bundler not picking up the Tailwind CSS file, you might need to customize `metro.config.js` to use **withNativeWind**. For Expo SDK 50+, this isn’t usually needed, but NativeWind docs show how to do it if necessary. In most cases, the above steps suffice.

At this point, you can start using Tailwind classes in your components! For example:

```tsx
<Text className="text-xl font-bold text-blue-600">Hello NativeWind</Text>
<View className="p-4 bg-gray-100 rounded-md"><!-- ... --></View>
```

NativeWind will apply these styles on native components. This gives you quick, consistent styling without writing StyleSheet objects for everything.

### 4. Set up Supabase and user authentication 🔐

Time to add our backend. **Supabase** will provide a Postgres database, authentication, and storage. We’ll use it for user accounts and role-based access.

**Step 4a: Create a Supabase project (in browser)** – Go to the [Supabase](https://app.supabase.com) dashboard and create a new project (choose the free tier for now). Once it’s ready, find your **API URL** and **anon key**: in your project, go to **Settings > API**, and copy the **Project URL** and the **anon public API key**. We’ll need these to connect from the app.

**Step 4b: Install Supabase client SDK** in the app:

**Command:**  
```bash
npx expo install @supabase/supabase-js @react-native-async-storage/async-storage react-native-url-polyfill
``` 

This installs the Supabase JS SDK and a couple of polyfills. We include `@react-native-async-storage/async-storage` because React Native needs a storage mechanism for Supabase to persist the auth session (Expo doesn’t have `localStorage`, so AsyncStorage is used). We also install `react-native-url-polyfill` which fixes URL API issues in React Native. Expo’s `expo install` ensures we get versions compatible with our Expo SDK.

**Step 4c: Initialize Supabase in your app.** We’ll create a helper module to set up the client:

Create a file (for example, **`lib/supabase.ts`** or **`utils/supabase.ts`**):

```ts
import 'react-native-url-polyfill/auto';  // polyfill URLs for RN
import AsyncStorage from '@react-native-async-storage/async-storage';
import { createClient } from '@supabase/supabase-js';

const supabaseUrl = "https://YOUR_PROJECT_ID.supabase.co";
const supabaseAnonKey = "YOUR_ANON_PUBLIC_KEY";

export const supabase = createClient(supabaseUrl, supabaseAnonKey, {
  auth: {
    storage: AsyncStorage,
    autoRefreshToken: true,
    persistSession: true,
    detectSessionInUrl: false
  }
});
``` 

Plug in your actual URL and anon key here. The config ensures Supabase Auth uses AsyncStorage to cache the user session, so the user stays logged in between app launches. We set `detectSessionInUrl: false` because we’re in a native app (no URL callbacks for auth). Supabase’s row-level security (RLS) means it’s okay to include the anon key in the app – that key can only perform actions allowed by your database policies.

Now, we can use this `supabase` client throughout the app. For example, in a login screen component:

```tsx
import { supabase } from "../lib/supabase";

const handleSignIn = async (email: string, password: string) => {
  const { error } = await supabase.auth.signInWithPassword({ email, password });
  if (error) Alert.alert(error.message);
};
```

And similarly for sign-up:

```tsx
await supabase.auth.signUp({ email, password });
```

Supabase handles the heavy lifting: storing user credentials securely and returning a session (JWT) if login is successful.

**Step 4d: Create a user profiles table with roles.** In Supabase, by default you have an `auth.users` table (with basic user info). We want to store extra data like user role (admin, editor, etc). The common approach is to make a **`profiles`** table that references `auth.users`. 

Using Supabase SQL Editor, run a script to create a profiles table, for example:

```sql
create table public.profiles (
  id uuid references auth.users on delete cascade primary key,
  username text,
  role text default 'user'
);
alter table public.profiles enable row level security;
```

Here, each profile row corresponds to a user (same ID as in `auth.users`), and we added a `role` column which defaults to `'user'`. Enabling Row Level Security (RLS) on the table means by default no one can read/write it without explicit policies – we will add policies so users can read their own profile, etc. (We’ll handle detailed RLS policies in Full Release phase, but enable it now for safety).

**(Optional)**: Set up an **insert trigger** so that when a new user signs up, a profile is created automatically. Supabase provides a “User Management Starter” SQL snippet that does exactly this (in the SQL Editor -> Quickstarts). It basically creates a function and trigger to insert a new `profiles` row for each new `auth.users` record. You can also write it manually, e.g. a trigger calling a function `handle_new_user()` that inserts into profiles with the new user’s ID and default role. This way, you won’t have to manually create profile entries in your app – it’s handled in the DB.

**Step 4e: Connect the app to the profiles table.** After a user logs in or signs up, you’ll want to fetch their profile (to get their role and any other info). You can do something like:

```tsx
const { data: profile } = await supabase
  .from('profiles')
  .select('*')
  .eq('id', supabase.auth.getUser().data?.user.id)
  .single();
```

(This assumes the user is logged in and we have their ID). Store this profile (perhaps in React Context or Recoil/Zustand state) so it’s accessible across the app. **Best Practice:** Use a global state (or context) to keep track of the authed user and their role. For example, create an `AuthContext` that provides `{ user, profile, role }` to the app. This makes it easy to conditionally render UI based on role.

At this point, we have authentication in place. **Try it out:** Create a simple SignUp and Login screen (with form inputs for email/password). On sign up, call `supabase.auth.signUp`; on sign in, call `signInWithPassword`. If successful, store the session (Supabase client does this automatically in AsyncStorage) and navigate to the main app. Expo Router integration: you can have a group of routes for signed-in users and a separate route for auth. For example, create an `app/(auth)/` group with `sign-in.tsx` and `sign-up.tsx` routes, and an `app/(app)/` group for the main app screens that requires login. We’ll refine this in the next steps.

**Step 4f: Seed roles** – via the Supabase Dashboard, mark your own user account as an **admin** for testing. For instance, in the **Table Editor**, find your user’s profile row and change `role` from "user" to "admin". Also create a couple more users (you can use the app or the dashboard to sign them up), and assign roles like "editor" or "moderator" to test out role-based behavior later. Now we have the roles in place: **admin, editor, moderator, user** (with default being user).

### 5. Implement basic role-based access in the app 👥

With roles in the database, we can enforce some rules on the client side. In MVP, we’ll do simple gating: only admins can see the admin dashboard screen, etc.

- **Admin screen**: Create a screen for admin functions, e.g. `app/admin/index.tsx`. This might be a placeholder for now, like a screen that says "Admin Dashboard - (Admin access only)". We will flesh it out later. 

- **Route protection**: We don’t want non-admin users navigating to the admin screen. Expo Router doesn’t have built-in route guards, but we can achieve it with a layout and some logic. Create a **route group** for admin. For example, put the admin screens inside a folder like `app/(admin)/`. Inside `app/(admin)/`, create an `_layout.tsx`. In that layout, we’ll check the user’s role from context and redirect if necessary:
  
  ```tsx
  import { Stack, useRouter } from "expo-router";
  import { useAuth } from "../../providers/AuthContext"; // your context hook

  export default function AdminLayout() {
    const { profile } = useAuth();  // assuming profile contains role
    const router = useRouter();
    if (profile && profile.role !== 'admin') {
      // If user is not admin, redirect them away
      router.replace("/"); // send to home (or a /unauthorized page)
      return null;
    }
    return <Stack />;
  }
  ``` 

  What’s happening: The `(admin)` group will apply this layout to all its child routes. If a non-admin enters any admin route, we immediately `replace` the navigation to home (or you could navigate to a dedicated “Not Authorized” screen). This pattern is similar to how you protect routes based on auth state using Expo Router’s groups, just extended to roles.

- **Conditional UI**: Also use the role info to conditionally show/hide navigation options. For instance, if you have a navigation menu or tab bar, only add the Admin tab if `role === 'admin'`. For editors/moderators, you might not need separate screens yet, but as an example, if moderators should have a “Moderation” screen, implement it similarly and gate it to moderators.

At MVP stage, we mainly ensure the admin screen is not accessible to normal users. We’ll implement real admin features in Beta. The key is that the app **knows the user’s role** (from Supabase) and uses it to control navigation and UI. 

### 6. Add Geolocation support 📍

Now, let’s integrate geolocation using Expo’s Location API. We can use this to fetch the user’s current location (and later, perhaps display it or use it in features).

**Command:**  
```bash
npx expo install expo-location
```

This installs Expo’s Location module. Expo Location allows us to get GPS coordinates on both iOS and Android (and even web, as lat/long via browser API). 

**Usage:** To use it, we must request permission at runtime, and then get the location:

For example, in a screen where you need location (maybe a map or “Nearby” feature):

```tsx
import * as Location from 'expo-location';

async function getUserLocation() {
  // Ask for permission
  const { status } = await Location.requestForegroundPermissionsAsync();
  if (status !== 'granted') {
    console.warn('Location perm not granted');
    return;
  }
  const location = await Location.getCurrentPositionAsync({});
  console.log("User's coordinates:", location.coords);
}
```

We request foreground location permission – Expo will pop up the OS dialog asking the user to allow location. If granted, we call `Location.getCurrentPositionAsync()` to get the device’s current GPS coordinates. The result includes latitude, longitude, altitude, etc. You can then use `location.coords.latitude` & `longitude` as needed.

⚠️ **Permissions configuration:** On iOS, you should add a usage description in app.json so that the permission dialog has a message. For example, in **app.json** under `ios.infoPlist`, add: 

```json
"NSLocationWhenInUseUsageDescription": "Allow ScriptHammer to access your location to show relevant data."
``` 

Expo will merge this into the iOS app config. On Android, the permission is automatically added from the library’s manifest configuration (Expo handles it). Always ensure you explain to the user why you need their location.

For now, simply log the location or show it on screen (you can set state with the coords and render them). This confirms geolocation is working. In Beta, we might integrate a map or more advanced location features.

### 7. Set up version control and basic CI 🤖

Even in MVP, it's wise to have your project in **version control** (git) and perhaps a minimal CI setup.

- **Initialize git:** Inside your project folder, run: 

  **Commands:**  
  ```bash
  git init 
  git add .
  git commit -m "Initial Expo app with router and basic setup"
  ```
  
  This creates a local git repository and commits the current state. If your project was created from a git template you might already have a git repo initialized – if so, just commit your changes. 

- **Create a remote repo:** e.g., on GitHub, create a new repository "ScriptHammer". Then add it as remote and push:

  ```bash
  git remote add origin https://github.com/<yourname>/ScriptHammer.git
  git branch -M main
  git push -u origin main
  ```

  Now your code is backed up to GitHub (or whichever git service you use). 

- **Continuous Integration (CI):** At this early stage, CI can be simple. You might set up a GitHub Action to run tests (even if you just have a trivial test or lint) on each push. Since our MVP is just being built, we can defer full CI until Beta. But to anticipate: Expo projects can be integrated with GitHub Actions easily using Expo’s official action. Using it, you can have CI run `expo build` or `eas build` commands automatically. 

For now, ensure code formatting and linting pass (if you used Expo’s template, ESLint/Prettier might be set up). You could add a simple workflow that runs `npm run lint` and `npm run test` on pull requests. We will expand on CI in the next phase, including automated builds and maybe deployment. 

## Phase 2: Beta 🚀 – Expand and Harden Features

In the Beta phase, we take our solid foundation and add more advanced features and polish. This includes implementing the actual functionality of the admin dashboard and other role-based content, improving the geolocation usage (perhaps adding a map or background tracking), and setting up full CI/CD (testing, builds, and possibly beta deployments). We’ll also refine project structure if needed and ensure everything is scalable and maintainable.

### 1. Build out the Admin Dashboard 🛠️

Now that we have an `admin` screen accessible only to admins, let's fill it with useful capabilities:

- **Admin functionality:** Decide what “admin” means for your app. Common admin features include: managing users (viewing a user list, changing roles), overseeing content (if your app has user-generated content), or viewing analytics. For ScriptHammer, let's assume the admin should be able to manage user roles and perhaps moderate content.

- **User Management UI:** Use Supabase to fetch all users/profiles for admins. For example, in `app/admin/index.tsx`, when the screen loads (use `useEffect`), call:

  ```ts
  const { data: users } = await supabase.from('profiles').select('id, username, role');
  ``` 

  However, by default RLS will prevent normal users from fetching others’ profiles. We need a policy that allows an admin to select all profiles. A simple RLS policy example: *"if auth.user role is admin then allow"* – we’ll implement that in the Full Release security hardening. For now, in a dev environment, you might disable RLS on profiles or use the Supabase service role key (NOT recommended in a client app) to fetch all users. **Safer approach:** create a Supabase **Edge Function** to list users that only admins can call. But to keep things simple, you may temporarily relax the RLS for reading profiles during development of this feature, or add a policy like `SELECT * on profiles where role = 'admin' OR id = auth.uid()` just so admins can see everyone.

  Render the list of users in a table or list. Show their email (you can join with auth.users via Supabase if needed) or at least their username and current role. This confirms that as an admin you can retrieve and view all user profiles.

- **Change user roles:** Add a feature to promote/demote users. For example, an admin can select a role from a dropdown next to each user. When changed, call a Supabase RPC (remote procedure) or direct update to the profile:

  ```ts
  const { error } = await supabase.from('profiles')
    .update({ role: newRole })
    .eq('id', userId);
  ```

  On the client side, ensure only admins see this option. On the server side, enforce that only admins *can perform it* (we’ll handle with RLS policies later). After updating, perhaps re-fetch the user list to see the change. This allows the admin to assign roles like *editor* or *moderator* to users.

- **Editor/Moderator screens:** If your app calls for it, you can create dedicated screens for editors or moderators. For instance, if “editor” role is supposed to edit content, you might have an `app/editor/` route group. Similar to admin, protect it with a layout that checks `profile.role === 'editor'`. Moderators might have a moderation queue screen. In many cases, these could also be links on the admin dashboard (since an admin might also do those tasks), but separating can help if editors/mods are distinct users. The implementation pattern is the same as admin: use route groups and context to guard, and build out the UI needed.

- **Testing role flows:** Try logging in as different roles (you might need to create separate dummy users for each role to test). Verify that:
  - An **admin** user can access Admin Dashboard and see the user list.
  - A **non-admin** (e.g. normal user) cannot access `/admin` (should be redirected out).
  - If you have an **editor** role and created an Editor screen, a non-editor should be blocked there, etc.
  - Admin’s ability to change roles works and persists (check in DB that the role updated).

This completes the core role-based feature: an admin interface and the concept of multiple roles in action. It transforms our app from just authentication to **authorization** (who can do what).

### 2. Enhance geolocation features 🗺️

With basic location retrieval working, we can enhance this functionality:

- **Show location on a Map:** A great user experience is seeing their location on a map. You can use a map library like `react-native-maps` or Expo’s Maps (currently in development as `expo-maps`). For simplicity, let’s use react-native-maps:
  
  **Commands:**  
  ```bash
  npx expo install react-native-maps
  ``` 

  (Expo will install a config plugin for it since it’s a native module). Then, create a MapView in a screen (say `app/map.tsx`). Use the latitude/longitude from `Location.getCurrentPositionAsync` to set the map’s region. For example:
  
  ```tsx
  import MapView, { Marker } from "react-native-maps";
  // inside your component after getting location:
  <MapView style={{ flex: 1 }}
    initialRegion={{
      latitude: location.coords.latitude,
      longitude: location.coords.longitude,
      latitudeDelta: 0.0922,
      longitudeDelta: 0.0421
    }}>
    <Marker coordinate={{
      latitude: location.coords.latitude,
      longitude: location.coords.longitude
    }} title="You are here" />
  </MapView>
  ```
  
  This will display a map with a marker at the user’s position. Make sure to handle the case where location isn’t available (you might need to ask permission on that screen if not already granted).

- **Background location (if needed):** If your app requires tracking location even when the app is backgrounded (e.g., a fitness tracker or delivery app), Expo can do that via `Location.startLocationUpdatesAsync` and Task Manager. This is more advanced and requires configuring tasks – possibly overkill for ScriptHammer unless that’s a key feature. If not needed, you can skip background location.

- **Geolocation error handling:** Make sure to handle scenarios like the user denying permission (show a gracious message that location is needed for X feature, and allow them to proceed without it or retry). Also consider adding a setting in-app to turn location on/off (which essentially just triggers the permission prompt again if off).

By the end of this, the app is using geolocation in a meaningful way – perhaps showing the user’s location on a map or using the coordinates for some feature (like finding nearby resources, etc., depending on your app's domain).

### 3. Refine project structure and code organization 📂

As the codebase grows in Beta, keep it organized:

- **Adopt a modular structure:** Perhaps create a `src/` directory now (if you haven’t already) and move your code inside it:
  - `src/app/` – all route files and layouts (so `src/app/_layout.tsx`, `src/app/index.tsx`, etc.).
  - `src/components/` – reusable presentational components (buttons, cards, etc.).
  - `src/hooks/` – custom hooks (e.g., for auth or data fetching).
  - `src/providers/` – context providers (AuthContext, ThemeProvider, etc.).
  - `src/utils/` – utility functions and maybe the `supabase.ts` client file.
  
  This keeps the root clean. Expo Router will automatically detect `src/app` as your app entry (it prefers `src/app` if it exists). Just ensure you update any imports accordingly. All your config files (babel.config.js, package.json, etc.) remain in root (per Expo docs).

- **Verify module resolution:** If you moved to `src/`, update your `tsconfig.json` baseUrl to `"./src"` (optional) and set up path aliases if useful (like `@components` -> `src/components`). Expo’s Metro bundler can handle that via Babel plugins or the `paths` field in tsconfig (since we’re still in a Node/Metro environment, not webpack).

- **Clean up tech debt:** Remove any placeholder code or console.logs that were used during MVP. Ensure that each module has a clear responsibility. For example, if your admin logic was all in one file, consider splitting parts out (maybe a separate component for the user list, etc.). This improves maintainability.

- **UI/UX improvements:** Not exactly “structure,” but as part of Beta polish, you might want to integrate a UI kit or just enforce consistent styles. NativeWind gives you consistency with Tailwind; you could also add some premade components (maybe use React Native Paper or UI Kitten if needed, or just build custom ones). The key is to avoid duplicating style logic – leverage the utility-first approach of Tailwind.

At this point, the codebase should be structured in a way that any developer joining the project can find things easily. We have a clear separation of concerns and a scalable layout for adding new features.

### 4. Testing 🧪 and Continuous Integration/Delivery (CI/CD)

Now that the app is more complex, **automated testing** and a robust **CI/CD pipeline** become important to catch issues and streamline deployments.

- **Write tests:** Set up **Jest** and React Native Testing Library for unit and integration tests.

  **Commands:**  
  ```bash
  npx expo install jest-expo @testing-library/react-native @testing-library/jest-native
  npm install -D jest @types/jest @testing-library/jest-dom
  ```
  
  The `jest-expo` preset simplifies Jest configuration for Expo apps. In your **package.json**, under `jest`, set:
  
  ```json
  "jest": {
    "preset": "jest-expo",
    "transformIgnorePatterns": [
      "node_modules/(?!(@?expo(-.*)?|@react-native|react-native|react-native-.*|@react-native-.*|nativewind|solito)/)"
    ]
  }
  ```
  
  (Expo might have already added a basic config). Write tests for critical pieces:
  - Auth logic: e.g., a test for the context that when given a user with role 'admin', `isAdmin` flag returns true.
  - Component rendering: test that the Admin screen shows the user list for admins, and perhaps that a non-admin would be redirected (this might require mocking the router or testing the logic in isolation).
  - Any utility functions.

  Also consider end-to-end testing. Expo Router apps can be tested with Detox or Cypress (if you run the web build). This might be advanced; you could plan to add e2e tests in Full Release if needed.

- **Set up GitHub Actions for CI:** We want every commit (or at least every pull request to main) to trigger our test suite and possibly build the app. Use the **Expo GitHub Action** which gives you access to Expo CLI and EAS CLI in the workflow.

  Create a file **.github/workflows/ci.yml** with content such as:

  ```yaml
  name: CI
  on: [push, pull_request]
  jobs:
    build-and-test:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v3
        - uses: expo/expo-github-action@v2
          with:
            expo-cache: true  # cache deps
        - run: npm ci
        - run: npm run test -- --watchAll=false
        - run: npx expo export --platform web  # example: build web bundle for testing, optional
  ```

  This would run the tests on each push. You can add `matrix` to run on multiple node versions or add lint step, etc., as needed. The key is: ensure the app builds and tests pass in CI.

- **Continuous Deployment** (CD): Now let's automate deploying the app for testing. With Expo, we have a couple of paths:
  - **EAS Build for app stores:** We can use EAS to build standalone binaries (APKs, IPAs). For Beta, we might not go to public stores yet, but we can distribute to internal testers.
  - **Expo Go / OTA updates:** Since we use Expo Router and managed workflow, we can push updates with `expo publish` for testing in Expo Go. But since we have native modules (e.g., reanimated, maps), a true OTA might not work for those without a rebuild. So EAS Build is preferable for full testing.

  **Set up EAS:** If you haven't, run `npx eas init` locally to create an **eas.json** (with profiles for development, preview, production). Define a **development** profile (for internal testing) and a **production** profile (for app store). Example **eas.json** snippet:
  
  ```json
  {
    "build": {
      "development": { "distribution": "internal", "android": { "buildType": "apk" } },
      "production": { "distribution": "store" }
    }
  }
  ```
  
  This way, `eas build --profile development --platform android` will generate an APK you can share, and `--platform ios` an .ipa for TestFlight.

  - **CI for EAS:** Use the Expo GitHub Action to run EAS builds on certain triggers. For example, on push to `main`, you might trigger a dev build:
  
    ```yaml
    - uses: expo/expo-github-action@v2
      id: eas-build
      with:
        eas-version: latest
        expo-cache: true
        token: ${{ secrets.EXPO_TOKEN }}
        run: |
          eas build --profile development --platform all --non-interactive
    ```
  
    This will kick off an Expo cloud build. Note you'll need to set an **EXPO_TOKEN** (you can generate one at expo.dev) in GitHub secrets, and also your **EAS project ID** (usually EAS finds it from app.json automatically). The action above caches and authenticates with Expo, then runs a build for both platforms in the cloud. You can monitor the build on expo.dev. 
  
  - **Deploy to testers:** If using internal distribution, the artifacts (APK/IPA) can be downloaded from the Expo dashboard. You could also automate uploading to a service or directly to app stores:
    - Use **EAS Submit** to send to TestFlight/Google Play Beta. E.g., after build, `eas submit -p ios --latest` can upload the latest iOS build to App Store Connect (you'll need credentials configured).
    - Or simply take the build output and share with testers (APK link or using Expo's internal distribution QR code).
  
  - **Version control & releases:** Adopt a versioning strategy. For Beta, maybe increment version for each build (Expo uses `version` and `buildNumber` in app.json for iOS, and `versionCode` for Android). You can automate bumping these via `expo-version` or manually update for each release. Consider tagging releases in git (e.g., v0.2-beta tag) so you know what commit went to testers.

At the end of Beta phase, we have a pipeline where every code change is tested, and we can easily push a button (or automatically on merge) to build a new release for internal testing. This greatly reduces friction and catches bugs early. 

Our app now is quite robust: it has multi-role support, a working admin panel, real styling, and a map feature (assuming you added it). We can proceed to finalize everything for a public release.

## Phase 3: Full Release 🎉 – Polish and Deploy to Production

In this phase, we focus on **performance, security, and deployment**. We want to ensure the app is production-ready: all role-based access controls are enforced server-side as well, the app is optimized, and we have a smooth process to deploy to the app stores (and perhaps web).

### 1. Harden security and backend rules 🔒

Up to now, we’ve enforced roles mostly on the client (hiding/showing UI, restricting navigation). For a production release, we **must enforce these rules on the backend** too, so malicious users can’t simply use the API directly to do forbidden actions.

- **Supabase RLS Policies:** We enabled RLS on the profiles table. Now define policies that implement our role logic:
  - *Allow users to read their own profile:* For example, a policy: **SELECT on profiles** with `using (auth.uid() = id)` – meaning a user can select their own row.
  - *Allow admin to read any profile:* A policy for admins: `using (exists(select 1 from profiles p where p.id = auth.uid() and p.role = 'admin'))` – i.e., if the requesting user is an admin, allow the operation (this is one way to check the caller's role by joining their profile).
  - *Allow updates to profile:* Regular users might update their own profile info (like username), but **not their role**. Set a policy: `using (auth.uid() = id and new.role = role)` to allow updates if it’s the user and they are not changing their role field (so they can edit other fields, but not assign themselves a new role). And then a separate policy to allow admins to update any profile (including role changes).
  
  Supabase’s policy syntax uses `auth.uid()` for the user’s ID and you can fetch JWT claims if needed (with `auth.jwt()` for custom claims). Since we didn’t implement custom JWT claims for role, using a sub-select on the profiles table is a straightforward way to check role inside a policy.

  For example, a policy to allow **admins to update profiles**:
  ```sql
  create policy "Admins can update profiles"
    on profiles for update
    using (
      (select role from profiles where id = auth.uid()) = 'admin'
    );
  ```
  This checks the calling user's own role. Similarly, "Admins can read profiles" for select. (You could combine some of these into one policy using OR logic as well).

  And a policy "Users can update their profile (excluding role)" using the `new` record to ensure `new.role = old.role` (so they can’t change their own role).

  Also consider any other tables: if you have an `items` table or posts that moderators should moderate, write policies for those (e.g., only moderators or owners can delete a post, etc.).

- **Use Supabase Custom Claims (optional advanced):** Supabase has a feature where you can add **custom JWT claims** via an Auth hook, so that the user’s role is embedded in the JWT token on login. This can simplify policies by letting you do `auth.jwt()['role'] = 'admin'` in policy instead of a sub-query. Implementing this involves writing a Go or TypeScript function in Supabase that runs on sign-in to attach the user’s role. Given our timeline, you might skip this or consider it later, as our sub-query approach works and is more straightforward to set up. (Custom claims are powerful but require self-hosting an auth hook or using Supabase Edge Functions.)

- **Secure the admin RPCs:** Ensure that any Supabase RPC (remote procedures) or direct queries used for admin functions are protected. For instance, if you made an RPC for updating roles, add an auth check inside it (using `auth.uid()` and checking role in SQL) or simply rely on the RLS policies by performing updates through the regular `profiles` table (which we have protected by policy).

- **Verify from client-side:** After setting policies, test from the client:
  - As a normal user, try to fetch another user’s data via `supabase.from('profiles').select(...)` – it should return nothing or an error now.
  - Try to change your role via API – it should be forbidden.
  - As admin, ensure you *can* still fetch all profiles and update roles.
  - Our app’s UI already prevents these actions, but it’s good to ensure the backend blocks them too if attempted.

By tightening these rules, we ensure **role-based access control is truly enforced**. Even if someone reverse-engineered our app or found our Supabase URL/key, they cannot do more than their role permits. This is crucial for a real app with sensitive data.

### 2. Final performance and quality optimizations 🚀

Before release, polish the app:

- **Optimize assets:** Use proper asset management for images (Expo Asset can bundle images or use CDN). Make sure large images are compressed. Use vector icons (Expo includes `@expo/vector-icons`) for things like icons to avoid many image assets.

- **Minimize app load time:** Expo apps are pretty optimized out of the box, but you can do things like code-splitting by routes (Expo Router automatically splits by route, so that’s a win). Ensure you’re not loading heavy data on the initial screen unnecessarily. Use lazy loading for screens if needed (Expo Router supports `React.lazy` for importing screens or using the `?` convention for group routes to defer loading).

- **Check bundle size:** Run `npx expo export` or `eas build` and see the JS bundle size. If huge, consider if you accidentally included something large (like moment.js with all locales, etc.) and use lighter alternatives or dynamic imports.

- **Crash and error monitoring:** Integrate an error tracking service (optional but good practice) like Sentry or LogRocket before release, so you can catch errors in production. Expo has guides for these (e.g., `expo install sentry-expo` and configure it).

- **Analytics:** It might be useful to add analytics (e.g., Firebase Analytics or Amplitude) to track usage, especially around roles (e.g., how many admin actions, etc.). This is not strictly required, but product-wise often included in final release.

- **UI polish:** Go through each screen and clean up any rough edges. Ensure consistent spacing, use SafeAreaView where appropriate (especially for iPhone X notches – Expo’s `SafeAreaProvider` is already installed via safe-area-context). Perhaps add a splash screen and app icon if not done (configure in app.json, provide the asset files).

- **Localization (if needed):** If you plan multi-language support, set it up now using something like i18n. If not, at least ensure all text is spell-checked and there are no placeholder texts left.

- **App Store compliance:** Make sure to fill out app.json fields like `version`, `buildNumber` (for iOS), `versionCode` (Android) and app name, etc. Expo will use those when building the binaries. For release, bump the version to 1.0.0 (or whatever scheme you use).

### 3. Finalize deployment pipeline 🏷️

Now we prepare to ship:

- **App Store accounts:** Ensure you have an Apple Developer account and a Google Play Developer account set up, and your app’s listing prepared (screenshots, description, etc.).

- **Production builds:** Use EAS to create release builds:
  
  **Commands (local or via CI):**  
  ```bash
  eas build --profile production --platform ios
  eas build --profile production --platform android
  ``` 

  This will generate an IPA for iOS and an AAB for Android (store-ready formats). If running via CI, you can have a workflow dispatch or a tag trigger for releases that does this automatically. For example, push a git tag `v1.0.0` – GitHub Actions could catch that and run the above build commands.

- **Submit to stores:** Once builds are done, either download them and upload through Xcode and Google Play Console, or use `eas submit`:

  ```bash
  eas submit -p ios --latest --apple-id <your-apple-id-email> --asc-app-id <app-store-app-id>
  eas submit -p android --latest --android-package <your.app.package>
  ```
  
  (You'll need to have credentials stored or pass them via flags/secrets – EAS CLI helps with this). This will send the builds to Apple App Store Connect and Google Play respectively. Then you can go to the store listings and release to beta or production as you wish.

- **Web deployment (optional):** If you want, you can deploy the web version of your Expo app (Expo Router works on web too!). Run `npx expo export:web` to generate a static site, and host it (GitHub Pages, Vercel, etc.). Make sure to set up any needed config (as per expo docs, maybe configure a base path or use `expo build:web`). This can be a nice bonus if you want a PWA version of ScriptHammer. Ensure to secure it similarly (the Supabase auth will work on web as well, but you'd need to allow auth redirects – Supabase needs redirect URLs set in its auth settings if using OAuth providers).

- **Version tagging:** Mark the release version in git (e.g., git tag `v1.0.0` and push tags). This, combined with CI, ensures traceability. You might also use semantic versioning for subsequent releases (1.0.1 for patches, 1.1.0 for minor features, etc.).

- **Post-release monitoring:** With the app live, use the integrated Sentry (if added) to monitor crashes. Also use Supabase’s logging and monitoring for your database (Supabase can show you logs of requests, etc., and you should monitor for any  errors or suspicious activity, especially around the protected routes – though with proper RLS, it should be safe).

Finally, celebrate 🎉! You have delivered ScriptHammer through MVP to a polished 1.0 release. 

Throughout this journey, we followed best practices:
- We stuck to Expo’s Managed Workflow, using `expo` CLI tools and config-driven native features (no eject = simpler maintenance).
- We leveraged **Expo Router** for intuitive navigation structure, and kept the project structure clean and conflict-free (using either `app/` or `src/app` consistently).
- We implemented **Supabase Auth** with **role-based access control**, handling roles both on the client (for UX) and in the database (for security), giving admins, editors, moderators, and users appropriate privileges.
- We documented tutorial-like steps with explanations, so any developer (new or experienced) can follow along and understand the *why* behind each action.
- We set up a CI/CD pipeline with testing and automated builds, so our deployments are reliable and our code is always validated by tests. This pipeline uses Expo’s tooling for efficiency (caching, managed credentials).
- We kept the tone fun with emojis and clear headings, making the guide enjoyable to read while being informative.

By following this roadmap, ScriptHammer is developed in a maintainable, scalable way. The modular approach in phases ensures that at each milestone (MVP, Beta, Full Release), the app is functional and testable, while continuously improving quality. Both newcomers and seasoned developers should find the structure logical and the reasoning sound. Now go forth and build that app! 🚀👏

