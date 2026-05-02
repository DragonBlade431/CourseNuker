# CourseNuker ⚔️

> Nuke your VTU course lectures. Blood for the GPA god.

**CourseNuker** automates marking VTU online course lectures as complete — parallel processing, smart retries, real-time progress, and a clean web UI.

*"Not even close, baby."* — The blade never dies.

---

## Features ✨

- ⚡ **Parallel Processing** — Multiple lectures at once (configurable batch size)
- 🔄 **Intelligent Retry Logic** — Auto session refresh; failed lectures are retried with clear reasons
- 📊 **Real-Time Progress** — Server-Sent Events (SSE) for live updates
- 🎯 **Job Queue** — Multiple jobs queued and processed with concurrency control
- 🔐 **Session Management** — Auto re-authentication on 401/419/403
- 📈 **Statistics** — Redis-backed analytics (optional)
- 🖥️ **Web UI + CLI + REST API**

---

## 🏠 Run It Yourself (Service down? No problem.)

> **The hosted service has limited capacity and may occasionally be unavailable.**  
> If it's down — don't wait. You have Git and Node. Run it locally in under 2 minutes.

### Requirements

- [Git](https://git-scm.com/downloads) (to clone)
- [Node.js 18+](https://nodejs.org/) (LTS recommended)
- Your VTU account credentials

### Steps

**1. Clone and install**
```bash
git clone https://github.com/DragonBlade431/CourseNuker.git
cd CourseNuker
npm install
```

**2. Start the local server**
```bash
npm run serve
```

**3. Open in browser**
```
http://localhost:3000
```

That's it. The web UI is identical to the hosted version — enter your credentials, paste your course slug, and hit go. Nothing is stored anywhere; credentials are only used in memory for the duration of the job.

> **Finding your course slug:** Go to your VTU course page. The slug is the last part of the URL, e.g. `https://online.vtu.ac.in/courses/1-social-networks` → slug is `1-social-networks`.

### No Redis needed

Redis is only used for the public hosted statistics counter. Running locally works perfectly without it — just skip any `KV_REST_API_*` env vars.

---

## How It Works

```
1. Infiltrate     → Authenticates with VTU, stores session cookie
2. Scout          → Lists all lectures across all modules
3. Execute        → Sends progress updates in parallel batches
4. Retry          → Re-attempts any lectures that survived the first wave
5. GG EZ          → Counts completed vs skipped, explains every skip
```

### Skip reasons

Every skipped lecture tells you exactly why:

| Status | Reason | Retried? |
|--------|--------|----------|
| `skip` | Zero duration — VTU has no content here | No — permanent data issue on VTU's side |
| `maxed` | Didn't reach 100% within the attempt limit | Yes — retried once |
| `error` | Network or server error during request | Yes — retried once |

---

## Usage

### Web UI *(recommended)*
```bash
npm run serve
# Open http://localhost:3000
```
Fill in email, password, course slug → submit → watch the live log.

### CLI *(requires `.env`)*
```bash
cp .env.example .env   # fill in VTU_EMAIL, VTU_PASSWORD, VTU_COURSE_SLUG
npm start
```

### Dev mode (auto-reload)
```bash
npm run dev
```

---

## Configuration Reference

| Variable | Default | Required For | Description |
|----------|---------|--------------|-------------|
| `VTU_EMAIL` | — | CLI only | VTU account email |
| `VTU_PASSWORD` | — | CLI only | VTU account password |
| `VTU_COURSE_SLUG` | `1-social-networks` | CLI only | Course URL slug |
| `VTU_BATCH_SIZE` | `10` | Optional | Lectures processed in parallel per batch |
| `VTU_MAX_ATTEMPTS` | `50` | Optional | Max progress-push attempts per lecture |
| `PORT` | `3000` | Optional | Server port |
| `MAX_CONCURRENT` | `2` | Optional | Max concurrent jobs (hosted only) |
| `KV_REST_API_URL` | — | Optional | Upstash Redis URL (statistics only) |
| `KV_REST_API_TOKEN` | — | Optional | Upstash Redis token |

> **Web server / REST API:** credentials are passed in the request body — no `.env` needed.  
> **CLI:** credentials must be in `.env`.

---

## REST API

**POST** `/api/jobs` — Submit a job
```json
{
  "email": "you@gmail.com",
  "password": "yourpassword",
  "courseSlug": "1-social-networks",
  "batchSize": 10,
  "maxAttempts": 50
}
```

**GET** `/api/jobs/:jobId` — Poll job state

**GET** `/api/jobs/:jobId/stream` — SSE stream (real-time events)

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `Login failed` | Check your VTU credentials — same ones you use on the website |
| `Course not found` | Verify the slug from the VTU URL (e.g. `1-social-networks`) |
| Lectures stuck at `maxed` | VTU API may be throttling — try a smaller `batchSize` (e.g. `5`) or increase `maxAttempts` |
| `Network error` / `ECONNRESET` | Transient VTU outage — these are auto-retried; if persistent, try again later |
| Port 3000 already in use | Set `PORT=3001` in your environment before running |
| Hosted site down | **Run it locally** — see [Run It Yourself](#-run-it-yourself-service-down-no-problem) above |

---

## Project Structure

```
CourseNuker/
├── automation.js          # Core automation engine
├── server.js              # Express server + job queue + SSE
├── index.js               # CLI entry point
├── lib/
│   └── redis.js           # Redis client & statistics helpers
├── frontend/
│   └── index.html         # Web dashboard (served at /)
├── public/
│   └── index.html         # Static fallback
├── package.json
└── .env.example           # Configuration template
```

---

## Tech Stack

| Layer | Technology |
|-------|-----------| 
| Runtime | Node.js |
| Server | Express |
| HTTP Client | Axios + tough-cookie |
| Real-Time | Server-Sent Events (SSE) |
| Stats (optional) | Upstash Redis |

---

## Security

- Never commit your `.env` file — it's in `.gitignore` for a reason
- Credentials passed to the web UI are held in memory only for the duration of the job and never persisted
- Use HTTPS in any production/hosted deployment

---

## License

MIT © DragonBlade

---

## Contributing

PRs and issues welcome.  
**GitHub**: [DragonBlade431/CourseNuker](https://github.com/DragonBlade431/CourseNuker)

---

*"If you want me to stop, then win."*  
*The blade never dies. ⚔️*
