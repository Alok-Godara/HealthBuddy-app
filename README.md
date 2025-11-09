# HealthBuddy (mobile)

HealthBuddy is an Expo + React Native mobile application that provides a simple Electronic Health Record (EHR) experience for patients: authentication, profile management, uploading and viewing medical records, and managing consent for provider access. The app uses Supabase for authentication, storage and database operations and is written in TypeScript.

This repository contains the mobile client built with Expo and `expo-router` structured for a tab-based flow (auth + main tabs).

## Tech stack

- React Native + Expo
- expo-router for file-based routing
- TypeScript
- Supabase (auth, Postgres, storage) via `@supabase/supabase-js`
- Expo libraries: `expo-image`, `expo-image-picker`, `expo-sharing`, `expo-file-system`, `expo-constants`
- UI helpers: `lucide-react-native` icons

## Key features

- Email/password authentication (Supabase)
- Sign up / Login screens
- Profile management (view & edit patient profile)
- Upload, view, share medical records (images/files stored in Supabase Storage)
- Consent / provider access requests UI
- Modular services layer under `Services/` that wraps Supabase operations

## Repository layout (important files)

- `app/` – Expo Router entry, routes and screens. Key routes:
  - `app/(auth)/login.tsx`, `app/(auth)/signup.tsx` – authentication screens
  - `app/(tabs)/upload.tsx`, `app/(tabs)/records.tsx`, `app/(tabs)/profile.tsx`, `app/(tabs)/consent.tsx` – main app tabs
  - `app/_layout.tsx` – root layout (fonts & splash handling)
- `Services/`
  - `Supabase.ts` – Supabase client initialization (uses `AsyncStorage` for session persistence)
  - `AuthService.ts` – signup/login/logout helpers
  - `Services.ts` – higher-level DB/storage helpers (uploads, row insert/update, query helpers)
- `components/` – reusable UI components (`RecordCard`, `ProfileCard`, `ConsentCard`, `SectionHeader`, etc.)
- `constants/` – `Colors.ts`, `Typography.ts`
- `types/medical.ts` – TypeScript interfaces for `MedicalRecord`, `PatientProfile`, `ConsentRequest`

## Environment variables

Create a local `.env` (or set environment variables in your CI/EAS) with the following keys used by `Supabase.ts`:

- `EXPO_PUBLIC_SUPABASE_URL` – your Supabase project URL
- `EXPO_PUBLIC_SUPABASE_ANON_KEY` – your Supabase anon/public key

Note: Keep your service_role key out of the client. Only use anon/public keys in the app.

## Setup (local development)

Prerequisites: Node.js (LTS), Yarn or npm, Expo CLI

1. Install dependencies

```powershell
npm install
# or
yarn install
```

2. Add environment variables (for example, in a `.env` file or in `app.json` / Expo's config)

3. Start the Expo dev server

```powershell
npm run start
# or
yarn start
```

4. Run on a device or simulator via the Expo CLI (see `package.json` scripts for `android`, `ios`, `web`).

## Scripts

Taken from `package.json`:

- `start` – `expo start`
- `android` – `expo start --android`
- `ios` – `expo start --ios`
- `web` – `expo start --web`
- `lint` – `expo lint`

## Supabase notes / DB expectations

- The app queries these tables by name:
  - `patients` – stores patient profile records (the code maps `user.id` from Supabase auth to `patients.id`).
  - `medical_events` – stores metadata about records (type, title, provider, event_date, etc.)
  - `documents` – expected to contain `file_url` for stored files and `medical_event_id` to associate documents to events
  - `provider_patient_access` – provider access / consent relations (used by `Services.getProviderAccessRequests`)

- Storage buckets: the code uses a bucket named `medical_data` (see `Services/` and `app/(tabs)/records.tsx`), and uses `getPublicUrl` to obtain public URLs for files. If you want stricter access, switch to signed URLs or a secure proxy.

## Important implementation notes & TODOs

- AuthService currently writes a plain `password` field into the `patients` table on signup — this is insecure and should be removed. Use server-side hashed passwords or rely solely on Supabase auth.
- Consider adding server-side functions or Postgres Row Level Security (RLS) policies to ensure only authorized users access patient records.
- Add E2E tests for flows: signup/login, upload record, view/share record, consent grant/revoke.
- Add a `.env.example` with the required environment vars and a short guide to set them up.

## Local development tips

- Use the Expo Go app for quick testing on a physical device.
- If testing file uploads and sharing, ensure the Supabase storage bucket CORS and public settings match your intended flow.
- Use the `expo-file-system` and `expo-sharing` flows (already wired in `app/(tabs)/records.tsx`).

## Contributing

Contributions are welcome. Please open issues for bugs or feature requests, and open a PR for changes. Add unit/integration tests where possible.

## License

This project doesn't contain a license file in the repository. Add a `LICENSE` (for example MIT) if you want the project to be open source.

---

If you'd like, I can:

- add a `.env.example` file with placeholder values
- add a short checklist for Supabase schema and storage setup (SQL snippets)
- add a basic CI workflow for running TypeScript checks and linting

Which of those shall I do next?
