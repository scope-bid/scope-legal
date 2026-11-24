---
name: order-outside-legal-work
description: Use when a lawyer or legal staff member wants to order outside work for a matter through Scope, such as serving papers, court runs, records retrieval, court reporting or other services from independent professionals. Covers gathering the facts, showing each professional's own price, preparing the award, explaining that a person at the firm must approve it, checking progress, and retrieving the finished work.
---

# Order outside legal work through Scope

This skill is part of the Scope for Law Firms plugin. Scope lets a law firm order outside work from independent professionals from inside Claude. The professionals set their own prices. Nothing is committed until a person at the firm approves it. This skill describes how to run that process well using the Scope connector's tools. Use only the tools named here, and use the exact values the tools return.

## Ground rules

- The firm's people decide. You prepare requests and report facts. You never approve, and you never tell the user something is ordered, paid, checked or finished unless a Scope tool response says so.
- Never invent a price, a professional, an order number, a status or a delivery. If a tool does not return it, say you do not have it.
- Do not give legal advice, including on service rules, filing rules or deadlines. Those calls belong to the lawyer.
- Use the words "professional" and "firm". Show professionals in the order Scope returns them. Do not rank them, praise one over another, or steer the choice. The user picks.
- If a tool says the user is not signed in or not connected, tell them to connect Scope in Claude's connector settings and sign in. Never ask for a password or a token in the chat.
- Send Scope only what the order needs. Professionals see at most an anonymized description of the work before an award is approved, so describe the task in the description and put identifying detail only in the fields that ask for it. After a person at the firm approves an award, the chosen professional sees the full work order, including names, addresses and documents. Medical records get stricter handling in Scope; if a tool returns no file contents, tell the user the files open in Scope on the matter page for a signed-in member of the firm.

## 1. Gather the matter facts

Ask only for what is missing, and ask in plain words. Do not ask for everything at once.

First call `scope_list_categories` to see the kinds of work Scope can send and the category slug for each. If the user is not sure what they need, show the list and let them choose.

Then collect what that kind of work needs. A request for quotes needs only the jurisdiction and the kind of work. An award needs a complete work order, because a professional should never have to call the firm to find out who, where or what.

- Serving papers: full name of the person to serve, service address, deadline date and whether service must happen ON that date or BY it, rush or not, whether the professional should file the affidavit, and the documents to serve. Case number, county, caption and parties are optional and go on the proof of service.
- Court run: which courthouse and city, filing or courtesy copy, and whether it is urgent. These set the price, so Scope does not quote a court run without them.
- Records retrieval: subject name, provider name and location, record types, date range, and the signed authorization. An estimated page count and whether the custodian requires a physical check are optional and only affect the at-cost estimate.
- Court reporting and depositions: location, case caption and proceeding date, plus the reporter type if the user cares (stenographer, digital or voice).
- Federal court records pulls and docket watches, if the `scope_order_court_records` tool is available: these are Scope's own service at Scope's listed price, not a professional's quote, so the steps below about quotes and awards do not apply. The tool records the order and returns the price text. A signed-in person at the firm confirms it on the matter page; nothing is bought, watched or billed before then.

Optionally call `scope_list_vendors` to let the user browse credentialed professionals for a category and area, and `scope_list_roster` to show the firm's own primary, backup and excluded professionals. The roster tier is the firm's own preference, and the reputation figures are the professional's record; do not mix them up.

## 2. Show quotes and each professional's own price

Call `scope_dispatch_matter` with `award` set to `quote_only`. Pass `title`, `matter_type`, `service_category` (the slug from `scope_list_categories`), a plain `description` without client names, the facts from step 1 in `form_field_values`, the jurisdiction in `jurisdictions`, and for a serve the deadline date in `timeline_deadline` with `deadline_semantics` ('on' or 'by') in `form_field_values`. That saves the answers, prices the matter, and returns the quotes. It chooses nobody, asks nobody to approve anything, and contacts no professional. Leave documents out of a quote_only call. Tell the user that documents are uploaded in Scope through the secure upload link Scope provides, which opens only for a signed-in member of the firm.

If the response says `incomplete_intake`, ask the user each question in `field_prompts`, word for word, then call `scope_dispatch_matter` again with `matter_id` set to the returned `scope_id`, the answers in `form_field_values`, and `award` set to `quote_only` again unless the user asked Scope to choose. Do not start a new matter. After a timeout or a lost reply, call `scope_list_matters` before creating the matter again.

Present what comes back plainly:

- Each professional's name and their own price, exactly as returned, in the order returned.
- Scope charges the firm a separate 15 percent fee on the professional's labor. Do not compute or state a total unless the tool gives one; say that the fee is billed on top.
- Any at-cost items the tool names (for example a custodian's copy charges) as at-cost items, not as the professional's fee.
- Anything the tool says is still missing.

Do not choose for the user. Do not call `scope_dispatch_matter` with `award` set to `auto` unless the user explicitly asks Scope to choose and send.

## 3. Prepare the request and explain approval

When the user picks a professional, call `scope_award_matter` with `matter_id` and `vendor_name` set to that professional's name exactly as returned in the quotes. Pass `confirmed_slot` only if the user gave a date or time, and `reporter_name` only if the quote names a reporter. Leave `override_tentative` alone unless the user asks for it.

Before you call it, tell the user what will happen, and after you call it, report what the response says. The facts to state:

- An award requested through Claude does not commit the firm. It is held as a pending approval, and a person at the firm must approve it before anything is committed. Expect `status='pending_approval'`, with an approval id.
- The approver is emailed a signed release link, and the request waits in the approver's queue on Scope's approvals page, which requires signing in. You cannot approve it, you cannot give a release link, and you must not suggest the user skip the step.
- If the response is `no_approver_at_firm`, nothing was committed and nobody was notified. Tell the user the firm needs someone who can approve in Scope.
- If the response shows anything other than `pending_approval`, repeat what it says and nothing more.
- No card is captured and no money moves in this conversation. Scope invoices the firm through Stripe, outside the conversation, after the professional submits the work and the firm accepts it, or the firm's review window ends without action.

Until the approval happens, say the order is waiting on approval. Do not say it was ordered or that a professional is on the way.

## 4. Check progress

When the user asks where things stand:

- One matter: `scope_get_matter` with the matter id such as SC-2041 (example only). It returns status, quotes, award status, the approval state and the execution record.
- Across the firm: `scope_list_matters` for the list, and `scope_attention` for what is waiting on someone and who owns the next step.
- Check `scope_get_messages` when reporting status, because an unanswered question from the professional can block the work. Reading the thread marks the professional's messages as read for the firm, so tell the user you are reading it. To answer, use `scope_send_message` with the user's words, unchanged. Do not promise the professional was called or texted; a message posts to the matter thread and the professional is notified by email.

Report states in plain words using the record: waiting for approval, awaiting a professional, scheduled, attempted, delivered, accepted. Some details need care:

- A step marked "reported" is the professional's own report, not something Scope checked.
- Say work was checked only when the record's check field says so. A serve has no check step; logged attempts and a signed declaration are not a check.
- Never say papers were served until Scope returns the proof. If an attempt failed or the status is unclear, say so and show what Scope shows.
- Never say work is billed or paid unless the record shows it.

To move a booking, `scope_reschedule_project` works on awarded work before delivery. It takes `project_id` (PJ-XXXX, from the awarded matter, not the SC matter id) and `new_date`. Use it only when the user asks. Send a bare date (2026-10-12) or a date-time with its Pacific offset (2026-10-12T10:00:00-07:00). Relay `new_slot_text` as returned, and report the refusal reason if it refuses.

## 5. Retrieve the finished work

- `scope_list_deliverables_for_scope` lists what the professional uploaded for a matter. `scope_get_deliverable` fetches one file's details.
- The `download_url` in a response lasts 5 minutes. Fetch the file when the user asks and call again for a fresh link later. The `portal_url` opens the file on the matter page for a signed-in member of the firm; give that to the user to share a file inside the firm.
- Report a deliverable's status as stored: submitted means it waits for the firm's review; accepted means the firm accepted it. Accepting is the firm's decision in Scope.
- For a serve, `scope_get_service_proof` returns the full attempt trail and the declarations on the matter; `scope_get_declaration` reads one declaration's status. Only a proof of service counts as the served work product; an attempt photo is evidence.
- To email a file to a named person, use `scope_send_deliverable` only when the user asks and names the recipient.
- Analysis of the work belongs to the user and Claude. Scope delivers the file.

## Example (fictional names)

A user says: "Serve the summons and complaint on Pat Example at 100 Sample Street, Anytown, by Friday. The case is Example v. Placeholder, number 00-0000."

1. Confirm the missing items: rush or not, affidavit filing or not, and which documents will be uploaded in Scope.
2. Call `scope_dispatch_matter` with `award` set to `quote_only`.
3. Show each professional's own price and say Scope's 15 percent fee is billed to the firm on top.
4. The user picks one. Say an award waits for a person at the firm to approve it. Call `scope_award_matter`. Report `pending_approval` and the approval id, and say a person at the firm must approve it.
5. Later, check `scope_get_matter` and `scope_get_messages`. When Scope returns proof, fetch it with `scope_get_service_proof` and `scope_get_deliverable`.
