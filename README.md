# 📧 Email Predictor — n8n Workflow

An automated n8n workflow that generates and verifies business email variations for a given contact, using the Reoon Email Verifier API.

---

## 🔍 What It Does

Given a person's **first name**, **last name**, and **company domain**, this workflow:

1. Generates **'n' possible business email variations** (e.g. `john.doe@company.com`, `jdoe@company.com`, etc.)
2. Submits each email to the **Reoon bulk email verification API**
3. Waits for the verification results
4. Filters for **safe/valid emails**
5. Returns the results via webhook response

---

## 🧩 Workflow Nodes

| Node | Type | Description |
|---|---|---|
| **Webhook** | Trigger | Receives incoming POST request to start the workflow |
| **Extract Contact Data** | Set | Defines `firstname`, `lastname`, and `companydomain` |
| **Clean & Format Data** | Code (JS) | Lowercases all string fields for consistency |
| **Code in JavaScript** | Code (JS) | Generates 50 email variations from the contact info |
| **HTTP Request** | HTTP | Submits each email to Reoon bulk verification API |
| **Wait** | Wait | Pauses execution while Reoon processes the emails |
| **HTTP Request1** | HTTP | Fetches the verification result using `task_id` |
| **Filter Safe Emails** | Code (JS) | Filters results to keep only valid/safe emails |
| **Respond to Webhook** | Webhook Response | Sends the final result back to the caller |

---

## 🔄 Flow Diagram

```
Webhook → Extract Contact Data → Clean & Format Data → Code in JavaScript
    → HTTP Request (submit to Reoon) → Wait → HTTP Request1 (get results)
        → Filter Safe Emails → Respond to Webhook
```

---

## 📥 Input

Send a `POST` request to the webhook URL with a JSON body:

```json
{
  "firstname": "Sana",
  "lastname": "Mansoori",
  "companydomain": "acquisitionx.pro"
}
```

> **Note:** Currently, contact data is hardcoded in the `Extract Contact Data` node. Update that node or modify it to read from the webhook body dynamically.

---

## 📤 Output

Returns a list of **verified safe emails** for the given contact and domain.

---

## ⚙️ Setup & Configuration

### 1. Reoon API Key
Replace the API key in the two HTTP Request nodes:
```
key=YOUR_REOON_API_KEY
```
Current key location:
- `HTTP Request` node → JSON body → `"key"` field
- `HTTP Request1` node → URL query param → `key=...`

### 2. Webhook URL
The workflow is triggered via a `POST` request to:
```
/webhook/49650893-3f49-4d63-b2c6-f82eca4af949
```

### 3. Wait Duration
The `Wait` node pauses between submission and result fetching. Adjust the wait time based on how long Reoon takes to verify emails in bulk (recommended: 10–30 seconds).

---

## 📧 Email Variation Examples

From `sana` + `mansoori` + `acquisitionx.pro`, the workflow generates patterns like:

```
sana.mansoori@acquisitionx.pro
smansoori@acquisitionx.pro
sanamansoori@acquisitionx.pro
sana@acquisitionx.pro
s.mansoori@acquisitionx.pro
mansoori@acquisitionx.pro
sm@acquisitionx.pro
... (50 total variations)
```

---

## 🛠️ Dependencies

- [n8n](https://n8n.io/) — Workflow automation platform
- [Reoon Email Verifier API](https://emailverifier.reoon.com/) — Bulk email verification service

---

## ⚠️ Notes

- The `Filter Safe Emails` node currently adds a placeholder field (`myNewField`). **Update this node** to actually filter emails based on Reoon's verification status (e.g. `status === "safe"`).
- API keys are hardcoded — consider using **n8n credentials** or environment variables for production use.
- The workflow processes emails one by one (50 separate API calls). For large-scale use, consider batching.
