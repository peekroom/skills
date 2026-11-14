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

Check the Peekroom tools are available (`create_room`, `upload_asset`, `set_watermark`, `set_price`, `get_room_link`, `publish_room`). If they aren't, tell the creator to connect Peekroom first: in Claude, Settings → Connectors → Add custom connector, URL `https://app.peekroom.art/mcp`, then sign in on the Peekroom page that opens. Stop there until it's connected.

## Step 1. Collect the brief

Ask only for what's missing, in one message. You need:

| Field | Notes |
|---|---|
| Brand name | As the brand writes it. Used as the room name the brand sees. |
| Room title | Short, e.g. "Serum launch film". Suggest one from the product if they don't care. |
| Files | Public download links (`https://…`) to each video/photo. Videos MP4/MOV/WebM up to 2 GB and 10 min; photos JPEG/PNG/WebP/HEIC up to 100 MB. |
| File names | What the brand sees, e.g. "Hero 9:16", "Cut-down 1:1". Default to the shape and length. |
| Price | One price in euros for the whole pack. Optional higher crossed-out list price. |
| Usage | Organic only, organic + paid, or full buyout. Goes in the room description. |
| Watermark | Studio name as text (any plan), their logo URL (Pro/Studio), or the Peekroom mark. Default: their studio name, tiled. |
| Language | The language the brand reads: en, pt or es. Default en. |
| Contact | Name of the person at the brand, if they have one. Only for the message. |

Files attached directly to the chat can't be passed as links. Ask for a public link (most AI video tools give a download URL for each render; a shared Drive/Dropbox link set to "anyone with the link" also works if it downloads the file directly). If they only have local files, create the room anyway and tell them to drag the files into the room in the Peekroom web editor, then continue.

Don't suggest a price. If they ask, help them reason from their costs, what the film replaces for the brand, and the usage they're granting, and let them pick the number.

Done when every required field (brand, files or the web-upload fallback, price) has a value the creator gave or accepted.

## Step 2. Build the draft room

1. `create_room` with `brand_name`, `title`, `locale`, `price_eur` (and `list_price_eur` if given), and a `description` that lists the files and the usage terms in two or three plain lines.
2. `upload_asset` once per file with `room_id`, `source_url` and `label`.
3. `set_watermark` with the creator's choice. Default: `mark: "text"`, `text: <studio name>`, `pattern: "tiled"`. If they want something cleaner, offer `diagonal` or `frame` and mention that tiled is the hardest to crop out.
4. Optional: if the creator names a frame for the cover, `set_cover` with `asset_id`, `time_seconds` and `room_cover: true` once the video is READY.
5. `list_rooms` with `include_files: true` to confirm every file reached READY or is still processing. Report any failed file with the reason the tool gives.

If a tool returns a plan limit or a Stripe message, relay it in plain words and carry on with what still works. Drafts can be built without Stripe; only publishing needs it.

Done when the room exists as DRAFT with every file uploaded (or the creator knows which to add on the web), the watermark set and the price set.

## Step 3. Show the summary and wait

Call `get_room_link` and show the creator:

- Room title, brand, status (Draft), price and usage.
- Each file with its name and status.
- Watermark choice.
- Link and access code.
- Any warning the tool returned (Stripe not connected, plan limits).
- One line: "The brand can't open this until it's published. Reply **publish** when you're happy, or tell me what to change."

Make any changes they ask for with the same tools, then show the summary again.

## Step 4. Publish on the creator's word

Only after an explicit yes: `publish_room` with `room_id`. If it fails because Stripe isn't connected, tell them to connect Stripe in Peekroom (Payments in the studio menu, app.peekroom.art) and say "publish" again. Relay any `archived_to_make_room` note.

## Step 5. Hand over the message

Write the message to the brand in the room's language, ready to paste, using the link and code from `publish_room`. Keep it under 120 words:

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

Then add two short tips: send it to a named person using contact details the brand publishes, and follow up once after about a week, adding something new (a new cut or format) to the same room. The room shows how many times it was viewed, which tells them whether to resend or to follow up.

Done when the creator has the published link, the access code and the message, and you've said plainly that nothing has been sent to the brand.
