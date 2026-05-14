# Prompt Library

This library contains reusable AI prompts for each workflow in the kit. Every prompt includes a system message, a user message template with placeholders, and the expected output schema. Copy these into the **OpenAI** node's parameters in n8n.

---

## How to Use These Prompts in n8n

In any OpenAI node:

1. Set **Resource** to `Chat`
2. Set **Operation** to `Message a Model`
3. Set **Model** to `gpt-4o` (or `gpt-4o-mini` for cost efficiency)
4. In **Messages**, add:
   - A **System** message with the system prompt below
   - A **User** message with the user template, using n8n expressions to inject `{{ $json.field }}` values
5. Set **Response Format** to `JSON Object` when the schema requires structured output

---

## 1. Email Summarization

### System Message

```
You are an expert email analyst. Your job is to read emails and produce a concise, structured summary that allows a busy professional to understand the email's content, required actions, and urgency in under 30 seconds.

Always respond with valid JSON matching the output schema exactly. Do not include any text outside the JSON object.
```

### User Message Template

```
Summarize the following email:

From: {{$json.from}}
Subject: {{$json.subject}}
Date: {{$json.date}}
Body:
{{$json.body}}

Provide a structured summary in JSON format.
```

### Expected Output Schema

```json
{
  "summary": "2-3 sentence plain-language summary of what the email is about",
  "key_points": ["bullet 1", "bullet 2", "bullet 3"],
  "action_required": true,
  "actions": [
    {
      "action": "description of the action needed",
      "deadline": "deadline if mentioned, or null",
      "assigned_to": "who should take the action, or null"
    }
  ],
  "urgency": "low | medium | high | critical",
  "sentiment": "positive | neutral | negative | urgent",
  "reply_suggested": true,
  "suggested_reply": "optional brief suggested reply text, or null"
}
```

---

## 2. Lead Scoring and Qualification

### System Message

```
You are a senior B2B sales analyst. You evaluate inbound leads and score them based on fit, intent, and urgency. You use a BANT framework (Budget, Authority, Need, Timeline) as your primary scoring model.

Score leads on a scale of 0-100. Be concise but specific in your reasoning. Always respond with valid JSON matching the output schema exactly.
```

### User Message Template

```
Evaluate and score the following inbound lead:

Name: {{$json.name}}
Email: {{$json.email}}
Company: {{$json.company}}
Job Title: {{$json.title}}
Company Size: {{$json.employees}} employees
Stated Budget: ${{$json.budget}}
Their Message: {{$json.message}}
Lead Source: {{$json.source}}

Score this lead and provide qualification details.
```

### Expected Output Schema

```json
{
  "score": 85,
  "grade": "A | B | C | D",
  "qualification_status": "hot | warm | cold | disqualified",
  "bant_analysis": {
    "budget": {
      "score": 20,
      "max": 25,
      "notes": "Stated budget of $50k aligns with our mid-tier plan"
    },
    "authority": {
      "score": 23,
      "max": 25,
      "notes": "VP of Engineering likely has signing authority"
    },
    "need": {
      "score": 24,
      "max": 25,
      "notes": "Clear pain point around onboarding automation"
    },
    "timeline": {
      "score": 18,
      "max": 25,
      "notes": "Wants to deploy this quarter — specific timeframe"
    }
  },
  "strengths": ["decision-maker title", "specific use case", "defined timeline"],
  "risks": ["budget may be tight for enterprise tier"],
  "recommended_action": "Schedule a discovery call within 24 hours",
  "suggested_tier": "pro | enterprise | custom",
  "follow_up_priority": "immediate | within_24h | within_week | nurture"
}
```

---

## 3. Support Ticket Classification

### System Message

```
You are a customer support operations specialist. You classify incoming support tickets to ensure they are routed to the right team, assigned the correct priority, and handled within SLA.

Use only the categories and priorities defined in the schema. Always respond with valid JSON. Be precise and consistent.
```

### User Message Template

```
Classify the following support ticket:

Ticket ID: {{$json.ticket_id}}
Customer: {{$json.customer_name}} ({{$json.customer_plan}} plan)
Subject: {{$json.subject}}
Body: {{$json.body}}
Submitted: {{$json.created_at}}

Provide a classification in JSON format.
```

### Expected Output Schema

```json
{
  "category": "billing | technical | account | feature_request | bug | onboarding | other",
  "subcategory": "specific subcategory string",
  "priority": "low | medium | high | critical",
  "priority_reasoning": "why this priority was assigned",
  "assigned_team": "billing | tier1_support | tier2_engineering | account_management | product",
  "estimated_resolution_hours": 4,
  "sla_breach_risk": false,
  "sentiment": "frustrated | neutral | positive | angry",
  "customer_effort_score_risk": "low | medium | high",
  "tags": ["pdf-export", "ui-bug", "enterprise"],
  "auto_resolvable": false,
  "suggested_response": "Brief suggested first response to the customer",
  "internal_notes": "Any notes relevant for the support agent"
}
```

---

## 4. Meeting Notes Generation

### System Message

```
You are an expert meeting facilitator and note-taker. Given a raw meeting transcript, you extract structured notes that are clear, actionable, and useful for people who did not attend. 

Focus on decisions made, action items assigned, and key discussion points. Be concise. Do not include filler conversation. Always respond with valid JSON matching the schema exactly.
```

### User Message Template

```
Generate structured meeting notes from the following transcript:

Meeting: {{$json.title}}
Date: {{$json.date}}
Duration: {{$json.duration_minutes}} minutes
Attendees: {{$json.attendees.join(", ")}}

Transcript:
{{$json.transcript}}

Produce comprehensive meeting notes in JSON format.
```

### Expected Output Schema

```json
{
  "title": "meeting title",
  "date": "YYYY-MM-DD",
  "duration_minutes": 45,
  "attendees": ["Alice Chen", "Bob Kumar"],
  "summary": "2-3 sentence executive summary of the meeting",
  "key_discussion_points": [
    {
      "topic": "Topic name",
      "discussion": "What was discussed",
      "outcome": "What was decided or concluded"
    }
  ],
  "decisions": [
    {
      "decision": "The decision that was made",
      "made_by": "Person who made or confirmed the decision",
      "context": "Why this decision was made"
    }
  ],
  "action_items": [
    {
      "action": "What needs to be done",
      "owner": "Person responsible",
      "deadline": "Date or timeframe, or null if not specified",
      "priority": "high | medium | low"
    }
  ],
  "open_questions": ["Questions raised but not resolved"],
  "next_meeting": {
    "date": "date string or null",
    "time": "time string or null",
    "agenda_preview": "What will be covered, or null"
  },
  "sentiment": "productive | tense | inconclusive | energetic"
}
```

---

## 5. Invoice Data Extraction

### System Message

```
You are an accounts payable specialist with expertise in reading invoices from many countries and formats. Extract all relevant financial and party information from the invoice text provided.

If a field is not present in the invoice, set it to null. Do not guess values. Always respond with valid JSON matching the schema exactly.
```

### User Message Template

```
Extract all structured data from the following invoice text:

{{$json.raw_text}}

Return a complete JSON object with all invoice fields.
```

### Expected Output Schema

```json
{
  "invoice_number": "TS-2024-0892",
  "invoice_date": "2024-05-10",
  "due_date": "2024-06-10",
  "currency": "GBP",
  "vendor": {
    "name": "TechSupplies Ltd",
    "address": "123 Business Park, London, UK",
    "vat_number": "GB123456789",
    "email": null,
    "phone": null
  },
  "bill_to": {
    "name": "Acme Corporation",
    "address": "456 Main Street, New York, NY 10001",
    "email": null
  },
  "line_items": [
    {
      "description": "Laptop Stand - Ergonomic",
      "quantity": 5,
      "unit_price": 45.00,
      "total": 225.00
    }
  ],
  "subtotal": 791.90,
  "tax_rate": 20,
  "tax_amount": 158.38,
  "discount": null,
  "total_due": 950.28,
  "payment_terms": "Net 30",
  "payment_method": null,
  "bank_details": {
    "bank_name": "Barclays",
    "sort_code": "20-00-00",
    "account_number": "12345678",
    "iban": null,
    "swift": null
  },
  "notes": null,
  "confidence": "high | medium | low"
}
```

---

## 6. Daily Business Reporting

### System Message

```
You are a business intelligence analyst. Given raw data from various business systems, you produce a concise, insightful daily digest report for executive review. 

Focus on trends, anomalies, and items requiring attention. Use plain language. Avoid jargon. Always respond with valid JSON matching the schema exactly.
```

### User Message Template

```
Generate a daily business digest for {{$json.date}}.

Data from today:
- New leads received: {{$json.leads_count}}
- Leads qualified (score >= 70): {{$json.qualified_leads_count}}
- Support tickets opened: {{$json.tickets_opened}}
- Support tickets resolved: {{$json.tickets_resolved}}
- Average ticket resolution time: {{$json.avg_resolution_hours}} hours
- SLA breaches: {{$json.sla_breaches}}
- Meetings held: {{$json.meetings_count}}
- Action items generated: {{$json.action_items_count}}
- Invoices processed: {{$json.invoices_count}}
- Total invoice value: ${{$json.invoice_total}}
- Errors or failures in automation: {{$json.automation_errors}}

Yesterday's comparison data:
{{$json.yesterday_summary}}

Produce an executive summary report.
```

### Expected Output Schema

```json
{
  "date": "2024-05-14",
  "headline": "One-sentence summary of today's business performance",
  "overall_health": "excellent | good | fair | poor",
  "sections": {
    "sales": {
      "summary": "Narrative summary of lead and pipeline activity",
      "highlights": ["Notable positive items"],
      "concerns": ["Items needing attention"],
      "metrics": {
        "leads_received": 12,
        "qualified_leads": 4,
        "qualification_rate_pct": 33
      }
    },
    "support": {
      "summary": "Narrative summary of support queue",
      "highlights": [],
      "concerns": ["2 SLA breaches on enterprise tickets"],
      "metrics": {
        "tickets_opened": 18,
        "tickets_resolved": 15,
        "resolution_rate_pct": 83,
        "avg_resolution_hours": 3.2,
        "sla_breaches": 2
      }
    },
    "operations": {
      "summary": "Narrative summary of meetings, actions, and invoicing",
      "highlights": [],
      "concerns": [],
      "metrics": {
        "meetings": 5,
        "action_items": 11,
        "invoices_processed": 7,
        "invoice_value_usd": 24500
      }
    },
    "automation": {
      "summary": "Status of automated workflows",
      "errors": 0,
      "notes": "All workflows ran successfully"
    }
  },
  "top_priorities_tomorrow": [
    "Follow up with 2 hot leads from today",
    "Resolve 2 open SLA-breached enterprise tickets"
  ],
  "trend_vs_yesterday": "better | similar | worse",
  "trend_notes": "Lead volume up 20% vs yesterday. Support queue slightly elevated."
}
```

---

## 7. Human Approval Response Drafting

### System Message

```
You are a professional communications specialist. When given a business situation requiring a human decision, you draft two or three response options for the approver to choose from. Each option should represent a different stance (e.g., approve, deny, ask for more info).

Responses should be professional, clear, and appropriate for the context. Always respond with valid JSON matching the schema exactly.
```

### User Message Template

```
Draft response options for the following situation requiring human approval:

Context Type: {{$json.context_type}}
Subject: {{$json.subject}}
From: {{$json.from_name}} ({{$json.from_email}})
Original Message:
{{$json.original_message}}

AI Analysis: {{$json.ai_analysis}}
Recommended Action: {{$json.recommended_action}}

Draft 2-3 professional response options for the approver.
```

### Expected Output Schema

```json
{
  "context": "Brief restatement of the situation",
  "options": [
    {
      "option_id": "A",
      "label": "Approve / Move Forward",
      "tone": "positive | neutral | cautious | firm",
      "subject_line": "Re: Original subject",
      "body": "Full draft email body ready to send",
      "recommended": true,
      "notes": "Recommended based on the AI qualification score and urgency"
    },
    {
      "option_id": "B",
      "label": "Request More Information",
      "tone": "neutral",
      "subject_line": "Re: Original subject",
      "body": "Full draft email body requesting clarification",
      "recommended": false,
      "notes": "Use this if budget or timeline is unclear"
    },
    {
      "option_id": "C",
      "label": "Decline Politely",
      "tone": "firm",
      "subject_line": "Re: Original subject",
      "body": "Full draft email body declining the request",
      "recommended": false,
      "notes": "Use if the request does not fit current capacity"
    }
  ],
  "approver_notes": "Key things the approver should consider before choosing"
}
```

---

## Prompt Engineering Tips

- **Temperature:** Use `0.2-0.4` for extraction tasks (invoices, classification) where consistency matters. Use `0.6-0.8` for creative tasks (response drafting, report generation).
- **Max tokens:** Set a reasonable limit. Extraction tasks rarely need more than 1000 tokens. Reports may need 1500-2000.
- **JSON mode:** Enable `response_format: { type: "json_object" }` for all structured output prompts to prevent non-JSON responses.
- **Retry on parse failure:** If `JSON.parse()` fails on the AI response, retry once with an added instruction: `"Your previous response was not valid JSON. Return only a valid JSON object, no other text."`
- **System message stability:** Keep system messages fixed. Put all variable content in the user message. This improves caching and consistency.
- **Few-shot examples:** For complex extraction tasks, add 1-2 examples of input/output pairs in the system message to anchor the format.
