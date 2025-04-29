# Supabase Notes Service

A minimal backend service for a personal notes application built with Supabase.

## Overview

This project implements a simple notes service with:
- Supabase database for storing notes
- Edge Functions for API endpoints
- User authentication via Supabase Auth

## Schema Design

The database schema is defined in `schema.sql`:

```sql
create table if not exists notes (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null,
  title text not null,
  content text,
  created_at timestamp with time zone default now()
);
```

### Why this schema?

- **UUID primary key**: Used `gen_random_uuid()` for globally unique IDs without sequence conflicts, making data migration and distribution easier.
- **User association**: Included `user_id` field to associate notes with specific users for proper data isolation.
- **Text content type**: Used Postgres `text` type for unlimited note length rather than `varchar` with length limits.
- **Timestamp with timezone**: Added `created_at` with timezone support for proper time tracking across regions.

## API Endpoints

### Create a Note (POST /notes)

The `post_notes.js` function handles creating new notes:
- Takes `title` and `content` from the request body
- Associates the note with the authenticated user
- Returns the newly created note with status 201

### Get Notes (GET /notes)

The `get_notes.js` function retrieves all notes for the authenticated user:
- No request body parameters needed
- Filters notes by the authenticated user's ID
- Returns an array of note objects

## Setup and Deployment

1. Create a new Supabase project at [supabase.com](https://supabase.com)

2. Set up your local environment:
   ```bash
   npm install
   ```

3. Apply the database schema:
   ```bash
   supabase db push schema.sql
   ```

4. Deploy Edge Functions:
   ```bash
   supabase functions deploy get_notes
   supabase functions deploy post_notes
   ```

5. Set required environment variables:
   ```bash
   supabase secrets set SUPABASE_URL=https://your-project-ref.supabase.co
   supabase secrets set SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
   ```

## Demo

### Create a Note (POST /notes)

```bash
curl -X POST 'https://your-project-ref.functions.supabase.co/post_notes' \
  -H 'Authorization: Bearer YOUR_JWT_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{"title": "Shopping List", "content": "Milk, eggs, bread"}'
```

Expected response (status 201):
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "user_id": "d5fca6fe-9344-4671-99a3-0318b7dce10f",
  "title": "Shopping List",
  "content": "Milk, eggs, bread",
  "created_at": "2025-04-29T12:34:56.789Z"
}
```

### Get Notes (GET /notes)

```bash
curl 'https://your-project-ref.functions.supabase.co/get_notes' \
  -H 'Authorization: Bearer YOUR_JWT_TOKEN'
```

Expected response (status 200):
```json
[
  {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "user_id": "d5fca6fe-9344-4671-99a3-0318b7dce10f",
    "title": "Shopping List",
    "content": "Milk, eggs, bread",
    "created_at": "2025-04-29T12:34:56.789Z"
  },
  {
    "id": "c3d5a4b8-3e44-4eb5-9f8d-66a3c1a70a00",
    "user_id": "d5fca6fe-9344-4671-99a3-0318b7dce10f",
    "title": "Meeting Notes",
    "content": "Discuss project timeline and deliverables",
    "created_at": "2025-04-29T14:22:33.456Z"
  }
]
```

## Security Considerations

- Authentication is enforced on all endpoints
- Row-level security ensures users can only access their own notes
- Service role is used for database operations within edge functions