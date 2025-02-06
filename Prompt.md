You are a seasoned full-stack developer tasked with building a complete, production-ready social network application. Your solution must consist of three standalone scripts that integrate seamlessly and adhere to industry best practices for maintainability, modularity, and production readiness. The scripts must work together as a cohesive whole, with no placeholders or incomplete snippets—every section should be complete and ready to run.

The three scripts are:

1. **SQL Script 1: Supabase Backend Setup with RBAC**
   - Create a comprehensive database schema on Supabase for a robust social network.
   - **Tables & Features:**
     - **Users:**
       - Integrate with Supabase Auth for email/password and OAuth authentication.
       - Include user metadata such as full name, email, and a profile picture URL.
       - Allow users to upload and edit their profile picture and displayname.
       - Include a role field that implements Role-Based Access Control (RBAC); the first user to sign up becomes the admin, while subsequent users default to regular users. Additionally, create roles for moderator and editor.
     - **Friend Requests:**
       - Store friend request details with statuses (pending, accepted, declined).
     - **Posts:**
       - Support text and image posts with real-time update capabilities.
     - **Comments:**
       - Support nested replies on posts.
     - **Reactions:**
       - Allow for likes, dislikes, and a variety of emoji reactions.
     - **Chats:**
       - Support both individual and group chats with all necessary metadata.
     - **Notifications:**
       - Record notifications for friend requests, chat messages, posts, and other interactions.
     - **Geolocation:**
       - Store user location data for real-time tracking. User must be able to toggle off easily
   - **Security & Real-Time:**
     - Implement robust Row-Level Security (RLS) policies on every table.
     - Explicitly include RBAC in your security design—ensure that the admin, moderator, editor, and regular user roles have appropriate access levels.
     - Configure real-time subscriptions for posts, messages, and notifications.

2. **Bash Script: React Native Expo Frontend Setup**
   - Automate the creation and configuration of a new React Native Expo project (SDK 52). The default project name is “ScriptHammer” (allowing for customization). The script must execute the following terminal commands:
     ```
     npx create-expo-app ScriptHammer
     cd ScriptHammer
     npm run reset-project
     rm -rf app-example
     ```
   - **Project Configuration:**
     - **Authentication:**
       - Integrate Supabase Auth for both email/password and OAuth.
     - **State Management:**
       - Set up Zustand for global state management.
     - **Geolocation:**
       - Add functionality for real-time geolocation tracking using libraries such as expo-location and react-native-maps.
     - **Global Theming:**
       - Implement a global theme using NativeWind, supporting system default, light, and dark modes.
     - **User Interface & Navigation:**
       - Include a branded header with a logo in the top left.
       - Create an integrated admin dashboard accessible within the app.
       - Develop a protected, tab-based layout with the following tabs:
         - **Home:** A feed displaying posts and comments.
         - **Messages:** Interfaces for individual and group chats.
         - **Friends:** A section to manage friend requests and view friend lists.
         - **Notifications:** A display for alerts regarding interactions and updates.
         - **Map:** A map that shows the user’s current location with real-time updates.
     - **Profile Management:**
       - Implement functionality for users to upload and edit their profile picture, integrating features like expo-image-picker and Supabase Storage.

3. **SQL Script 2: Dummy Data Population**
   - This script is intended to run after the admin user has signed up.
   - **Data Insertion:**
     - Insert three dummy users with distinct roles: one regular user, one moderator, and one editor.
     - Populate the database with sample data including:
       - Friend requests between users.
       - Sample posts, comments, and reactions from each dummy user.
       - Simulated chat conversations between each dummy user and the admin.
       - Geolocation data for each user to simulate real-time tracking.

**Important Requirements:**
- Each script must be a complete, standalone file or code block with no missing sections or placeholders.
- The generated code must be fully functional and ready for production use.
- Follow industry best practices for code modularity, maintainability, and integration with existing systems.
- Ensure that RBAC is explicitly implemented in the backend to manage different user roles (admin, moderator, editor, regular user).
- Pay particular attention to the feature for users to upload and edit their profile picture.
- Provide the final output as one complete response with clear demarcations between the three scripts.

Before deploying in production, thoroughly test all scripts in a development environment to ensure compatibility with your system's configurations and dependencies.
