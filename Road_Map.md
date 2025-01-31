Below is an updated roadmap that better reflects your iterative, merge‐and‐update workflow. The key change is that every new phase is no longer treated as an “extension” appended at the end, but as an integrated iteration that merges seamlessly into your existing codebase. In addition, the system prompt snippets now explicitly instruct ChatGPT to update or merge with existing files instead of simply adding new steps. You can use the roadmap below as a guide for future phases and to inform the system prompts you share.

---

# ScriptHammer Roadmap (Iterative & Merged Approach)

1. ~~Fake tokens + ephemeral sign-up~~  
2. ~~Add real Supabase auth + RLS~~  
3. ~~DB triggers for automatic profile creation~~  
4. ~~Minimal token storage (SecureStore or localStorage)~~  
5. ~~Real-time subscription for user’s profile~~  
6. ~~Offline/local-first approach (Dexie for web, SQLite for native)~~

---

## Next Phases (All phases must be merged into the existing codebase, updating files rather than simply appending new ones):

### **Phase 7 – Admin Dashboard (Web Interface)**
  
**Potential Objectives:**
- **User Management:** View, search, and disable accounts from the Supabase DB.
- **Content Moderation:** Edit or remove user posts.
- **Analytics:** Basic usage stats, sign-ups per day.

**Why It Matters:**
- A web-based admin panel simplifies management of user data and content moderation.

**System Prompt Code (Phase 7):**

```
You are a senior Expo/React Native developer focusing on Phase 7 of ScriptHammer: “Admin Dashboard (Web Interface).”

I need a fully functional tutorial (no placeholders) showing how to build a web-based admin panel that:
1. Integrates with our Supabase instance for user management and content moderation.
2. Follows best practices (secure routes, role checks, etc.).
3. Ends with a Node script that merges or updates existing files to maintain parity with the current iteration.

IMPORTANT:
- Merge this admin panel code into the existing code from Phases 1-6 (Dexie, SQLite, RLS, triggers, etc.).
- Provide complete code blocks with no missing imports.
- The tutorial must NOT treat the admin panel as an afterthought; it must be woven into the existing structure rather than appended at the end.
- The final Node script must preserve the existing architecture and update or add only what’s needed to integrate this feature.
```

### **Phase 8 – CLI Scaffolding Tool (Like Bonfire’s Builder)**
  
**Potential Objectives:**
- **Command-Line Interface:** e.g., `npx script-hammer generate module posts`.
- **Auto-Generate:** React Native screens, Supabase table creation scripts, context files.
- **Configurable:** Use a YAML/JSON configuration to describe module fields.

**Why It Matters:**
- CRUD scaffolding accelerates development and ensures consistency across modules.

**System Prompt Code (Phase 8):**

```
You are a senior Expo/React Native developer focusing on Phase 8 of ScriptHammer: “CLI Scaffolding Tool.”

I need a fully functional tutorial (no placeholders) showing how to:
1. Build a Node-based CLI that generates common files (RN screens, Supabase table scripts).
2. Follow best practices so new modules integrate seamlessly into ScriptHammer’s existing codebase.
3. End with a Node script that merges or updates existing logic, ensuring full parity with the current iteration.

IMPORTANT:
- Merge with the existing code from Phases 1-7 (including the admin panel).
- Provide complete code blocks with no placeholders or missing imports.
- The final Node script must incorporate the new logic from this phase in a way that does not overwrite or ignore previous phases.
```

### **Phase 9 – Advanced Roles & Permissions**

**Potential Objectives:**
- **Supabase Policies:** Expand RLS with roles such as admin and moderator.
- **UI Changes:** Hide or show features based on roles.
- **Data Model:** Optionally store roles in a separate table or extend `profiles`.

**Why It Matters:**
- Real-world applications need granular control; not every user should have full permissions.

**System Prompt Code (Phase 9):**

```
You are a senior Expo/React Native developer focusing on Phase 9 of ScriptHammer: “Advanced Roles & Permissions.”

I need a fully functional tutorial (no placeholders) that:
1. Extends our Supabase RLS with roles like admin and moderator.
2. Shows how the front end checks these roles to hide or show certain features (merging into the existing admin panel).
3. Concludes with a Node script that merges or updates existing files/policies so everything remains in sync.

IMPORTANT:
- Merge with the existing code from Phases 1-8 (including the admin panel).
- No placeholders or truncated imports; everything is integrated into the existing structure (contexts, DB triggers, etc.).
- The scaffolding script should update or add what’s needed for these new permissions, preserving the current iteration’s integrity.
```

### **Phase 10 – Social/Business Features (Posts, Uploads, Notifications)**

**Potential Objectives:**
- **Posts & Feeds:** Create a `posts` table in Supabase, show a dynamic feed.
- **File Uploads:** Integrate file uploads to Supabase Storage or a third-party service.
- **Notifications:** Implement push notifications or in-app notifications.
- **Realtime:** Automatically update the feed as posts are added or updated.

**Why It Matters:**
- Elevates your project from a simple profile system to a robust social/business platform.

**System Prompt Code (Phase 10):**

```
You are a senior Expo/React Native developer focusing on Phase 10 of ScriptHammer: “Social/Business Features (Posts, Uploads, Notifications).”

I need a fully functional tutorial (no placeholders) demonstrating:
1. A “posts” feed with Supabase (integrated with existing offline logic if feasible).
2. File uploads (images/docs) via Supabase Storage or a third-party.
3. Optional push notifications for new likes/comments.
4. Ends with a Node script that merges new modules/tables into the existing code in a way that preserves parity.

IMPORTANT:
- Merge with the existing code from Phases 1-9 (including admin, roles, etc.).
- Provide complete code blocks with no missing imports or placeholders.
- Show how these features integrate with existing contexts/offline logic.
- The final script updates or adds only what’s needed to seamlessly incorporate these features.
```

### **Phase 11 – Production Pipeline (Expo EAS, Env Vars, Logging, Monitoring)**

**Potential Objectives:**
- **Expo EAS Build:** Prepare builds for iOS and Android.
- **Environment Variables:** Securely manage secrets.
- **Logging/Monitoring:** Integrate services like Sentry or Bugsnag.
- **Performance:** Code splitting, caching, etc.

**Why It Matters:**
- A stable release pipeline is essential for production, ensuring updates, secure configurations, and robust monitoring.

**System Prompt Code (Phase 11):**

```
You are a senior Expo/React Native developer focusing on Phase 11 of ScriptHammer: “Production Pipeline (Expo EAS, Env Vars, Logging, Monitoring).”

I need a fully functional tutorial (no placeholders) showing how to:
1. Use Expo EAS to build/submit iOS and Android apps.
2. Manage environment variables securely.
3. Integrate logging or crash reporting (Sentry, Bugsnag) in production.
4. Provide a Node script that updates or adds the necessary files to maintain parity with the current iteration’s pipeline.

IMPORTANT:
- Merge with the existing code from Phases 1-10.
- Provide complete code blocks, ensuring the final code is fully integrated.
- The final script must not overwrite or ignore prior logic. It should carefully merge these pipeline enhancements into the existing structure.
```

---

## How to Use These System Prompt Blocks

1. **Start a fresh ChatGPT conversation** for the phase you’re implementing.  
2. Copy the phase’s **System Prompt Code** (including the triple backticks) into your **System** message.  
3. In your **User** message, add any additional details if needed.  
4. ChatGPT should then reply with a fully integrated tutorial that merges into the existing codebase along with a Node script that updates only what is necessary.

---

## Final Note

With **Phases 1–6** completed (including Dexie on Web, SQLite + SecureStore on Native, real-time updates from Supabase, and our offline-first approach), you have a strong foundation. The subsequent phases will iteratively merge new features into this codebase rather than adding separate blocks. This approach ensures that every enhancement (admin dashboard, CLI scaffolding, advanced roles, social features, and production pipeline) is fully integrated and maintains parity with your existing system.

Happy building with ScriptHammer!

---

This updated roadmap should guide future iterations and help ensure that every phase is merged into the current codebase instead of being tacked on as a separate step.
