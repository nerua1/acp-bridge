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
| **Hermes** | `acpx hermes` | ✅ Działa | Wymaga `~/.acpx/config.json` z override `hermes` → `hermes acp` |

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

### Hermes

**Krok 0 — konfiguracja (raz)**
```bash
mkdir -p ~/.acpx
cat > ~/.acpx/config.json << 'EOF'
{
  "agents": {
    "hermes": {
      "command": "hermes acp"
    }
  }
}
EOF
```

**Krok 1 — utwórz sesję**
```bash
npx acpx hermes sessions new --name hermes-main
```

**Krok 2 — wyślij wiadomość**
```bash
npx acpx hermes prompt "Treść wiadomości" --session hermes-main
```

**One-shot bez sesji**
```bash
npx acpx hermes exec "Treść wiadomości"
```

---

## Ograniczenia — WAŻNE

1. **Shell nie działa** w `acpx exec` — `spawn ... ENOENT`. Nie używaj Shell/bash do zapisu plików.
2. **WriteFile ograniczony do cwd** — acpx domyślnie ustawia `cwd=/Users/nerucb1`. Rozwiązanie: `--cwd /Volumes/2TB_APFS/.openclaw`.
3. **Hermes ACP config** — `acpx` nie zna natywnie komendy `hermes acp`. Wymaga override w `~/.acpx/config.json` (patrz sekcja Hermes powyżej).

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

---

If this saved you time: [support me](https://www.paypal.me/nerudek)
