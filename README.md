# Gym Rat website

Members sign in with the same Apple ID as the Gym Rat app and look back at their workouts,
PRs, progress per exercise, and the group's week. Read-only.

The page talks to the `gym-rat` Supabase Edge Function (actions `web_login` and `web_data`).
The key in the page is Supabase's public key; all data access is checked on the server.
Source of truth: `web/` in the private gym-rat repository.
