# Rails Vibecoding Skills

Claude Code plugin marketplace untuk BelajarGPT Vibecoding course — Rails 8 + Hotwire + Solid Suite + Kamal + Mayar + RubyLLM.

## Install

```bash
claude plugin marketplace add github:belajargpt/rails-vibecoding-skills
claude plugin install rails-vibecoding
```

## What's in the Plugin

`rails-vibecoding` ships with skills yang bikin Claude Code jadi expert di stack Rails 8 + Hotwire untuk shipping production apps:

- **rails-conventions** — DHH-style patterns (fat models, skinny controllers, REST purity, prefer Rails built-in)
- **hotwire-patterns** — Turbo Frames, Turbo Streams, Stimulus recipes _(coming soon)_
- **solid-suite-config** — Solid Queue/Cache/Cable setup _(coming soon)_
- **auth-setup** — Rails 8 built-in auth + authorization patterns _(coming soon)_
- **rails-debug-helper** — error diagnosis + log inspection + deploy pre-flight _(coming soon)_
- **mayar-payment-integration** — Mayar checkout + webhook re-fetch pattern _(coming soon)_
- **rubyllm-patterns** — RubyLLM gem usage + streaming + cost-aware model selection _(coming soon)_

## Update

```bash
claude plugin update rails-vibecoding
```

Quarterly update cycle untuk perubahan Rails / Claude Code / Kamal / Mayar.

## Course Context

Plugin ini dirancang sebagai companion untuk **BelajarGPT Vibecoding** course — teaching non-programmer Indonesian untuk ship real Rails 8 apps pake Claude Code. Plugin carries the technical load (skills auto-invoked), student fokus ke PRD + product thinking.

Learn more: [BelajarGPT Vibecoding course info](https://belajargpt.co/vibecoding)

## Reference

Pola DHH-style Rails conventions sebagian di-inform oleh [Unofficial 37signals Coding Style Guide](https://github.com/marckohlbrugge/unofficial-37signals-coding-style-guide) — transferable Rails patterns extracted from 37signals' Fizzy codebase.

## License

MIT (plugin code). See LICENSE file.
