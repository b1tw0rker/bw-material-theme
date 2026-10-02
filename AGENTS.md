# AGENTS.md – bw-material-theme

Gemeinsame Anleitung für KI-Agenten (Claude Code, Codex, Copilot …). `CLAUDE.md` importiert diese Datei.

## Projekt

Reines **VS-Code-Farbschema** (kein Code, kein Build-Schritt, keine Tests), veröffentlicht im Marketplace als
`BITWORKER.bw-material-theme` → https://marketplace.visualstudio.com/items?itemName=BITWORKER.bw-material-theme

„Nick's Personal Theme – A theme with the charme of the sixties.“ Dunkles Theme (`uiTheme: vs-dark`), Akzentfarbe Orange.

| Datei | Zweck |
|---|---|
| `themes/bw-material-theme-color-theme.json` | **Das** Theme: `colors` (~340 Workbench-Farben) + `tokenColors` (~230 TextMate-Regeln). Fast alle Änderungen passieren hier. |
| `package.json` | Extension-Manifest, `version` wird pro Release erhöht. |
| `CHANGELOG.md` | Release-Notizen, neuester Eintrag oben (`### x.y.z` + `• …`). |
| `README.md` | Marketplace-Seite; enthält ebenfalls die Versionshistorie → **mit CHANGELOG synchron halten**. |
| `.vscodeignore` | Hält PSD, `.vsix`, `Notes.txt`, `.env`, `backup.zip`, Agent-Dateien, `.vscode/` aus dem Paket. |
| `.gitignore` | Hält `.env`, `Notes.txt`, `*.vsix`, `backup.zip` aus Git. |
| `.vscode/launch.json` | F5 = Extension Development Host zum Live-Testen des Themes. |
| `bitworker.png` | Marketplace-Icon. `VsCode-Snippets.psd` = Grafik-Quelle, nie anfassen. |
| `*.vsix` | Gebaute Pakete (Artefakte, nicht versioniert). |
| `backup.zip` | Komplettsicherung des Stands vor 0.0.18 – enthält Zugangsdaten, nie committen. |

Git-Repo: https://github.com/b1tw0rker/bw-material-theme (Branch `main`); `package.json` → `repository`/`bugs`/`homepage` zeigen dorthin.

## Farbpalette (verbindlich)

Neue Farben **aus dieser Palette** wählen, keine fremden Töne erfinden. Transparenz per 8-stelligem Hex (`#E15D1040`).

| Rolle | Hex |
|---|---|
| Akzent / Fokus / Rahmen aktiv / Variablen | `#E15D10` |
| Akzent dunkler (Buttons) | `#CD5A3C` |
| Editor-Hintergrund | `#13161B` |
| Sidebar / Panel / Tabs | `#161A21` |
| Widgets / Inputs / erhöhte Flächen | `#1B1E26` |
| Hover auf erhöhten Flächen | `#22262F` |
| Code-Blöcke (tiefer als Editor) | `#0E1115` |
| Rahmen / inaktiv / Zeilennummern | `#353A4A` |
| Vordergrund-Text | `#A0AABE` |
| Heller Text / Inline-Code | `#D8DEE9`, `#ffffff` |
| Kommentare / Platzhalter | `#5C6370` |
| Grün (Strings, Keywords, „added“) | `#85a486` |
| Blau (Konstanten, Control-Keywords) | `#5c8ea9` |
| Rot (Fehler, „removed“) | `#FF3A48` |
| Gelb (Warnungen) | `#BEB12B` / `#ECDA32` |

**Grundregel:** Es ist ein Dark Theme. **Keine hellen Hintergründe** (`#ffffff` o. ä.) bei `*.background`-Keys – Webviews wie Claude Code, Codex oder Copilot Chat nutzen `textCodeBlock.background`, `textPreformat.*`, `textBlockQuote.*`, `chat.*` usw. für ihre Nachrichten- und Code-Blöcke. Ein heller Wert dort erzeugt weiße Blöcke (Bug in ≤ 0.0.16, behoben in 0.0.17).

## Konventionen in der Theme-JSON

- Einrückung: **6 Leerzeichen** pro Ebene (Farb-Keys stehen auf 12 Spaces) – siehe `.vscode/settings.json` (Prettier `tabWidth: 6`).
- Hex in der Regel großgeschrieben, ältere Einträge teils klein – beim Bearbeiten den Stil der Nachbarzeile übernehmen.
- Neue Workbench-Keys thematisch zu verwandten Keys gruppieren (z. B. `chat.*` direkt nach `text*`).
- **Strikt gültiges JSON**: kein Trailing Comma, keine Kommentare, keine doppelten Keys (VS Code toleriert das, andere Tools nicht).
- Gültige Keys: https://code.visualstudio.com/api/references/theme-color – nur dokumentierte Keys verwenden.
- `tokenColors`-Einträge haben `name`, `scope`, `settings` (`foreground`, optional `fontStyle`). Scopes ermitteln mit *Developer: Inspect Editor Tokens and Scopes*.

### Validierung nach jeder Änderung

```bash
node -e "const s=require('fs').readFileSync('themes/bw-material-theme-color-theme.json','utf8');JSON.parse(s);const k=[...s.matchAll(/^\s{12}\"([\w.]+)\":/gm)].map(m=>m[1]);const d=k.filter((x,i)=>k.indexOf(x)!==i);if(d.length)throw new Error('dup keys: '+d);console.log('ok')"
```

## Release-Ablauf

1. Theme-JSON ändern + validieren (siehe oben).
2. `version` in `package.json` erhöhen (Patch: `0.0.x`).
3. Eintrag oben in `CHANGELOG.md` **und** im Versionsteil von `README.md` ergänzen (Englisch, Stil `• kurze Beschreibung`).
4. Paket bauen und Inhalt prüfen – es dürfen nur Theme, `package.json`, README, CHANGELOG, LICENSE, Icon drin sein:
   ```bash
   npm run build    # = npx -y @vscode/vsce package
   ```
   (`vsce` ist nicht global installiert → immer `npx @vscode/vsce`.)
5. Veröffentlichen – Token liegt in `.env` als `TOKEN=…`:
   ```bash
   export VSCE_PAT=$(grep '^TOKEN=' .env | cut -d= -f2- | tr -d '\r"')
   npx -y @vscode/vsce publish --packagePath bw-material-theme-<version>.vsix
   ```
   Publisher: `BITWORKER`. Token-Fehler `TF400813` = Token abgelaufen/falscher Scope → neues PAT unter dev.azure.com/BITW0RKER (Scope *Marketplace → Manage*, „All accessible organizations“).

Lokal testen ohne Publish: F5 (Extension Development Host) oder *Extensions: Install from VSIX…*, danach *Developer: Reload Window*.

## Bekannte Fallstricke

- `editor.selectionForeground` wirkt **nur in High-Contrast-Themes**. Lesbarkeit markierten Codes nur über die Alpha von `editor.selectionBackground` steuern (aktuell `#E15D1060`; mehr Deckkraft macht orange Variablen unlesbar).
- Keinen Sprach-Root-Scope (`source.ts`, `source.php` …) in `tokenColors` aufnehmen – das färbt die ganze Datei.
- Gleicher Scope-String in mehreren Regeln: die **letzte** gewinnt. Vor dem Hinzufügen prüfen, ob der Scope schon existiert, und dann die bestehende Regel ändern.
- Gültige `fontStyle`-Werte: `italic`, `bold`, `underline`, `strikethrough` oder `""`. `foreground` muss Hex sein (kein `inherit`).
- Keys aus Extensions (`gitDecoration.*` → Git, `gitlens.*` → GitLens, `parameterHints.*` → Extension „parameter-hints“) fehlen in der VS-Code-Doku, sind aber gültig.
- Welche Theme-Farben Claude Code / Codex nutzen: `var(--vscode-…)` in `~/.vscode/extensions/anthropic.claude-code-*` bzw. `openai.chatgpt-*` greppen (`.` im Key ↔ `-` in der CSS-Variable).

## Sicherheit

- `.env` und `Notes.txt` enthalten **Zugangsdaten**. Niemals ausgeben, zitieren, committen oder ins Paket aufnehmen. Beim Lesen von `.env` Werte maskieren.
- Vor jedem `vsce package` sicherstellen, dass `.vscodeignore` beide Dateien weiterhin ausschließt.

## Sprache

Der Nutzer schreibt Deutsch – Antworten auf Deutsch. Changelog/README-Einträge auf Englisch (bestehender Stil).
