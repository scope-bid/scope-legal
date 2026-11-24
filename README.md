# Scope for Law Firms

Order outside legal work from inside Claude. Scope connects your firm's AI to independent professionals who do work outside your office: serving papers, court runs, records retrieval, court reporting, and other services. Where enabled, Scope also offers federal court records pulls and docket watches at Scope's own listed prices. You tell Claude what the matter needs. Claude gathers the facts, shows you each professional's own price, and prepares the request. A person at your firm approves it before anything is committed.

This plugin has two parts:

- A connector pointer to Scope's remote server at `https://scope.bid/api/mcp/legal`. Signing in uses OAuth in Claude. You never paste a password or token.
- One skill, `order-outside-legal-work`, that teaches Claude how to run an order from the first question to the finished file.

## What a request looks like

1. You describe the work in plain words: who, where, which documents, the deadline.
2. Claude asks only for what is missing, then returns quotes. Each professional sets their own price, and Claude shows the prices as Scope returns them.
3. You pick the professional. Claude prepares the award.
4. The award waits as a pending approval. A person at your firm approves it from a signed link Scope emails to the approver, or from the approvals page in Scope, which requires signing in. Claude cannot approve for you.
5. Claude checks progress when you ask: status, messages from the professional, what is waiting on whom.
6. When the work comes back, Claude retrieves it, with proof of service for a serve.

## What you pay and how

- Professionals set their own prices.
- Scope charges the firm a separate 15 percent fee on the professional's labor.
- Scope invoices the firm through Stripe, outside the conversation. No card is collected in Claude, and no Scope tool moves money.

## Approval is the firm's

Every award or dispatch requested through this connector is held as a pending approval until a person at the firm approves it. Nothing is committed and no professional is released before then, and no firm setting removes that step. If Scope has no one at the firm to send the approval to, the request is refused and nothing is committed.

## Requirements

- A Scope firm account. To get one, see https://scope.bid.
- A Claude plan that supports connectors and plugins.
- At least one person at your firm who can approve requests in Scope.

## Your data

- Claude sends Scope what you type for an order: the work needed, jurisdiction, and the facts the work order asks for. Documents are uploaded in Scope through a secure link that opens for a signed-in member of the firm. Scope's tools receive that and return quotes and matter status.
- Before an award is approved, professionals see at most an anonymized description of the work. Firm and party names are redacted at that stage.
- After a person at the firm approves the award, the chosen professional sees the full work order, including names, addresses and documents. Messages you send post to the matter thread, and the professional is notified by email.
- Reading the matter thread marks the professional's messages as read for your firm.
- Medical records get stricter handling in Scope.
- Scope's privacy policy, including how long data is kept: https://scope.bid/privacy.
- The plugin itself stores nothing and runs no code. It contains instructions for Claude and a pointer to the connector.

## What this plugin does not do

- It does not give legal advice, including on service rules, filing rules or deadlines.
- It does not approve anything for you.
- It does not collect payment or hold funds.

## Documentation and support

- Documentation: https://scope.bid/docs
- Support: the form at https://scope.bid/contact
- Privacy: https://scope.bid/privacy

## License

MIT. See `LICENSE`.
