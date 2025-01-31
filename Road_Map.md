# ScriptHammer Roadmap (Struck-Through for Completed)

1. ~~Fake tokens + ephemeral sign-up~~  
2. ~~Add real Supabase auth + RLS~~  
3. ~~DB triggers for automatic profile creation~~  
4. ~~Minimal token storage (SecureStore or localStorage)~~  
5. ~~Real-time subscription for user’s profile~~  
6. ~~Offline/local-first approach (Dexie for web, SQLite for native)~~  

---

## Next Phases:

7. **Admin Dashboard** (web)  
8. **CLI Scaffolding** (like Bonfire’s Builder)  
9. **Advanced Roles/Permissions** (admin, moderator)  
10. **Social/Business Features** (posts, file uploads, notifications)  
11. **Production Pipeline** (Expo EAS, env vars, logging, monitoring)

Below, each phase is described in detail, including **why it matters**, **potential objectives**, and a **System Prompt Code** snippet. 

When you’re ready for a phase, start a **fresh** ChatGPT conversation, paste the snippet as your **system** message, and ask for the tutorial in your **user** message. The tutorial **must**:

1. **Integrate** with the existing code from previous phases (no separate or appended code).  
2. **Provide fully functional code blocks** (no placeholders, no missing imports).  
3. **End with a Node script** that **maintains parity** with the current iteration—i.e., it merges or updates *existing* files and structures appropriately, rather than merely adding new files in isolation.

---

## **Phase 7** – **Admin Dashboard** (Web Interface)

### Potential Objectives
1. **User Management**: view/search/disable accounts (from the same Supabase DB).  
2. **Content Moderation**: e.g. edit or remove user posts.  
3. **Analytics**: basic usage stats, sign-ups per day.

### Why It Matters
- A web-based admin panel is easier for staff to manage user data, moderate content, etc.

### **System Prompt Code (Phase 7)**

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

*(Start a new ChatGPT conversation, paste the code as the **System** message, then in your **User** message, request the tutorial. ChatGPT’s reply should be a single integrated set of instructions plus a Node script that merges everything.)*

---

## **Phase 8** – **CLI Scaffolding Tool** (Like Bonfire’s “Builder”)

### Potential Objectives
1. **Command-Line Interface** – e.g. `npx script-hammer generate module posts`.  
2. **Auto-Generate** – React Native screens, Supabase table creation scripts, context files.  
3. **Configurable** – maybe a YAML/JSON config describing module fields.

### Why It Matters
- CRUD scaffolding drastically accelerates development and fosters consistency.

### **System Prompt Code (Phase 8)**

```
You are a senior Expo/React Native developer focusing on Phase 8 of ScriptHammer: “CLI Scaffolding Tool.”

I need a fully functional tutorial (no placeholders) showing how to:
1. Build a Node-based CLI that generates common files (RN screens, Supabase table scripts).
2. Follow best practices so new modules integrate seamlessly into ScriptHammer’s existing codebase.
3. End with a Node script that merges or updates existing logic, ensuring full parity with the current iteration.

IMPORTANT:
- Merge with the existing code from Phases 1-7 (including the admin panel).
- Provide complete code blocks with no placeholders or missing imports.
- The final Node script must incorporate the new logic from this phase in a way that does not overwrite or ignore previous phases. It should update or add only what’s necessary to integrate seamlessly.
```

---

## **Phase 9** – **Advanced Roles & Permissions**

### Potential Objectives
1. **Supabase Policies** for role-based access (admin, moderator).  
2. **UI** changes: hide or show features unless user is admin or moderator.  
3. Possibly store roles in a separate table or within `profiles`.

### Why It Matters
- Real-world apps need more than “everyone can do everything.”

### **System Prompt Code (Phase 9)**

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

---

## **Phase 10** – **Social/Business Features** (Posts, Uploads, Notifications)

### Potential Objectives
1. **Posts & Feeds** – create a `posts` table in Supabase, show a feed.  
2. **File Uploads** – e.g. images or docs to Supabase Storage or 3rd-party.  
3. **Notifications** – push notifications (Expo or FCM) or in-app real-time.  
4. **Realtime** – feed updates automatically.

### Why It Matters
- Grows your project from a simple profile system to a more robust social/business platform.

### **System Prompt Code (Phase 10)**

```
You are a senior Expo/React Native developer focusing on Phase 10 of ScriptHammer: “Social/Business Features (Posts, Uploads, Notifications).”

I need a fully functional tutorial (no placeholders) demonstrating:
1. A “posts” feed with Supabase (integrated with existing offline logic if feasible).
2. File uploads (images/docs) via Supabase Storage or a 3rd-party.
3. Optional push notifications for new likes/comments.
4. Ends with a Node script that merges new modules/tables into the existing code in a way that preserves parity.

IMPORTANT:
- Merge with the existing code from Phases 1-9 (including admin, roles, etc.).
- Provide complete code blocks with no missing imports or placeholders.
- Show how these features integrate with existing contexts/offline logic.
- The final script updates or adds only what’s needed to seamlessly incorporate these features.
```

---

## **Phase 11** – **Production Pipeline** (Expo EAS, Env Vars, Logging, Monitoring)

### Potential Objectives
1. **EAS Build** for iOS/Android.  
2. **Environment Variables** for secrets.  
3. **Logging/Monitoring** with Sentry or Bugsnag.  
4. **Performance** – code splitting, caching, etc.

### Why It Matters
- A stable release pipeline ensures quick updates, environment configs, logging, etc.

### **System Prompt Code (Phase 11)**

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

## **How to Use These System Prompt Blocks**

1. **Start a fresh ChatGPT conversation** for whichever phase (7–11) you’re implementing.  
2. Copy that phase’s **System Prompt Code** (including triple backticks) into your **System** message.  
3. In your **User** message, add any special requests or details.  
4. ChatGPT should then respond with **fully merged code** plus a **Node script** that updates or adds the new logic without disrupting prior phases.

This ensures each **phase** is a **progressive** iteration, thoroughly integrated with previously existing code—**never** an afterthought or separate extension. 

---

### Final Note

With **Phases 1–6** finished (Dexie on Web, SQLite + SecureStore on Native, real-time from Supabase, RLS, etc.), you have a solid foundation. The next steps are:

- **Phase 7**: Admin Dashboard  
- **Phase 8**: CLI Scaffolding  
- **Phase 9**: Advanced Roles/Permissions  
- **Phase 10**: Social/Business expansions  
- **Phase 11**: Production pipeline  

At each **new** phase, use the **System Prompt Code** above so ChatGPT produces a **fully integrated** tutorial (with a final **Node scaffolding script** that merges seamlessly into your current iteration). Happy building with ScriptHammer!
