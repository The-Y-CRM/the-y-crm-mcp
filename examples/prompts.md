# Example prompts

Start with an overview, then let the AI do one area at a time. Check the result in the app; every step is in the change log.

## Getting started

- "Show me what is still missing in my CRM."
- "What may this connection do, and what can't it do?"

## Mailbox and web chat (permission: Set up)

- "Set up our mailbox support@example.com. It's a Gmail address."
- "Switch on the web chat and give me the code for our website."

Claude asks before it sets up a mailbox or switches the web chat on or off. The web chat counts as live in the overview once the first real visitor message has arrived.

## Phone system (permission: Set up)

- "Create extensions 101 to 104 for Anna, Ben, Carla and David and put them in one ring group called Support."
- "Monday to Friday 9:00 to 17:00 calls go to the Support ring group, otherwise to voicemail."
- "Add our number +43 1 2345678 and send its calls to the phone hours you just created."
- "Create the SIP line for our provider: server sip.example.net, user 123456." (You enter the password in the app afterwards.)

## Contacts, deals and tasks (permission: Set up)

- "Import these contacts, but show me a preview first: …" (The preview saves nothing. It shows per row what would happen, without IDs.)
- "Create a deal 'Website relaunch' worth 4,800 EUR for the contact Emma Fischer, expected to close at the end of the month."
- "Create a callback task for tomorrow at 10:00 for the newest WhatsApp conversation."

## Inbox (permission: Read and write drafts)

- "Which open conversations have been waiting longest?"
- "Read the newest email conversation and save a reply draft. Don't send it."

## Sending (separate permission: May send messages to customers)

- "Send the draft in that conversation."

Claude asks before every send. If the AI sends exactly the text of its draft, the draft is marked as done. In a test access nothing is delivered, and the AI says so.

Without the send permission, the AI saves replies as internal notes headed "NOT sent yet".
