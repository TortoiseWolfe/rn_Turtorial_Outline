Below is the complete roadmap and system prompt that you requested. This version includes the detailed Phase One MVP roadmap (with completed tasks marked using strike‑through formatting) as well as a refined system prompt that instructs the assistant to take time to think through each step carefully. You can paste this into your LLM chat to help guide your team’s work:

---

# Complete Roadmap

## Phase One MVP Roadmap

### ~~Completed Tasks~~
- ~~**Supabase Backend Setup**~~  
  - ~~SQL commands have been executed to create tables for users (with role), posts, comments, chat messages, and notifications.~~  
  - ~~RLS policies are active on each table.~~  
  - ~~A trigger has been implemented to auto‑create a user record on signup with the first user as `"admin"`.~~

- ~~**Expo Project Setup & Environment Variables**~~  
  - ~~Project created using:~~  
    ```bash
    npx create-expo-app ScriptHammer
    cd ScriptHammer
    npm run reset-project
    rm -rf app-example
    ```  
  - ~~`app.json` is configured with Supabase URL and anon key placeholders.~~

- ~~**Supabase Client & Authentication Context**~~  
  - ~~The client is configured in `lib/supabaseClient.ts` using environment variables.~~  
  - ~~An `AuthProvider` (in `app/context/AuthProvider.tsx`) is in place to manage user state.~~  
  - ~~The global layout (`app/_layout.tsx`) wraps all routes with the AuthProvider.~~

- ~~**Routing with Expo Router (Protected & Admin Groups)**~~  
  - ~~Protected and admin routes have been set up with proper redirection logic to ensure only authenticated users access protected content.~~

- ~~**Authentication Screens (Login & Signup)**~~  
  - ~~Login and Signup screens are fully functional with error handling, loading indicators, and proper redirection.~~

- ~~**Social Network Features**~~  
  - ~~Home feed (`app/(protected)/home.tsx`), Create Post (with image upload), and Chat functionality (with realtime updates) are implemented.~~

- ~~**Admin Dashboard**~~  
  - ~~The admin dashboard (`app/(admin)/dashboard.tsx`) displays users and posts and is restricted to admin users.~~

- ~~**Automated Setup Bash Script**~~  
  - ~~The `setup.sh` script bootstraps the project by creating the project, deleting default files, installing dependencies, creating the folder structure, and writing required files.~~

- ~~**Documentation & Table of Contents**~~  
  - ~~The tutorial is written in Markdown with a clickable Table of Contents using explicit HTML anchor tags for GitHub viewers.~~

- ~~**Consolidated Dependency Installation**~~  
  - ~~The dependency installation commands are now functional as part of the initial setup process.~~

### Pending / To Be Done for Phase One MVP

- **Enhanced Testing & Error Handling**  
  - Develop and integrate unit and integration tests (using Jest or similar) for:
    - Authentication flows  
    - Data fetching (including realtime updates)  
    - API service layer  
  - Refine error handling in API interactions and authentication flows.

- **UI/UX Improvements**  
  - Polish visual design and responsiveness for:
    - Authentication screens (Login/Signup)  
    - Home feed, Create Post, and Chat screens  
    - Admin dashboard layout  
  - Provide clearer loading indicators and refined error messages.

- **Documentation & Troubleshooting Enhancements**  
  - Expand documentation with additional troubleshooting tips (e.g., file overwriting vs. duplication, proper environment variable setup).  
  - Update the README to outline any manual post-setup steps if needed.

---

## Phase Two Roadmap (Future Enhancements)

- **Advanced CRUD Scaffolding Interface**  
  - Develop a dynamic, customizable interface to allow small business owners to create and manage custom CRUD operations.

- **Additional Social Network Features**  
  - Enhance chat functionality with richer features (e.g., message threads, read receipts).  
  - Expand notifications and other social interactions.

- **Analytics & Reporting**  
  - Add analytics dashboards to provide insights on user engagement and system performance.

- **CI/CD Integration & Automated Testing**  
  - Implement continuous integration/continuous deployment pipelines to automate testing and deployment.

- **Performance Optimization & Scalability**  
  - Optimize database queries and API endpoints to better handle increased load.

- **Further UX Refinements**  
  - Continuously improve the user interface based on user feedback, ensuring accessibility and an intuitive experience.

---

# System Prompt for Phase One MVP

Below is the refined system prompt. Paste this into your LLM chat to ensure discussions and further outputs remain focused on completing the Phase One MVP while acknowledging planned future enhancements for Phase Two:

```
SYSTEM PROMPT:

You are an expert React Native and Expo developer building an opinionated framework to help small business owners launch their first mobile app into the iPhone App Store quickly. This framework extends Expo, integrates Supabase for the backend, uses NativeWind for global styling (with dark/light mode toggling), leverages Storybook for component development, and includes a built-in testing framework.

Follow these instructions exactly:

1. **Focus on Phase One MVP:**
   - Work on completing the remaining tasks for Phase One MVP, including:
     - Enhancing testing and error handling with a focus on unit/integration tests and robust API error handling.
     - Polishing UI/UX for authentication screens, social features (home feed, create post, chat), and the admin dashboard.
     - Updating documentation with detailed troubleshooting steps and a comprehensive README outlining any necessary manual post-setup steps.
2. **Do Not Introduce Phase Two Features Yet:**
   - While you are aware that future enhancements (such as advanced CRUD scaffolding, additional social features, analytics, CI/CD integration, and performance optimizations) are planned for Phase Two, the current conversation should remain strictly focused on finalizing the Phase One MVP.
3. **Take Time to Think Step-by-Step:**
   - Do not rush through your response. Carefully consider and articulate your reasoning step-by-step.
   - Before presenting your final solution or code, outline your thought process and reasoning in a methodical, detailed manner.
4. **Project Setup and File Management:**
   - Adhere strictly to the following starting commands:
     ```
     npx create-expo-app ScriptHammer
     cd ScriptHammer
     npm run reset-project
     rm -rf app-example
     ```
   - Replace default files (e.g., `app/index.tsx`) rather than duplicating them.
5. **Environment Variables and Dependencies:**
   - Provide clear and explicit commands for installing any required dependencies.
6. **Realtime and Authentication:**
   - Use the latest Supabase v2 API methods (e.g., `supabase.channel(...)` for realtime updates).
   - Ensure that client-side authentication and navigation (login, signup, protected routes) are correctly implemented.
7. **Table of Contents:**
   - Every tutorial output must begin with a fully functional clickable Table of Contents using Markdown anchor links.

Additional instructions:
- When provided with a GitHub repository link, analyze it to understand its existing architecture and coding conventions.
- Provide complete, functional code blocks rather than pseudocode or placeholders.
- Emphasize best practices, maintainability, code quality, and incremental, test-driven development.
- Clearly differentiate between Phase One MVP tasks and Phase Two future improvements in your output.

End SYSTEM PROMPT.
```

---

This complete roadmap and system prompt should help guide your team in finalizing the Phase One MVP while keeping future Phase Two enhancements on the radar. Take your time to think through each step carefully, ensuring a deliberate, methodical approach to the remaining work. Happy coding!
