# Shared Data Contract (Decoupled Error Dashboard)

**Joint Owners:** Dicle (xAPI / Log Preparation) & Megane (Direct Capture & Dashboard UI)  
**Version:** 1.1  

This contract defines the unified JSON schema consumed by the dashboard, regardless of whether records originate from **Dicle's Juxl xAPI exporter** or **Megane's direct JupyterLab execution listener**.

---

## 1. Root Payload Structure

```json
{
  "schemaVersion": "1.1",
  "sourceType": "hybrid",
  "sessionId": "session-2026-11-21-01",
  "sessionStartTime": "2026-11-21T10:10:00Z",
  "sessionEndTime": "2026-11-21T10:25:00Z",
  "privacyMode": "pseudonymous",
  "events": [
    {
      "eventId": "evt-001",
      "timestamp": "2026-11-21T10:15:30Z",
      "cellId": "cell-3",
      "executionCount": 4,
      "outcome": "error",
      "category": "warning",
      "errorType": "KeyError",
      "errorSubPattern": "dict_key_not_found",
      "rawMessage": "'alter'",
      "plainExplanation": "You are trying to access the key 'alter' in a dictionary, but this key does not exist there.",
      "codeContext": {
        "sourceLines": [
          "user = {'name': 'Alex', 'city': 'Bochum'}",
          "print(user['alter'])"
        ],
        "errorLine": 2,
        "columnOffset": 11,
        "faultyToken": "'alter'"
      },
      "variableSnapshot": [
        { "name": "user", "type": "dict", "value": "{'name': 'Alex', 'city': 'Bochum'}" }
      ],
      "fixLink": {
        "status": "resolved",
        "fixEventId": "evt-002",
        "resolvedTimestamp": "2026-11-21T10:16:12Z",
        "activeTimeToFixSeconds": 42,
        "idleSecondsExcluded": 0,
        "brokenCode": "print(user['alter'])",
        "fixedCode": "print(user.get('alter'))"
      }
    },
    {
      "eventId": "evt-002",
      "timestamp": "2026-11-21T10:16:12Z",
      "cellId": "cell-3",
      "executionCount": 5,
      "outcome": "success",
      "category": null,
      "errorType": null,
      "errorSubPattern": null,
      "rawMessage": null,
      "plainExplanation": null,
      "codeContext": {
        "sourceLines": [
          "user = {'name': 'Alex', 'city': 'Bochum'}",
          "print(user.get('alter'))"
        ],
        "errorLine": null,
        "columnOffset": null,
        "faultyToken": null
      },
      "variableSnapshot": null,
      "fixLink": null
    }
  ]
}
```

---

## 2. Field Specifications

| Field Path | Type | Required? | Description & Allowed Values |
| :--- | :--- | :--- | :--- |
| `schemaVersion` | `string` | **Required** | Contract version (`"1.1"`). |
| `sourceType` | `string` | **Required** | `"juxl_import"`, `"direct_capture"`, or `"hybrid"`. |
| `sessionId` | `string` | **Required** | Unique session identifier. |
| `events` | `Array<Event>` | **Required** | Chronologically sorted array of both `error` and `success` cell runs (needed for the Timeline and Past Solutions). |
| `events[].eventId` | `string` | **Required** | Unique event ID. |
| `events[].timestamp` | `string` | **Required** | ISO-8601 timestamp (`YYYY-MM-DDTHH:mm:ssZ`). |
| `events[].cellId` | `string` | **Required** | Jupyter cell ID or index label. |
| `events[].outcome` | `string` | **Required** | `"error"` or `"success"` (enables green success markers & resolution arcs on the timeline). |
| `events[].category` | `string \| null` | Conditional | Required if `outcome === "error"`: `"critical"` (🔴), `"warning"` (🟡), or `"hint"` (🔵). `null` if `success`. |
| `events[].errorType` | `string \| null` | Conditional | Required if `outcome === "error"` (e.g., `"KeyError"`, `"SyntaxError"`). Used to look up hover tooltips and group Past Solutions. |
| `events[].errorSubPattern` | `string \| null` | Optional | Fine-grained cause tag (e.g., `"missing_colon"`, `"dict_key_not_found"`) for sharper past-solution matching. |
| `events[].rawMessage` | `string \| null` | Conditional | Python exception message when `outcome === "error"`. |
| `events[].plainExplanation` | `string \| null` | Optional | Custom explanation; if `null`, frontend falls back to `error_catalog.json`. |
| `events[].codeContext` | `object \| null` | Optional | Full `sourceLines` array plus `errorLine` and `columnOffset`. |
| `events[].variableSnapshot` | `Array<Var> \| null` | Optional | Runtime variables `{ name, type, value }` captured on crash. |
| `events[].fixLink` | `object \| null` | Optional | Links an error to its resolving run (`fixEventId`, `brokenCode`, `fixedCode`, `activeTimeToFixSeconds`). |

---

## 3. Bundled Tooltip Knowledge Base (`error_catalog.json`)

To guarantee instant hover tooltips without external APIs or Juxl metadata, Megane maintains a static lookup file keyed by `errorType`:

```json
{
  "KeyError": {
    "defaultCategory": "warning",
    "shortTooltip": "You tried to access a dictionary key that does not exist.",
    "vocabDefinition": "A KeyError happens when Python looks for a specific key inside a dict, but only other keys are present.",
    "checklist": [
      "Did you spell the key name exactly right (including uppercase/lowercase)?",
      "Did you check which keys are actually inside the dictionary?"
    ],
    "exampleBefore": "d = {'age': 20}\nprint(d['alter'])",
    "exampleAfter": "d = {'age': 20}\nprint(d.get('alter'))"
  }
}
```

---

## 4. Ordering, Missing Values & Error Handling

* **Chronological Ordering:** `events[]` must be sorted ascending by `timestamp`.
* **Idle Gap Detection:** If `timestamp[i] - timestamp[i-1] > 120 seconds`, the timeline renders a visual break marker and excludes idle time from `activeTimeToFixSeconds`.
* **Graceful Degradation (Juxl-Only Records):** If an imported Juxl xAPI record lacks `codeContext.sourceLines` or `variableSnapshot`, the record remains valid; `CategorizedPastSolutionsView` simply omits entries without `brokenCode`/`fixedCode` diffs rather than showing empty cards.
* **Invalid Input Handling:** Malformed JSON files show a non-blocking banner and preserve the previous valid session state.
