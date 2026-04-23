# Filing Database

Filing Database is a lightweight web app for tracking company document filings — articles of incorporation, bylaws, board minutes, annual reports, SEC filings, and tax filings. Search by title or reference number, filter by company and status, upload PDFs, and share signed URLs. Built with Next.js 14 and Supabase (Postgres + Storage).

## Features

- Dashboard with counts + 5 most recent filings
- Filings table with search (title / reference # / description), company filter, and status filter
- Create / edit / delete filings, including document uploads to a private Supabase Storage bucket
- Create / edit / delete companies (filings cascade-delete with their company)
- Seeded filing-type lookup (Articles of Incorporation, Annual Report, Board Minutes, 10-K, etc.)
- Server components + server actions — no custom REST API needed; Supabase's PostgREST handles queries
- Row Level Security enabled with permissive starter policies you can tighten once you add auth

## Quick start

### 1. Create a Supabase project

1. Go to <https://supabase.com/dashboard> and create a new project.
2. In **Project Settings → API**, copy:
   - **Project URL** → `NEXT_PUBLIC_SUPABASE_URL`
   - **Project API keys → anon public** → `NEXT_PUBLIC_SUPABASE_ANON_KEY`

### 2. Run the schema

Open the Supabase dashboard **SQL Editor** and run the contents of
[`supabase/migrations/20260423000000_init.sql`](./supabase/migrations/20260423000000_init.sql).
This creates the `companies`, `filing_types`, `filings`, `filing_tags`, and
`filings_tags` tables, seeds ~13 common filing types, creates the
private `filing-documents` storage bucket, and enables RLS with permissive
starter policies.

Optionally run [`supabase/seed.sql`](./supabase/seed.sql) to load sample
companies and filings.

If you use the [Supabase CLI](https://supabase.com/docs/guides/cli) instead:

```bash
supabase link --project-ref YOUR-PROJECT-REF
supabase db push
```

### 3. Configure the web app

```bash
cp .env.local.example .env.local
# then edit .env.local with your real NEXT_PUBLIC_SUPABASE_URL and NEXT_PUBLIC_SUPABASE_ANON_KEY
npm install
npm run dev
```

Visit <http://localhost:3000>.

## Schema overview

| Table          | Purpose                                                             |
| -------------- | ------------------------------------------------------------------- |
| `companies`    | The legal entities whose filings you're tracking.                   |
| `filing_types` | Seeded lookup (Articles of Incorporation, 10-K, Tax Filing, etc.).  |
| `filings`      | Individual filings — title, dates, status, reference #, document.   |
| `filing_tags`  | Optional free-form tags.                                            |
| `filings_tags` | Many-to-many join between filings and tags.                         |

Uploaded documents live in the `filing-documents` Storage bucket, keyed as
`{company_id}/{uuid}.{ext}`. The app shows them via short-lived signed URLs
so the bucket stays private.

## Project layout

```
supabase/
  migrations/20260423000000_init.sql   # schema + RLS + seed filing types
  seed.sql                             # optional sample data
src/
  app/
    layout.tsx                         # nav shell + setup banner
    page.tsx                           # dashboard
    actions.ts                         # server actions (CRUD + uploads)
    filings/
      page.tsx                         # list + filters
      new/page.tsx                     # create form
      [id]/page.tsx                    # detail + edit + delete
    companies/
      page.tsx                         # CRUD table
  components/
    FilingForm.tsx                     # create/edit filing
    CompanyList.tsx                    # CRUD UI for companies
    DeleteButton.tsx                   # shared destructive action button
    SetupBanner.tsx                    # banner when Supabase env is missing
  lib/
    supabase/client.ts                 # browser client
    supabase/server.ts                 # server client (RSC / actions)
    supabase/env.ts                    # env presence check
    types.ts                           # typed Database schema
    format.ts                          # date + status badge helpers
```

## Security

The starter RLS policies are **permissive** — anyone with the anon key can
read and write. Before putting this in front of real users:

1. Add Supabase Auth (e.g. magic links).
2. Replace the `using (true)` / `with check (true)` policies with ones
   scoped to `auth.role() = 'authenticated'` or tenant / owner columns.
3. Decide whether the storage bucket should stay private (it is by default)
   and lock down the storage policies the same way.

## Scripts

```bash
npm run dev      # start the dev server
npm run build    # production build
npm run start    # run the built app
npm run lint     # next lint
```

## License

MIT
