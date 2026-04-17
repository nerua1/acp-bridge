---
name: acp-bridge
description: Komunikacja między agentami przez ACP (Agent Client Protocol) — komendy acpx dla Claude, Kimi, OpenClaw, Hermes. Uwaga: Hermes wymaga workaround (MCP lub pliki).
type: reference
created: 2026-04-17
authors: [Claude Code, Kimi K2.5]
version: "1.1.0"
---

# ACP Bridge 🌉

Skill dokumentuje jak agenci (Claude, Kimi/Rook, Hermes, OpenClaw) komunikują się bezpośrednio przez ACP (Agent Client Protocol) via `acpx`.

## Instalacja

`acpx` jest zainstalowany globalnie:
```bash
npm install -g acpx@latest
```
Aktualna wersja: `0.5.3`

---

## Status ACP per agent

| Agent | Komenda | Status | Uwagi |
|-------|---------|--------|-------|
| **Claude Code** | `acpx claude` | ✅ Działa | Natywne wsparcie |
| **Kimi (Rook)** | `acpx kimi` | ✅ Działa | Natywne wsparcie |
| **OpenClaw** | `acpx openclaw` | ✅ Działa | Natywne wsparcie |
| **Hermes** | `acpx hermes` | ⚠️ Częściowo | TUI na stdout zaciera JSON-RPC. Użyj MCP lub plików. |

---

## Komendy — wysyłanie wiadomości

### Działające natywnie

```bash
# Claude Code
npx acpx claude exec "wiadomość do Claude"
npx acpx claude "wiadomość (persistent session)"

# Kimi / Rook
npx acpx kimi exec "wiadomość do Kimi"
npx acpx kimi "wiadomość (persistent session)"

# OpenClaw
npx acpx openclaw exec "wiadomość do OpenClaw"
npx acpx openclaw "wiadomość (persistent session)"
```

### Hermes — workaroundi

**Opcja A: Hermes MCP (preferowana dla tool-calling)**
```bash
# Terminal 1: wystaw Hermesa jako MCP server
hermes mcp serve

# Terminal 2: inny agent łączy się przez MCP client
# (Claude Code, OpenClaw, Kimi — wszyscy mają MCP client)
```

**Opcja B: Pliki współdzielone (fallback)**
```bash
# Zapisz do AGENT-HANDOFF.md
cat >> /Volumes/2TB_APFS/openclaw-data/workspace/obsidian-memory/bridge/AGENT-HANDOFF.md << 'EOF'
## Handoff [$(date +%Y-%m-%d %H:%M)] — [Agent nadawcy]
- **Do**: hermes
- **Treść**: wiadomość...
EOF
```

**Opcja C: ACP via wrapper (eksperymentalna)**
```bash
# Użyj wrappera który filtruje logi TUI
# Zobacz: ~/.local/bin/hermes-acp-wrapper
npx acpx --agent "hermes-acp-wrapper" exec "wiadomość"
# ⚠️ Niestabilne — hermes-acp wymaga patcha upstream (logi na stdout zamiast stderr)
```

---

## Ograniczenia — WAŻNE

1. **Shell nie działa** w `acpx exec` — `spawn ... ENOENT`. Nie używaj Shell/bash do zapisu plików.
2. **WriteFile ograniczony do cwd** — acpx domyślnie ustawia `cwd=/Users/nerucb1`. Rozwiązanie: `--cwd /Volumes/2TB_APFS/.openclaw`.
3. **Hermes ACP TUI bug** — `hermes acp` wypisuje banner + tools + skills na stdout, co psuje JSON-RPC. Fix wymaga patcha Hermesa (issue: logi INFO → stderr, TUI → /dev/null w trybie acp).

---

## Przykład — handoff zadania między agentami

### Claude → Kimi ( przez ACP)
```bash
npx acpx kimi exec "Przejmij zadanie X — szczegóły w AGENT-HANDOFF.md"
```

### Kimi → Hermes (przez pliki)
```bash
# Kimi zapisuje
npx acpx kimi exec "Zapisz do AGENT-HANDOFF.md handoff dla Hermesa"

# Hermes odczyta przy następnym starcie
```

### OpenClaw → Claude ( przez ACP)
```bash
npx acpx claude exec "OpenClaw prosi o review kodu: ..."
```

---

## Plik handoff

Ścieżka:
```
/Volumes/2TB_APFS/openclaw-data/workspace/obsidian-memory/bridge/AGENT-HANDOFF.md
```

Format wpisu:
```markdown
## Handoff [YYYY-MM-DD HH:MM] — [Agent nadawcy]
- **Do**: [Agent odbiorcy]
- **Treść**: ...
- **Otwarte / TODO**: ...
- **Ważne decyzje**: ...
- **Następny krok**: ...
```

---

## Historia

- **2026-04-17**: Pierwszy test ACP między Claude i Kimi — sukces.
- **2026-04-17**: OpenClaw podłączony przez ACP — sukces.
- **2026-04-17**: Hermes — TUI bug zidentyfikowany. Workaround: MCP lub pliki.
