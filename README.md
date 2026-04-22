# DHH Vibecoding Plugin

> Vibecoding dengan filosofi DHH. Distilled from 37signals' Fizzy, Campfire, dan Writebook — production apps yang ship dengan **zero build tools**, **no devise**, **no Sidekiq**, **no Redis**, **no Tailwind**.

*"The best code is the code you don't write. The second best is the code that's obviously correct."* — DHH

---

## Philosophy

Dunia vibecoding butuh north star. Kita pilih **DHH sebagai vibecoding prophet**.

**Kenapa?**
- **Rich domain models > service objects** — AI gak bingung nyari logic di 10 tempat
- **REST purity** — 1 cara bikin endpoint, AI gak harus invent ulang
- **Convention over configuration** — less decision fatigue, less token
- **Prefer Rails built-in over gems** — fewer dependencies = fewer failure modes
- **Database-backed everything** — no Redis, no Memcached, Solid Suite ftw

Kami pelajari:
- **Fizzy** — 37signals kanban app (~14K lines CSS, zero build tools)
- **Campfire** & **Writebook** — shipped proofs of DHH's philosophy
- **Unofficial 37signals Style Guide** — transferable patterns distilled
- **DHH's PR reviews** — real critique patterns from production codebases

This plugin ships that philosophy as Claude Code skills. Install once → Claude Code kamu jadi fluent di vanilla Rails, vanilla CSS, Hotwire, Solid Suite, Kamal. Prompting kamu lebih ringan, hasil lebih konsisten, decisions di-curate oleh "The DHH Way."

Your vibecoding partner learns from the best.

---

## Install

```bash
claude plugin marketplace add github:belajargpt/dhh-vibecoding-plugin
claude plugin install dhh-vibecoding
```

## Update

```bash
claude plugin update dhh-vibecoding
```

Quarterly update cycle untuk perubahan Rails / Claude Code / Kamal / Mayar.

---

## 10 Skills Inside

Each skill auto-invokes based on trigger keywords in your prompts.

### Rails Backend
- **`rails-conventions`** — DHH-style patterns (fat models, skinny controllers, REST purity, prefer Rails built-in over gems)
- **`auth-setup`** — Rails 8 built-in auth + authorization + Current attributes (no devise, no pundit)
- **`solid-suite-config`** — Solid Queue / Cache / Cable setup (no Redis, no Sidekiq)
- **`rails-debug-helper`** — error diagnosis, log reading, CI tools (Brakeman, Bundler Audit, Rubocop), deploy pre-flight

### Hotwire (UI)
- **`turbo-frames`** — single-section updates, lazy loading, `turbo_frame_tag` patterns from Fizzy
- **`turbo-streams`** — multi-section updates, broadcasts (declarative + manual), `method: :morph`
- **`stimulus-controllers`** — client-side behavior sprinkles, 55+ Fizzy patterns catalog

### Presentation
- **`vanilla-css`** — cascade layers, OKLCH colors, CSS variables, `:has()`, `@starting-style`, native nesting — zero build tools

### Integrations
- **`mayar-payment-integration`** — Mayar hosted checkout + webhook re-fetch pattern (Indonesian payment gateway)
- **`rubyllm-patterns`** — LLM APIs via RubyLLM gem (Claude/OpenAI/Gemini), streaming, cost-aware model selection

---

## Course Context

Plugin ini dirancang sebagai companion untuk **BelajarGPT Vibecoding** course — teaching non-programmer Indonesian untuk ship real Rails 8 apps pake Claude Code.

Plugin carries technical load (10 skills auto-invoked). Student fokus ke **PRD + product thinking**. AI nulis code, student arahin + verify.

Learn more: [BelajarGPT Vibecoding](https://belajargpt.co/vibecoding)

---

## Credits & References

This plugin stands on shoulders of:
- **DHH** — for the philosophy
- **37signals team** (Jason Fried, Jorge Manrubia, Jason Zimdars, et al.) — for shipping Fizzy, Campfire, Writebook as living proofs
- **Marc Köhlbrugge** — for extracting the patterns in [Unofficial 37signals Coding Style Guide](https://github.com/marckohlbrugge/unofficial-37signals-coding-style-guide)

We synthesized, verified, and re-expressed these patterns as Claude Code skills. Credit flows upstream.

---

## License

MIT — plugin code is yours to use, modify, share.

Reference content adapted from sources above (O'Saasy licensed code examples; please respect their terms).
