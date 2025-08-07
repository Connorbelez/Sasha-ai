# Sasha-ai
# Sasha-ai
# SASHA — The Anti-Phone-Anxiety, Anti-Executive-Dysfunction Assistant  
![CI](https://img.shields.io/github/actions/workflow/status/your-org/sasha/ci.yml)
![License](https://img.shields.io/github/license/your-org/sasha)
![Made with TypeScript](https://img.shields.io/badge/TS-5.x-blue)
![Open to PRs](https://img.shields.io/badge/contributions-welcome-brightgreen)

> “SASHA talks, you tap.”  
> Hold times, schedule-juggling, debt calls, and subscription clean-ups  
> handled by an AI secretary—while you stay in flow.

---

## Table of Contents
- Features
- Tech Stack
- High-Level Architecture
- Quick Start
- Local Development
- Deployment Targets
- Roadmap
- Contributing
- Security & Compliance
- License

---

## ✨ Features (PoC scope)

- Inbound call triage with hold music, live brief card, and warm transfer
- ‘Handle It Later’ deferral that auto-books a meeting once you approve
- Daily Build-in-Public digest: PRs → GPT-4o-mini summary → Google Doc
- Google Calendar free-busy lookup + Gmail invite send
- CopilotKit dashboard: approve/decline cards in real time
- Mastra workflows with exactly-once suspend/resume
- Twilio voice relay (PCM ↔ Mastra Realtime) <400 ms RTT
- Vector search over ingested Google Docs for quick context lookup
- Zero-ops developer experience: Vercel, Mastra Cloud, Convex (optional)

---

## 🛠 Tech Stack

| Layer                 | Choice                                  |
|-----------------------|-----------------------------------------|
| Front-end            | Next.js 15 + React 18 + CopilotKit      |
| Workflows & Agents   | Mastra Cloud                            |
| Storage (PoC)        | LibSQL (via Mastra Storage)             |
| Voice                | Twilio Voice + optional Hume provider   |
| Realtime relay       | Bun 1.x micro-service on Fly.io         |
| LLM / Embeddings     | OpenAI gpt-4o-mini + text-embedding-3-small |
| Vector search        | pgvector (LibSQL)                       |
| Infra as Code        | simple Terraform modules                |

---

## 🖼 Architecture (Mermaid)

```mermaid
graph TD
    subgraph Frontend
        A[Next.js + CopilotKit]
    end
    subgraph Cloud
        B[Mastra&nbsp;Cloud<br>Workflows]
        C[LibSQL<br>Mastra&nbsp;Storage]
        D[Vercel Edge API]
        E[Fly Relay<br>(WS Duplex)]
    end
    subgraph Vendors
        TW(Twilio Voice)
        GG(Google Calendar / Gmail)
        GH(GitHub App)
        OA(OpenAI API)
    end

    TW --webhook--> D
    D --startSaga--> B
    E --PCM--> B
    B --snapshot--> C
    B --WS--> A
    A --decision--> D
    B --API--> GG
    GH --PR webhook--> D
    D --summarise--> OA
    D --append--> GG
```

---

## ⚡ Quick Start (local)

```bash
# 1. Clone
git clone https://github.com/your-org/sasha.git
cd sasha

# 2. Bootstrap .env (see .env.example)
cp .env.example .env

# 3. Spin up dev stack (Next.js, LibSQL, ngrok tunnel for Twilio)
docker compose up --build

# 4. Start Mastra worker (separate terminal)
pnpm mastra:dev
```

Visit `http://localhost:3000` → sign up → call the ngrok number and watch
the Hold Card pop in.

---

## 🧑‍💻 Local Development

- `pnpm dev` – hot-reload Next.js + CopilotKit
- `pnpm mastra:dev` – run workflows against local LibSQL
- `pnpm relay` – Twilio PCM bridge (needs `TWILIO_*` envs)
- `pnpm test` – Vitest unit tests
- `pnpm lint` – ESLint + Prettier (print-width 80)

---

## 🚀 Deployment Targets

| Target      | Command / Action                         |
|-------------|------------------------------------------|
| Vercel FE   | Git push → CI builds & deploys           |
| Mastra Cloud| `pnpm mastra:deploy` (CLI)               |
| Fly Relay   | `fly deploy`                             |
| Twilio      | `terraform apply twilio-number`          |

Secrets are stored in GitHub Actions OIDC → Vercel / Fly secrets.  
See `.github/workflows/ci.yml` for matrix build.

---

## 🗺 Roadmap (condensed)

- [ ] EPIC 1 Foundations & DevOps
- [ ] EPIC 2 User On-Boarding
- [ ] EPIC 3 Inbound Call Triage
- [ ] EPIC 4 Handle-It-Later Saga
- [ ] EPIC 5 Google Calendar/Gmail
- [ ] EPIC 6 Dashboard polish
- [ ] EPIC 8 GitHub digest
- [ ] EPIC 10 Observability pass

Detailed issues live in GitHub Projects.

---

## 🤝 Contributing

1. Fork ➜ create feature branch (`feat/thing`)  
2. `pnpm test && pnpm lint` must be green  
3. PR with a clear description—GitHub Actions will preview Vercel deploy  
4. One review needed for merge to `main`

Please keep code width ≤80 chars and add/extend Zod schemas for all new
API payloads.

---

## 🔐 Security & Compliance

- All phone calls begin with a configurable consent blurb.  
- PII columns (`email`, `phone`) encrypted at rest in LibSQL via `sqlcipher`.  
- OAuth refresh tokens stored with AES-256 GCM + kms-wrapped key.  
- Twelve-Factor secret management; no secrets committed.  
- Basic SOC-2 checklist in `/docs/compliance/`.


---

## 📄 License

None. 
