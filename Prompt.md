---

**Prompt:**

I need two SQL scripts that sandwich one Bash script to fully set up a robust social network using Supabase for the backend and React Native Expo with Zustand for the frontend. 

### **SQL Script 1 (Setup Supabase Backend)**
This script should:
- Define a full-featured database schema with best practices, including tables for:
  - Users (with authentication via Supabase Auth, role-based access control, and user metadata)
  - Friend requests (with status tracking: pending, accepted, declined)
  - Posts (supporting images, text, and real-time updates)
  - Comments (nested replies supported)
  - Reactions (likes, dislikes, emojis, etc.)
  - Direct messages (including individual and group chats)
  - Notifications (for friend requests, message updates, etc.)
  - Geolocation tracking (storing user locations with real-time updates)
- Implement **Row-Level Security (RLS)** for all necessary tables.
- Enable **real-time subscriptions** to posts, messages, and notifications.
- Ensure relationships between tables are well-structured with foreign keys and constraints.

---

### **Bash Script (Setup React Native Expo Frontend)**
This script should:
1. Execute the necessary terminal commands:
   ```bash
   npx create-expo-app ScriptHammer
   cd ScriptHammer
   npm run reset-project
   rm -rf app-example
   ```
   - (Allowing flexibility for a different project name)
2. Set up **authentication** with Supabase Auth (email/password, OAuth options like Google, Apple, etc.).
3. Implement **state management** using Zustand.
4. Include **geolocation** (suggested: `expo-location` and `react-native-maps` for tracking & displaying user locations in real-time).
5. Add a **global theme** using NativeWind, with support for system theme, light/dark mode toggle.
6. Set up an **admin dashboard** integrated into the app, allowing the first user to register as the admin.
7. Include a **protected tab-based layout**:
   - **Home:** Feed displaying real-time posts & comments.
   - **Messages:** Chat functionality with direct & group messaging.
   - **Friends:** List of friends & pending requests.
   - **Notifications:** Updates for interactions.
   - **Map:** Real-time user location tracking.

---

### **SQL Script 2 (Populate Database with Test Data)**
This script should:
- Run **after the first admin signs up**.
- Create **three dummy users**: a general user, a moderator, and an editor.
- Generate sample **friend requests**, **posts**, **comments**, and **reactions**.
- Simulate **chat conversations** between each dummy user and the admin.
- Populate the geolocation table with random locations for users.

---

### **General Requirements**
- The system should be fully modular and expandable.
- The frontend should be built for maintainability and ease of extension.
- Authentication and state management should be robust.
- The UI should include branding with a logo in the top left corner.
- The admin should have control over managing users, posts, and reports.
- **No shortcuts**—the social network should be as feature-complete as possible with best practices.

---
