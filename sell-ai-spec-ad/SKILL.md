---
name: sell-ai-spec-ad
description: Turn a finished AI spec ad (a demo film or photos made for a brand that didn't commission them) into a private, watermarked Peekroom room with a price, plus a ready-to-send message to the brand. Use when the creator says "sell this spec ad", "send this ad to [brand]", "make a room for [brand]", "how do I get [brand] to pay for this", or shares a finished Higgsfield/Kling/Runway/Veo render meant for a brand. Needs the Peekroom connector (https://app.peekroom.art/mcp).
license: MIT
---

# Sell an AI spec ad with Peekroom

You are helping a creator get paid for a spec ad. The deliverable is a **draft** room in their Peekroom studio, its link and access code, and a message to the brand that the creator sends themselves.

Two hard guardrails, both about the creator staying in control:

- Call `publish_room` only after the creator says "publish" (or an equally clear yes) in reply to your summary.
- You draft the message to the brand. The creator sends it. You have no tool for contacting brands; never look for one.

## Before you start

Check the Peekroom tools are available: `create_room`, `upload_asset`, `confirm_upload`, `set_watermark`, `list_rooms`, `get_room_link`, `publish_room`, `reorder_assets` (also `set_cover`, `rename_asset`, `set_price`, `update_room`, `replace_asset`, `remove_asset` for changes). If they aren't, tell the creator to connect Peekroom first and stop there until it's connected:

- Claude (web or desktop): Settings → Connectors → Add custom connector, URL `https://app.peekroom.art/mcp`, then sign in on the Peekroom page that opens.
- Claude Code: `claude mcp add --transport http peekroom https://app.peekroom.art/mcp`, then `/mcp` to sign in.
- ChatGPT, Cursor, Gemini and scripts: https://www.peekroom.art/agents

## Step 1. Collect the brief

Ask only for what's missing, in one message. You need:

| Field | Notes |
|---|---|
| Brand name | As the brand writes it. The brand sees it on the room. |
| Room title | Short, e.g. "Serum launch film". Suggest one from the product if they don't care. |
| Files | Local files or public download links (`https://…`). Videos MP4/MOV/WebM up to 2 GB and 10 min; photos JPEG/PNG/WebP/HEIC up to 100 MB. |
| File names | What the brand sees, e.g. "Hero 9:16", "Cut-down 1:1". Default to the shape and length. |
| Price | One price in euros for the whole pack. Optional higher crossed-out list price. |
| Usage | Organic only, organic + paid, or full buyout. Goes in the room description, which the brand sees. |
| Watermark | The room starts with the studio name as text, tiled. Other options: a logo URL (Pro/Studio) or the Peekroom mark. |
| Language | The language the brand reads: en, pt or es. Default en. |
| Contact | Name of the person at the brand, if they have one. Only for the message. |

Don't suggest a price. If they ask, help them reason from their costs, what the film replaces for the brand, and the usage they're granting, and let them pick the number.

Done when every required field (brand, files, price) has a value the creator gave or accepted.

## Step 2. Build the draft room

1. `create_room` with `brand_name`, `title`, `locale`, `price_eur` (and `list_price_eur` if given), and a `description` that lists the files and the usage terms in two or three plain lines. Keep the `room_id`.
2. Add each file with `upload_asset` (`room_id`, `label`, and one of the two ways below).
   - **Public link:** pass `source_url`. The server downloads the file. Nothing else to do.
   - **Local file:** pass `filename`, `content_type` (e.g. `video/mp4`, `image/jpeg`) and `bytes` (the exact size). The tool returns an upload URL valid for one hour and a ready `curl` command. If you can run shell commands (Claude Code, a terminal agent), run that `curl` from the folder that holds the file, then call `confirm_upload` with the `asset_id`. If you can't (a file attached to a chat can't be sent on), give the creator the `curl` to run, or tell them to drag the file into the room in the Peekroom editor (the `edit_url` from `create_room`), and continue.
   - If `confirm_upload` says the file hasn't arrived, the `curl` didn't finish: run it again, then confirm again.
3. The watermark is already the studio name, tiled. Call `set_watermark` only if the creator wants something else: another `text`, `mark: "logo"` with `logo_url` (Pro/Studio), or `mark: "peekroom"`, and a `pattern` (`tiled`, `diagonal`, `frame`, `line`, `center`, `corner`). Mention that tiled is the hardest to crop out. Previews already made are re-rendered on their own.
4. `list_rooms` with `include_files: true` to check each file and its `position`. Previews usually take seconds to a minute. While a file is PROCESSING, wait and check again; report any FAILED file with the reason the tool gives.
5. Put the film first. The brand starts the room with file 1, and files land in the order they were uploaded. If the main film isn't at position 1, call `reorder_assets` with `room_id`, the film's `asset_id` and `position: 1` (or `asset_ids` with every file in the order the creator wants). Reordering never changes the room cover.
6. Optional: if the creator names a frame for the cover, `set_cover` with `asset_id`, `time_seconds` and `room_cover: true` (or `image_url` for their own picture). This only works once the video is READY; if the tool says it is still processing, wait and retry.

If a tool returns a plan limit or a Stripe message, relay it in plain words and carry on with what still works. Drafts can be built without Stripe; only publishing needs it.

Done when the room exists as DRAFT with every file READY (or the creator knows which ones to add in the editor), and the price set.

## Step 3. Show the summary and the message, then wait

Call `get_room_link`. The link and access code it returns stay the same after publishing. Show the creator:

- Room title, brand, status (Draft), price and usage.
- Each file with its name and status.
- Watermark choice.
- Link and access code, and the editor link to see the room as the brand will.
- Any warning the tools returned. If Stripe isn't connected, say: "To publish, connect Stripe first: Payments in Peekroom (https://app.peekroom.art/pagamentos). Your draft stays saved." On the Free plan, add that a draft nobody edits or publishes for 7 days loses its files.
- The message to the brand (below), ready to paste once the room is published.
- One line: "The brand can't open this until it's published. Reply **publish** when you're happy, or tell me what to change."

Make any changes they ask for with the same tools (`rename_asset`, `reorder_assets`, `set_price`, `set_watermark`, `set_cover`, `update_room` for the title, brand name, description or language), then show the summary again. To swap a file for a new version, upload it with `upload_asset` and call `replace_asset` (old `asset_id`, `new_asset_id`). To drop a file, `remove_asset`. Both delete a file for good, so confirm with the creator first.

### The message to the brand

Write it in the room's language, using the link and code. Keep it under 120 words:

```
Subject: A [product] film I made for [Brand]

Hi [contact name],

I'm [creator name]. I made a [length] ad for [product] on spec, because [one specific reason from the creator].

Watch it here, watermarked:
[link]
Access code: [code]

The clean files are €[price] for [files], [usage]. You can pay in the room and download straight away. If you'd like a different cut, tell me.

[creator name]
```

Fill only what the creator told you. Leave a visible `[bracket]` for anything you don't know; never invent a reason, a past client or a result.

## Step 4. Publish on the creator's word

Only after an explicit yes: `publish_room` with `room_id`. It needs Stripe connected, a price and at least one READY file.

- If it says Stripe isn't connected: nothing was lost. Tell them to open Payments in Peekroom (https://app.peekroom.art/pagamentos), connect Stripe, and say "publish" again. Don't retry before they confirm.
- If it says to upload a file and wait until it's ready: check `list_rooms` and retry when a file is READY.
- Relay any `archived_to_make_room` note (on the Free plan, publishing past the active-room limit archives the oldest room in 7 days).

## Step 5. Hand over

Once published, repeat the link, the access code and the message (updated if anything changed). Then add two short tips: send it to a named person using contact details the brand publishes, and follow up once after about a week, adding something new (a new cut or format) to the same room. The room shows how many times the brand opened it (`brand_views` in `list_rooms`), which tells them whether to resend or to follow up.

Done when the creator has the published link, the access code and the message, and you've said plainly that nothing has been sent to the brand.
