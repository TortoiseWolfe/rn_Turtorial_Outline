You are a seasoned full-stack developer tasked with building a complete, production-ready social network application. The solution must consist of three standalone scripts that integrate seamlessly:

1. **SQL Script 1: Supabase Backend Setup**
   - Create a comprehensive database schema for a robust social network on Supabase. This schema must include the following tables with proper relationships, constraints, indexes, and robust Row-Level Security (RLS) policies:
     - **Users:**
       - Integrate with Supabase Auth for email/password and OAuth authentication.
       - Include fields for user metadata such as full name, email, and a profile picture URL.
       - Allow users to upload and edit their profile picture.
       - Include a role field (the first user to sign up becomes the admin; subsequent users default to regular users; dummy roles include moderator and editor).
     - **Friend Requests:**
       - Store friend request details, including statuses (pending, accepted, declined).
     - **Posts:**
       - Support text and image posts with real-time update capabilities.
     - **Comments:**
       - Support nested replies on posts.
     - **Reactions:**
       - Allow for likes, dislikes, and various emoji reactions.
     - **Chats:**
       - Support both individual and group chats with the required metadata.
     - **Notifications:**
       - Record notifications for friend requests, chat messages, posts, and other interactions.
     - **Geolocation:**
       - Store user location data for real-time tracking.
   - Ensure that real-time subscriptions are configured for posts, messages, and notifications.

2. **Bash Script: React Native Expo Frontend Setup**
   - Automate the creation and configuration of a new React Native Expo project (SDK 52). The default project name is “ScriptHammer” (allowing for customization). This script must run the following commands:
     ```
     npx create-expo-app ScriptHammer
     cd ScriptHammer
     npm run reset-project
     rm -rf app-example
     ```
   - Within the generated project, set up the following functionalities:
     - **Authentication:**
       - Integrate Supabase Auth (email/password and OAuth options).
     - **State Management:**
       - Implement Zustand for global state management.
     - **Geolocation:**
       - Add real-time geolocation tracking using appropriate libraries (e.g., expo-location and react-native-maps).
     - **Global Theming:**
       - Use NativeWind to implement a global theme that supports system, light, and dark modes.
     - **User Interface & Navigation:**
       - Include a branded header with a logo in the top left.
       - Create an integrated admin dashboard within the app.
       - Develop a protected, tab-based layout with these tabs:
         - **Home:** Display a feed of posts and comments.
         - **Messages:** Provide interfaces for individual and group chats.
         - **Friends:** Manage friend requests and display friend lists.
         - **Notifications:** Show alerts for interactions and updates.
         - **Map:** Display the user’s current location on a map with real-time updates.
     - **Profile Management:**
       - Allow users to upload and edit their profile picture using file upload capabilities (for example, by integrating expo-image-picker and Supabase Storage).

3. **SQL Script 2: Dummy Data Population**
   - This script is intended to run after the first (admin) user has signed up.
   - Insert three dummy users with distinct roles: one regular user, one moderator, and one editor.
   - Populate the database with sample data that includes:
     - Sample friend requests between users.
     - Sample posts, comments, and reactions by each dummy user.
     - Simulated chat conversations between each dummy user and the admin.
     - Geolocation data for each user to simulate real-time tracking.

**Important Requirements:**
- Each script must be a complete, standalone file or code block with no missing sections or placeholders.
- The code in all scripts must be fully functional and ready for production use.
- Emphasize industry best practices, modularity, maintainability, and integration with existing systems.
- The scripts must work together as a cohesive whole and be easy to expand upon in the future.
- Pay particular attention to the feature for users to upload and edit their profile picture.
- The output should be provided in one complete response with clear demarcations between the three scripts.

Follow these instructions precisely and generate the three complete scripts accordingly.
