```sql
-- =============================================================================
-- Production-Ready SQL Schema for SupaBase Backend (Using CamelCase for displayName)
-- =============================================================================
-- This script creates the necessary tables and view to support:
--   1. Authentication & Profiles (using SupaBase Auth)
--   2. Posts
--   3. Conversations & Messages (Chat System)
--   4. A public "users" view for front-end queries
--
-- IMPORTANT:
-- - Ensure your SupaBase project has the "pgcrypto" extension enabled for
--   gen_random_uuid() to work.
-- - This version uses a quoted "displayName" column in the profiles table to match
--   your front-end expectations. Use this approach if you wish to work with camelCase.
-- =============================================================================

-- Enable extension for UUID generation (if not already enabled)
CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- ---------------------------------------------------------------------------
-- 1. Profiles Table: Stores user details (avatar, displayName, bio)
-- ---------------------------------------------------------------------------
CREATE TABLE public.profiles (
    id uuid PRIMARY KEY,
    avatar text,
    "displayName" text,
    bio text,
    created_at timestamptz NOT NULL DEFAULT now(),
    updated_at timestamptz NOT NULL DEFAULT now(),
    CONSTRAINT fk_profiles_users FOREIGN KEY (id)
        REFERENCES auth.users (id) ON DELETE CASCADE
);

-- ---------------------------------------------------------------------------
-- 2. Posts Table: Stores posts created by users
-- ---------------------------------------------------------------------------
CREATE TABLE public.posts (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id uuid NOT NULL,
    content text NOT NULL,
    created_at timestamptz NOT NULL DEFAULT now(),
    updated_at timestamptz NOT NULL DEFAULT now(),
    CONSTRAINT fk_posts_user FOREIGN KEY (user_id)
        REFERENCES public.profiles (id) ON DELETE CASCADE
);

-- Index to speed up lookups by user_id
CREATE INDEX idx_posts_user_id ON public.posts(user_id);

-- ---------------------------------------------------------------------------
-- 3. Conversations Table: Represents chat sessions between two users
-- ---------------------------------------------------------------------------
CREATE TABLE public.conversations (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    user1 uuid NOT NULL,
    user2 uuid NOT NULL,
    created_at timestamptz NOT NULL DEFAULT now(),
    CONSTRAINT fk_conversations_user1 FOREIGN KEY (user1)
        REFERENCES public.profiles (id) ON DELETE CASCADE,
    CONSTRAINT fk_conversations_user2 FOREIGN KEY (user2)
        REFERENCES public.profiles (id) ON DELETE CASCADE,
    CONSTRAINT chk_different_users CHECK (user1 <> user2)
);

-- Indexes for efficient querying
CREATE INDEX idx_conversations_user1 ON public.conversations(user1);
CREATE INDEX idx_conversations_user2 ON public.conversations(user2);

-- ---------------------------------------------------------------------------
-- 4. Messages Table: Stores individual messages within a conversation
-- ---------------------------------------------------------------------------
CREATE TABLE public.messages (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    conversation_id uuid NOT NULL,
    sender_id uuid NOT NULL,
    content text NOT NULL,
    created_at timestamptz NOT NULL DEFAULT now(),
    updated_at timestamptz NOT NULL DEFAULT now(),
    CONSTRAINT fk_messages_conversation FOREIGN KEY (conversation_id)
        REFERENCES public.conversations (id) ON DELETE CASCADE,
    CONSTRAINT fk_messages_sender FOREIGN KEY (sender_id)
        REFERENCES public.profiles (id) ON DELETE CASCADE
);

-- Indexes for faster querying in messages
CREATE INDEX idx_messages_conversation_id ON public.messages(conversation_id);
CREATE INDEX idx_messages_sender_id ON public.messages(sender_id);

-- ---------------------------------------------------------------------------
-- Trigger Function: Automatically update the 'updated_at' column on updates
-- ---------------------------------------------------------------------------
CREATE OR REPLACE FUNCTION public.update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Triggers for auto-updating the updated_at column in relevant tables
CREATE TRIGGER update_profiles_updated_at
BEFORE UPDATE ON public.profiles
FOR EACH ROW
EXECUTE PROCEDURE public.update_updated_at_column();

CREATE TRIGGER update_posts_updated_at
BEFORE UPDATE ON public.posts
FOR EACH ROW
EXECUTE PROCEDURE public.update_updated_at_column();

CREATE TRIGGER update_messages_updated_at
BEFORE UPDATE ON public.messages
FOR EACH ROW
EXECUTE PROCEDURE public.update_updated_at_column();

-- ---------------------------------------------------------------------------
-- 5. Public Users View: Joins auth.users with profiles to facilitate queries
-- ---------------------------------------------------------------------------
CREATE OR REPLACE VIEW public.users AS
SELECT
    u.id,
    u.email,
    p.avatar,
    p."displayName",
    p.bio,
    p.created_at,
    p.updated_at
FROM auth.users u
LEFT JOIN public.profiles p ON u.id = p.id;

-- =============================================================================
-- Reminder:
-- - Verify that these schema changes integrate seamlessly with your existing system.
-- - Test all functionality (authentication, posting, messaging, and profile management)
--   in a staging environment before deploying to production.
-- =============================================================================
```
