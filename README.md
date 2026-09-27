# Discord Ticket Bot

A dedicated ticket bot for Discord — nothing else. Three ways to open a
ticket (button, form, or category-picker dropdown), staff roles with
optional claiming, adding people into a ticket, closing without deleting,
and automatic HTML transcripts sent to a webhook.

## 1. Setup

**Option A — hardcode credentials directly in `index.js`** (recommended for
hosts like Hiden Cloud that only run one start command):

Open `index.js` and fill in the three constants near the top:

```js
const TOKEN = 'your-bot-token-here';
const CLIENT_ID = 'your-application-client-id';
const GUILD_ID = '';   // optional, for instant command sync while testing
```

⚠️ If you do this, **never** share `index.js`, push it to a public repo, or
paste its contents anywhere (including in chat with anyone). Treat the token
like a password — if it ever leaks, reset it immediately in the Developer
Portal. -just9oo

**Option B — use environment variables** (a `.env` file locally, or your
host's environment variable panel):

```bash
npm install
cp .env.example .env
```
```
DISCORD_TOKEN=your-bot-token-here
CLIENT_ID=your-application-client-id
GUILD_ID=
```

Either works — if both are set, the values hardcoded in `index.js` take
priority.

### Discord Developer Portal settings

1. Create an application → Bot → reset/copy the **Token**.
2. Under **Bot**, enable the **Server Members Intent** (privileged intent).
   That's the only one this bot needs — no message content access is
   required, since everything here runs through buttons, dropdowns, and
   forms rather than reading messages.
3. Invite the bot with these scopes/permissions (OAuth2 → URL Generator):
   - Scopes: `bot`, `applications.commands`
   - Permissions: Manage Channels, Manage Roles, View Channels, Send
     Messages, Embed Links, Attach Files (needed for transcripts), Read
     Message History, Mention Everyone (so staff role pings actually notify
     people even if the role isn't set "mentionable").
4. **Important:** in Server Settings → Roles, drag the bot's own role
   **above** any staff role you configure with `/ticket-staff add` — it
   needs to be positioned higher to manage permissions in ticket channels.

## 2. Run the bot

```bash
npm start
```

Slash commands are deployed automatically every time the bot starts — no
separate deploy step needed, which also means this works on hosts that only
let you run one start command with no shell access.

---

## How it works

### 1. Set up the basics
- `/ticket-staff add role:<@role>` for each support role — they'll be
  pinged when a ticket opens and (if you enable claiming) are the only ones
  who can claim.
- `/ticket embed` (optional) — customize the welcome message posted inside
  every new ticket.
- `/transcript set webhook_url:<url>` (optional) — get an HTML transcript
  posted to a webhook every time a ticket closes.
- `/ticket-log set channel:<#channel>` (optional) — get a short log entry
  every time a ticket is closed (who opened it, who closed it, who claimed
  it).
- `/ticket-claim enable` (optional) — require a staff member to explicitly
  claim a ticket before any staff can talk in it; once claimed, only that
  person (not the rest of staff) can reply.

### 2. Pick a panel type and configure it

| Type | Extra setup needed | What the user experiences |
|---|---|---|
| **Regular Button Open** | `/ticket-category add` (one or more categories — auto-overflow when one fills up) | Clicks **Open Ticket** → channel is created immediately |
| **Form Open** | `/ticket-category add` **and** `/ticket-form add` (up to 5 questions) | Clicks **Open Ticket** → a form (modal) pops up asking your questions → on submit, the channel is created with their answers shown in the ticket embed |
| **Sidebar Open** | `/ticket-option add label:<> category:<>` (one entry per option, any number up to 25) | Clicks **Open Ticket** → a dropdown menu appears listing your options → picking one creates the ticket directly in that option's assigned category |

### 3. Post the panel

`/ticket panel channel:<#channel> type:<button|form|sidebar> title:<> description:<> ...`

You can post multiple panels of different types in different channels —
e.g. a Sidebar panel in #support and a Form panel in #bug-reports.

### Closing a ticket

Closing (via the **Close Ticket** button) does **not** delete the channel:
1. The ticket opener, the claimer (if any), and all ticket staff roles lose
   send access — the channel becomes read-only history.
2. The channel is renamed to `closed-<original-name>`.
3. A **Delete Ticket** button appears — only ticket staff or admins can use
   it, and it permanently deletes the channel.
4. If a transcript webhook is configured, an HTML transcript of the whole
   conversation is generated automatically and sent there.
5. If a ticket log channel is configured, the closure is logged there.

## All commands

### Admin setup (Administrator permission required)
| Command | What it does |
|---|---|
| `/ticket-staff add/remove/list` | Manage which roles are ticket staff |
| `/ticket-category add/remove/list` | Manage the category pool for Button/Form panels |
| `/ticket-option add/remove/list` | Manage the dropdown choices (+ their category) for Sidebar panels |
| `/ticket-form add/remove/list/clear` | Manage the questions asked by Form panels |
| `/ticket embed title description [color]` | Set the welcome embed shown inside new tickets |
| `/ticket panel channel type title description ...` | Post a ticket-opening panel |
| `/ticket-claim enable/disable` | Require staff to claim before replying |
| `/transcript set/disable/show` | Configure the webhook that receives closed-ticket transcripts |
| `/ticket-log set/disable/show` | Configure where ticket closures are logged |

### Staff
| Command | What it does |
|---|---|
| `/add user:<@user>` | Add someone into the current ticket (run inside the ticket) |
| **Claim** button | Claims an unclaimed ticket |
| **Close Ticket** button | Locks + renames the ticket, sends the transcript, logs the closure |
| **Delete Ticket** button | Permanently deletes a closed ticket |

### Everyone
Click **Open Ticket** on whichever panel is posted — behavior depends on
that panel's type (see the table above).

## Notes & limitations

- A user can only have **one open ticket at a time**.
  
- All bot data is stored locally as JSON in `src/data/<guildId>.json` — no
  external database required.
  
