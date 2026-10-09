# n8n Email Notification (Friday Task)

This is my Friday task from the n8n week. An n8n workflow pulls a random quote from a public API, filters it, sends it to a test webhook, and then emails the quote to my Gmail inbox.

## What this project does

1. Runs automatically on a schedule (every minute while published).
2. Gets a random quote from the DummyJSON Quotes API (same API I used in Week 2).
3. Keeps only long quotes (more than 60 characters) using a Filter node.
4. Sends the quote as a `POST` request to webhook.site.
5. Sends the quote to my Gmail inbox using the Gmail node.

## API used

**DummyJSON Quotes API**: `https://dummyjson.com/quotes/random`

It is free and needs no API key. Each call returns a random quote with three fields: `id`, `quote` and `author`.

## Workflow

```
Schedule Trigger → HTTP Request → Filter → HTTP Request1 → Send a message (Gmail)
```

| Node | What it does |
| --- | --- |
| Schedule Trigger | Starts the workflow automatically every 1 minute |
| HTTP Request | `GET` request to the quotes API, returns `id`, `quote`, `author` |
| Filter | Passes the item only if `{{ $json.quote.length }}` is greater than 60 |
| HTTP Request1 | `POST` request that sends the quote data to webhook.site |
| Send a message | Gmail node, emails the quote to my inbox |

### Email message

The Gmail message uses this expression:

```
{{ $('HTTP Request').item.json.quote }} - {{ $('HTTP Request').item.json.author }}
```

I had to read the quote from the first HTTP Request node directly. The webhook node (HTTP Request1) replaces the data with its own response, so `{{ $json.quote }}` was empty in the email at first.

## Result 1: workflow canvas

![Workflow canvas](canvas.png)

All five nodes ran successfully (green ticks). The Filter node kept 1 item: a quote by Maya Angelou, which is longer than 60 characters.

## Result 2: email received

![Email in Gmail inbox](email.png)

The email "New quote from n8n" arrived in my Gmail inbox with the same quote and author that the workflow pulled in the run above.

## What I learned

- A notification node (Gmail) sends a message when the workflow reaches it.
- A Filter node stops items that do not match its condition. For short quotes, the workflow stops at the Filter and no email is sent.
- Each node replaces the data with its own output. To use data from an earlier node, I can reference it with `$('Node Name').item.json.field`.
- Order of nodes matters. The Filter must come right after the HTTP Request so it can see the `quote` field.
- A Schedule Trigger only runs automatically when the workflow is published. I unpublished it after testing so it does not keep sending emails.

## Files in this repo

- `README.md`: this file
- `canvas.png`: screenshot of the workflow canvas
- `email.png`: screenshot of the email in my inbox
