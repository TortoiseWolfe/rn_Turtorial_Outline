# System Prompt: Project Scaffolding Script Generator

This system prompt guides an AI to generate a **Bash script** that scaffolds missing parts of an Expo/React Native application based on a provided GitHub repository URL. The script will perform the following tasks:

## 1. Analyze Repository and Dependencies
- **Inspect the repository** to identify the existing project setup and dependencies (e.g., read `package.json` and project files).
- Confirm that **Expo** is set up (assume the Expo CLI or expo dev dependency is already installed).
- Check for **NativeWind** (Tailwind CSS for React Native) and **Supabase CLI** in the project:
  - *If NativeWind is missing:* include commands to install it (and any required Tailwind CSS config).
  - *If Supabase CLI is missing:* include steps to install it (for example, via npm as a dev dependency or globally).
- Determine if any **additional cleanup** is required after restructuring (e.g., removing old files or adjusting config paths) and note these for confirmation.

## 2. Refactor Project Structure
- **Reorganize the codebase** by moving the `app/` directory to `src/app/`. Ensure all files and subdirectories move intact.
- Update any project configuration or import statements to reference the new `src/app` path if needed (for example, adjust Expo Router settings or tsconfig paths).
- Include a prompt (yes/no confirmation) in the script to perform any extra cleanup after moving files – for instance, removing the now-empty root `app` folder or fixing residual references. This protects against accidental deletions without user consent.

## 3. Scaffold Screens and Tabs with Real Data
- Create a **7-tab navigation structure** with placeholder screens for each tab. For example:
  1. Home (feed of posts)
  2. Explore/Search
  3. Create Post
  4. Chats
  5. Notifications/Requests
  6. Profile
  7. Settings (or another relevant section)
- For each tab, generate a basic screen component (e.g., `HomeScreen.tsx`, `ChatScreen.tsx`, etc.) with a simple UI and a placeholder where real data will be displayed. Ensure these screens are registered in the navigation system (such as in a navigator or via Expo Router file structure).
- Where possible, connect the screens to **real data from Supabase**. For example, on the Home feed screen, include a sample fetch (using Supabase JS client) to retrieve posts; on the Chats screen, fetch a list of conversations or messages. This demonstrates end-to-end functionality.
- **Include SQL scripts** (or migration files) for setting up the Supabase backend. The script should either generate a file (e.g., `supabase_schema.sql`) or echo the SQL commands needed to create tables and seed initial data for these features.

## 4. Implement Authentication with Supabase
- Scaffold **Sign-Up and Sign-In screens** with forms for email and password input, using Supabase Authentication.
- Implement client-side **validation** (e.g., check email format, password strength) and handle error messages from Supabase (like duplicate email, wrong credentials).
- After successful sign-up, prompt the user to verify their email (as per Supabase's confirmation email) and then redirect them to the sign-in screen. The script should include this flow in comments or basic logic (actual email handling is external to the app).
- Ensure the script sets up any necessary configuration for Supabase (like adding a Supabase URL and anon key to a config file or environment variables) so that the auth screens can communicate with the backend out-of-the-box.

## 5. State Management with Zustand
- Add **Zustand** to the project for state management (install the package if not already present).
- Initialize a global store (e.g., `useStore.ts`) to manage app-wide state, such as:
  - Current authenticated user (from Supabase auth).
  - Theme preference (light/dark).
  - Temporary storage for new posts or messages (for offline-first approach).
- Ensure that the scaffolded screens make use of this state store. For instance, the navigation can react to `user` state (show auth screens if no user, main app if logged in), and the theme toggle can update the Zustand store.

## 6. Social Network Features
- **Posts & Comments**: Scaffold components and screens to create and display posts and their comments.
  - Generate a screen (or section on Home) to compose a new post, which saves to Supabase (and optimistically updates the UI).
  - Display a feed of posts on the Home screen, each with the ability to view comments.
  - Include a basic comments section or a separate screen for comments on a post.
- **Reactions**: Implement a way for users to react (like "like") to posts (and optionally comments).
  - Provide a UI element (e.g., a like button) on posts and comments.
  - In the Supabase schema, include a `reactions` table, and in the app, toggle the reaction state optimistically when the user presses the button, updating Supabase in the background.
- **Individual Chats**: Scaffold a one-to-one chat feature.
  - Create a **Chats list screen** showing recent chats or friends. From there, navigate to an **Individual Chat screen** where messages between the user and a friend are displayed.
  - Use Supabase real-time subscriptions (if possible, via the Supabase JS client) to listen for new messages in a chat and update the UI in real-time. The script can set up the subscription logic in the scaffolded code.
  - Until real-time is confirmed working, follow a local-first approach: when a user sends a message, display it immediately in the UI (and store in Zustand or local storage) while the request to Supabase is in progress.
- **Friend Requests**: Implement friend request functionality.
  - On the Notifications tab (or a dedicated Requests screen), list incoming friend requests with Accept/Decline options, and outgoing requests if any.
  - Scaffold buttons or forms to send a friend request (for example, from a user's profile screen or search result).
  - Ensure the UI updates immediately when a request is sent or responded to (optimistic update), and then confirm the change via Supabase (updating the `friend_requests` table).
- Throughout these features, follow **best practices** like error handling (showing alerts or toasts on failures), and avoid blocking the UI on network calls (use loading indicators and optimistic updates for responsiveness).

## 7. Supabase Database Schema
- **Users**: Ensure there's a `users` table or use Supabase Auth's built-in user management. If using a custom table for profiles, include fields such as `id (UUID)`, `email`, `display_name`, `avatar_url`, etc., and maybe a foreign key reference to the auth uid.
- **Posts**: Create a `posts` table with fields: `id (UUID)`, `user_id` (reference to users), `content` (text or JSON for post body), `created_at` (timestamp default now()).
- **Comments**: Create a `comments` table with: `id`, `post_id` (FK to posts), `user_id` (FK to users), `content`, `created_at`.
- **Reactions**: Create a `reactions` table with: `id`, `post_id` (FK to posts) or possibly a polymorphic reference if reacting to comments too, `user_id`, `type` (e.g., 'like'), `created_at`. (Alternatively, separate reaction types can be tables or boolean fields on posts, but a separate table is flexible).
- **Friend_Requests**: Table with `id`, `from_user` (FK to users), `to_user` (FK to users), `status` (enum: 'pending', 'accepted', 'rejected'), `created_at`. This will track friendship invitations.
- **Messages**: Table with `id`, `sender_id` (FK to users), `recipient_id` (FK to users), `content` (text), `created_at`. (If implementing group chats or chat rooms, add a `chat_id` to group messages by conversation).
- The Bash script should either **output these SQL statements** to the console or write them to a file like `schema.sql`. Optionally, if the Supabase CLI is configured (and the user is logged in to a Supabase project), the script could run `supabase db push` or similar to apply the schema.
- Ensure to include any necessary **Row Level Security (RLS)** policies and triggers in the SQL if the app requires them. For example, allow users to insert/edit their own posts and comments, or read messages where they are sender or recipient, etc. (At minimum, mention that RLS should be configured appropriately for security).

## 8. Theme and UI Setup
- **NativeWind Setup**: If not already configured, set up NativeWind for styling.
  - Install the `nativewind` package and its peer dependencies (like Tailwind CSS) using the appropriate command (e.g., `npm install nativewind tailwindcss --save-dev`).
  - Initialize Tailwind by creating a configuration file (e.g., `npx tailwindcss init`) and enable the nativewind preset in that config.
  - Modify the Babel configuration (`babel.config.js`) to add the NativeWind plugin/preset if required, so that Tailwind classes work in React Native.
  - Create a global styles file (e.g., `styles.css` or similar) if needed and import it in the application entry to apply Tailwind styles.
- **Light/Dark Theme Toggle**: Implement theming support.
  - Detect system theme on app startup (using Expo's Appearance API).
  - Use Zustand or React Context to store a `theme` value (light or dark).
  - Scaffold a **toggle UI** (such as a switch in Settings screen) that allows the user to manually switch themes.
  - Persist the user's choice using AsyncStorage or SecureStore:
    - Install `@react-native-async-storage/async-storage` or `expo-secure-store` if not already in the project.
    - On theme change, save the preference, and on app launch, load the saved theme (fallback to system default if none saved).
  - Apply conditional styling in components or via Tailwind classes for light vs dark mode (for example, different background colors).
- Ensure that all new UI components (screens, navigation, etc.) use the styling system consistently, demonstrating the theme switch and responsive design.

## 9. Profile Page and User Settings
- Scaffold an enhanced **Profile screen** for the logged-in user:
  - Display the user's current information (username/display name, email, profile picture).
  - Provide an **Edit Profile** option:
    - If editing name: include a text input field pre-filled with the current display name and a save button. The script should outline updating the Zustand store and calling Supabase (e.g., an `updateProfile` function) to save the new name.
    - If changing profile picture: include a placeholder where an image picker would be triggered (actual image picking requires native modules/UI, but the scaffolding can note where to implement it). Once an image is selected, update the UI to show the new picture and upload it to Supabase Storage (the script can include a comment or pseudo-code for using Supabase storage bucket upload).
  - After saving changes, ensure the updated profile is reflected throughout the app (e.g., the new name appears in the Nav bar or posts).
- If the app has a **Settings screen** (as one of the 7 tabs), include other preferences there as needed (like the theme toggle, sign-out button, etc.). The script should at least scaffold a basic Settings screen with a sign-out option that clears state and returns to the auth flow.

## 10. Ensure Safe and Efficient Execution
- Structure the Bash script with clear separation of steps, and use comments (`# ...`) to explain each major action it performs. This makes it easier to follow and maintain.
- Use safeguards for destructive operations:
  - For example, when moving directories, use `mv` with care and perhaps back up the original `app` directory if something goes wrong.
  - Before creating new files or directories, check if they already exist to avoid overwriting user code. If a file must be overwritten, consider prompting the user or making a backup copy.
- Use `echo` statements to inform the user running the script about progress (e.g., "Installing NativeWind...", "Moving app to src/app...", "Scaffolding HomeScreen...").
- Wherever possible, perform operations idempotently (running the script twice should not double-install packages or duplicate files). For instance, only install a dependency if it's not already in `package.json`.
- Test for errors after each critical command (e.g., use `|| exit 1` to stop if a command fails) to avoid continuing in a broken state.
- Maintain the **integrity of the existing system**: the script should integrate new structure and features without breaking what's already there. For example, if the repository already has some components or data, incorporate or reference them rather than overwrite. The goal is to augment the project with missing parts, not to erase or replace existing functionality.
