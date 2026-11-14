# sell-ai-spec-ad (Claude skill)

Turns a finished AI spec ad into a private, watermarked Peekroom room with a price, and drafts the message to the brand. The skill builds a draft and stops: it publishes only when you say "publish", and it never sends anything to the brand.

## Install

- Claude (web/desktop): zip this folder and upload it in the Skills section of Settings.
- Claude Code: copy the folder to `~/.claude/skills/sell-ai-spec-ad/`.

You also need the Peekroom connector: Settings → Connectors → Add custom connector, URL `https://app.peekroom.art/mcp`.

## Try

"Sell this spec ad to Maison Aube for €250: https://…/hero.mp4"

MIT licence. Made by Peekroom (peekroom.art).
