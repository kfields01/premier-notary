# /add-agent — Scaffold a new AI monitoring agent

Adds a fully wired agent end-to-end: collector function, COLLECTORS map, DB row, cron job, sidebar nav entry, squad page role, and router route.

---

## Usage

```
/add-agent <slug> "<Display Name>" "<mission one-liner>" <schedule_cron>
```

Example: `/add-agent marketing "Marketing" "Monitors email campaigns and social scheduling health" "0 */8 * * *"`

---

## Step-by-step checklist

### Step 1 — Send ONE message to Lovable covering ALL code changes

Project ID: `fce70c2a-5da6-4929-8f46-7cab89e64261`

The single Lovable message must include all four of these code changes:

#### A. Add `collectXxx()` function to `supabase/functions/agent-run/index.ts`

Insert before the `const COLLECTORS` map. Follow this template:
```typescript
// <Display Name> — <one-line description>
async function collect<PascalSlug>() {
  const now = new Date();
  const since7d = new Date(now.getTime() - 7 * 86400000).toISOString();
  const since30d = new Date(now.getTime() - 30 * 86400000).toISOString();
  const as_of = now.toISOString();

  const [/* query results */] = await Promise.all([
    // Add relevant DB queries here
    // Use admin.from("table_name").select("...").limit(N)
  ]);

  return {
    as_of,
    note: "<Display Name> monitors <what it watches>.",
    <slug>_analysis_guidance: [
      // 8-12 rules telling the LLM EXACTLY when to alert vs stay silent
      // Must include: which classifications are allowed, what data thresholds trigger each
      // Must forbid: vague generic alerts, invented URLs, duplicate alerts
      "action_url for ALL <slug> alerts MUST be one of these real admin pages only: '/admin/...' — never invent paths.",
    ],
  };
}
```

#### B. Add to COLLECTORS map in `supabase/functions/agent-run/index.ts`

```typescript
"<slug>": collect<PascalSlug>,
```

#### C. Add nav entry to `src/components/AdminSidebar.tsx` in `squadItems`

```typescript
{ title: "<Display Name>", url: "/admin/squad/<slug>", icon: <IconName> },
```

Choose an appropriate lucide-react icon. Import it if not already imported.

#### D. Add role to `src/pages/AdminSquadMember.tsx`

1. Add `"<slug>"` to the `SquadRole` type union
2. Add entry to `ROLES` object:
```typescript
  "<slug>": {
    name: "<Display Name> Assistance",
    title: "<short subtitle>",
    eyebrow: "Assistance · <Category>",
    icon: <IconName>,  // same icon as sidebar
    mission: "<2-3 sentence mission statement>",
    responsibilities: [
      // 4-6 bullet responsibilities
    ],
    pipelines: [
      // 2-3 "A → B → C" pipeline strings
    ],
    tools: [
      // relevant admin page links
      { title: "...", url: "/admin/...", icon: <Icon>, description: "..." },
    ],
  },
```

#### E. Add route to `src/App.tsx`

After existing `squad/*` routes:
```tsx
<Route path="squad/<slug>" element={<AdminSquadMember role="<slug>" />} />
```

---

### Step 2 — Insert DB row into `agents` table

Use `mcp__Lovable__query_database` with project_id `fce70c2a-5da6-4929-8f46-7cab89e64261`:

```sql
INSERT INTO agents (slug, name, description, model, schedule_cron, enabled, action_url)
VALUES (
  '<slug>',
  '<Display Name> Assistance',
  '<mission one-liner>',
  'claude-opus-4-8',
  '<cron_expression>',
  true,
  '/admin/squad/<slug>'
)
ON CONFLICT (slug) DO UPDATE SET
  name = EXCLUDED.name,
  description = EXCLUDED.description,
  schedule_cron = EXCLUDED.schedule_cron,
  action_url = EXCLUDED.action_url;
```

---

### Step 3 — Register cron job

Use `mcp__Lovable__query_database`:

```sql
SELECT cron.schedule(
  'agent-auto-<slug>',
  '<cron_expression>',
  $$
  SELECT net.http_post(
    url := (SELECT value FROM app_settings WHERE key = 'SUPABASE_EDGE_URL') || '/functions/v1/agent-run',
    headers := jsonb_build_object(
      'Content-Type', 'application/json',
      'Authorization', 'Bearer ' || (SELECT value FROM app_settings WHERE key = 'SUPABASE_ANON_KEY'),
      'x-cron-secret', (SELECT value FROM app_settings WHERE key = 'CRON_SECRET')
    ),
    body := jsonb_build_object('slug', '<slug>')
  );
  $$
);
```

If that cron format doesn't work (some projects use a simpler format), check existing cron jobs first:
```sql
SELECT jobname, schedule, command FROM cron.job ORDER BY jobid DESC LIMIT 5;
```
Then match the format of the existing working jobs exactly.

---

## Key constraints — memorize these

- **Never query `amount_money`** — use `amount_cents` (integer) on `square_payments`
- **Never query `quick_intake_leads` for `service_type` or `urgency`** — those columns don't exist
- **`agents` table columns**: `slug`, `name`, `description`, `model`, `schedule_cron`, `enabled`, `action_url` — NOT `schedule` or `is_active`
- **All code changes go through Lovable** (`mcp__Lovable__send_message`) — never edit local files directly
- **All DB changes go through `mcp__Lovable__query_database`** — never use `mcp__Supabase__execute_sql`
- **Post-LLM guards** exist for `content`, `devops`, `research` slugs — check if the new agent needs one too
- **action_url** in guidance rules must only point to REAL existing admin pages

## Standard cron schedules (pick one)

| Frequency | Cron |
|---|---|
| Every 4 hours | `0 */4 * * *` |
| Every 6 hours | `0 */6 * * *` |
| Every 8 hours | `0 */8 * * *` |
| Every 12 hours | `0 */12 * * *` |
| Daily 6am CST | `0 11 * * *` |
| Weekdays 6am CST | `0 11 * * 1-5` |

## DB tables available for collectors

- `appointments` — booking data, review_request_sent_at, payment_status
- `square_payments` — amount_cents, status (COMPLETED/FAILED), buyer_email
- `square_refunds` — refund_id, amount_cents
- `admin_unpaid_bookings` — view: balance_due_cents
- `admin_stale_checkouts` — view: checkout_created_at
- `quick_intake_leads` — lead_code, service_interest, status, source_type
- `lead_sequence_enrollments` — drip campaign status
- `email_send_queue` — scheduled emails
- `sales_leads` — B2B pipeline
- `outreach_leads` — outbound outreach status
- `google_reviews` / `google_reviews_individual` — review data
- `blocked_time_slots` — schedule blocks
- `phone_messages` — voicemail/callback log
- `cron.job` — scheduled job status (via `admin.rpc("get_cron_job_status")`)
