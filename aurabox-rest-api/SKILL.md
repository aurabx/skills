---
name: "Aurabox REST API"
description: "Generate client code and integrations for the Aurabox REST API. Covers patients, cases (de-identified patients), and studies endpoints with Bearer token authentication. Use when building Aurabox API integrations, querying patient or study data, creating API clients, or working with Aurabox programmatically."
---

# Aurabox REST API

## What This Skill Does

Generates correct, working client code for the Aurabox Cloud REST API. Knows the full API surface -- patients, cases, studies -- including authentication, pagination, and error handling. Produces idiomatic code in Python, TypeScript/JavaScript, PHP, and curl.

## Prerequisites

- An Aurabox account with API access enabled
- An API key (generated from the Aurabox dashboard under your team settings)
- Familiarity with REST APIs

---

## Quick Reference

### Base URL

```
https://au.aurabox.cloud
```

### Authentication

All requests require a Bearer token in the `Authorization` header:

```
Authorization: Bearer {YOUR_AUTH_KEY}
Content-Type: application/json
Accept: application/json
```

API keys are generated in the Aurabox web interface. They are scoped to a team realm.

**Security**: API keys are sensitive. Never commit them to source control. Use environment variables or a secrets manager.

### OpenAPI / Postman

Machine-readable specs are available:
- OpenAPI: `https://au.aurabox.app/docs/openapi.yaml`
- Postman collection: `https://au.aurabox.app/docs/collection.json`

---

## API Surface

### Patients

Full CRUD for patient records within the authenticated team realm.

#### Data Model

```json
{
  "id": "003fb6c7-a399-46de-8199-ac51e031bd10",
  "given_names": "Jessica",
  "family_name": "Jones",
  "date_of_birth": "1985-06-15",
  "sex": "female",
  "address": {
    "street": "123 Main Street",
    "city": "Sydney",
    "region": "NSW",
    "postcode": "2000",
    "country": "au"
  },
  "status": "active",
  "archived": false,
  "created_at": "2025-01-15T10:30:00.000000Z",
  "updated_at": "2025-01-15T10:30:00.000000Z"
}
```

#### Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/patients` | List patients (paginated, searchable, sortable) |
| `POST` | `/api/v1/patients` | Create a patient |
| `GET` | `/api/v1/patients/{id}` | Retrieve a patient |
| `PUT` | `/api/v1/patients/{id}` | Update a patient |
| `DELETE` | `/api/v1/patients/{id}` | Delete a patient |

#### List Parameters

| Parameter | Type | Values |
|-----------|------|--------|
| `per_page` | integer | 1-100 |
| `search` | string | Max 255 chars |
| `sort` | string | `name`, `created_at`, `updated_at`, `date_of_birth` |
| `direction` | string | `asc`, `desc` |
| `archived` | boolean | `true`, `false` |

#### Create/Update Fields

| Field | Type | Required (create) | Notes |
|-------|------|-------------------|-------|
| `given_names` | string | Yes | Max 255 chars |
| `family_name` | string | Yes | Max 255 chars |
| `date_of_birth` | string | Yes | Date format, must be before today |
| `sex` | string | Yes | `male`, `female`, `other`, `unknown` |
| `address` | object | No | Contains `street`, `city`, `region`, `postcode` (max 20), `country` (max 2, ISO) |

### Cases (De-identified Patients)

Cases are patients whose identity has been removed. Only a label and non-identifying metadata are stored.

#### Data Model

```json
{
  "id": "003fb6c7-a399-46de-8199-ac51e031bd10",
  "label": "CASE-00142",
  "status": "deidentified",
  "archived": false,
  "created_at": "2025-01-15T10:30:00.000000Z",
  "updated_at": "2025-01-15T10:30:00.000000Z"
}
```

#### Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/cases` | List cases (paginated, searchable, sortable) |
| `POST` | `/api/v1/cases` | Create a case |
| `GET` | `/api/v1/cases/{patient_id}` | Show a case |
| `PUT` | `/api/v1/cases/{patient_id}` | Update a case |
| `DELETE` | `/api/v1/cases/{patient_id}` | Delete a case |

#### List Parameters

| Parameter | Type | Values |
|-----------|------|--------|
| `per_page` | integer | 1-100 |
| `search` | string | Max 255 chars (searches label) |
| `sort` | string | `label`, `created_at`, `updated_at`, `date_of_birth` |
| `direction` | string | `asc`, `desc` |
| `archived` | boolean | `true`, `false` |

#### Create/Update Fields

| Field | Type | Required (create) | Notes |
|-------|------|-------------------|-------|
| `label` | string | Yes | Max 255 chars |
| `sex` | string | No | `male`, `female`, `other`, `unknown` |
| `date_of_birth` | string | No | Date format, must be before today |

### Studies

Studies are nested under patients. Each study represents a medical imaging study (e.g., a CT scan, MRI, X-ray).

#### Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/patients/{patient_id}/studies` | List studies for a patient |
| `POST` | `/api/v1/patients/{patient_id}/studies` | Create a study |
| `GET` | `/api/v1/patients/{patient_id}/studies/{id}` | Retrieve a study |
| `PUT` | `/api/v1/patients/{patient_id}/studies/{id}` | Update a study |
| `DELETE` | `/api/v1/patients/{patient_id}/studies/{id}` | Delete a study |

---

## Pagination

All list endpoints return paginated responses using Laravel-style pagination:

```json
{
  "data": [ ... ],
  "links": {
    "first": "https://au.aurabox.cloud/api/v1/patients?page=1",
    "last": "https://au.aurabox.cloud/api/v1/patients?page=5",
    "prev": null,
    "next": "https://au.aurabox.cloud/api/v1/patients?page=2"
  },
  "meta": {
    "current_page": 1,
    "from": 1,
    "last_page": 5,
    "per_page": 25,
    "to": 25,
    "total": 112
  }
}
```

To paginate, follow the `links.next` URL or increment the `page` query parameter.

---

## Code Examples

### Python Client

```python
import requests
import os

BASE_URL = "https://au.aurabox.cloud"


class AuraboxClient:
    """Client for the Aurabox REST API."""

    def __init__(self, api_key: str):
        self.base_url = BASE_URL
        self.session = requests.Session()
        self.session.headers.update({
            "Authorization": f"Bearer {api_key}",
            "Content-Type": "application/json",
            "Accept": "application/json",
        })

    def _request(self, method: str, path: str, **kwargs) -> dict:
        url = f"{self.base_url}{path}"
        response = self.session.request(method, url, **kwargs)
        response.raise_for_status()
        if response.status_code == 204:
            return {}
        return response.json()

    # --- Patients ---

    def list_patients(self, search: str = None, per_page: int = 25,
                      sort: str = "created_at", direction: str = "desc",
                      archived: bool = False) -> dict:
        params = {"per_page": per_page, "sort": sort,
                  "direction": direction, "archived": archived}
        if search:
            params["search"] = search
        return self._request("GET", "/api/v1/patients", params=params)

    def get_patient(self, patient_id: str) -> dict:
        return self._request("GET", f"/api/v1/patients/{patient_id}")

    def create_patient(self, given_names: str, family_name: str,
                       date_of_birth: str, sex: str,
                       address: dict = None) -> dict:
        body = {
            "given_names": given_names,
            "family_name": family_name,
            "date_of_birth": date_of_birth,
            "sex": sex,
        }
        if address:
            body["address"] = address
        return self._request("POST", "/api/v1/patients", json=body)

    def update_patient(self, patient_id: str, **fields) -> dict:
        return self._request("PUT", f"/api/v1/patients/{patient_id}", json=fields)

    def delete_patient(self, patient_id: str) -> dict:
        return self._request("DELETE", f"/api/v1/patients/{patient_id}")

    # --- Cases (de-identified patients) ---

    def list_cases(self, search: str = None, per_page: int = 25,
                   sort: str = "created_at", direction: str = "desc",
                   archived: bool = False) -> dict:
        params = {"per_page": per_page, "sort": sort,
                  "direction": direction, "archived": archived}
        if search:
            params["search"] = search
        return self._request("GET", "/api/v1/cases", params=params)

    def get_case(self, case_id: str) -> dict:
        return self._request("GET", f"/api/v1/cases/{case_id}")

    def create_case(self, label: str, sex: str = None,
                    date_of_birth: str = None) -> dict:
        body = {"label": label}
        if sex:
            body["sex"] = sex
        if date_of_birth:
            body["date_of_birth"] = date_of_birth
        return self._request("POST", "/api/v1/cases", json=body)

    def update_case(self, case_id: str, **fields) -> dict:
        return self._request("PUT", f"/api/v1/cases/{case_id}", json=fields)

    def delete_case(self, case_id: str) -> dict:
        return self._request("DELETE", f"/api/v1/cases/{case_id}")

    # --- Studies (nested under patients) ---

    def list_studies(self, patient_id: str) -> dict:
        return self._request("GET", f"/api/v1/patients/{patient_id}/studies")

    def get_study(self, patient_id: str, study_id: str) -> dict:
        return self._request("GET",
            f"/api/v1/patients/{patient_id}/studies/{study_id}")

    def create_study(self, patient_id: str, **fields) -> dict:
        return self._request("POST",
            f"/api/v1/patients/{patient_id}/studies", json=fields)

    def update_study(self, patient_id: str, study_id: str, **fields) -> dict:
        return self._request("PUT",
            f"/api/v1/patients/{patient_id}/studies/{study_id}", json=fields)

    def delete_study(self, patient_id: str, study_id: str) -> dict:
        return self._request("DELETE",
            f"/api/v1/patients/{patient_id}/studies/{study_id}")

    # --- Pagination helper ---

    def paginate(self, method, *args, **kwargs):
        """Iterate through all pages of a paginated endpoint."""
        response = method(*args, **kwargs)
        while True:
            yield from response.get("data", [])
            next_url = response.get("links", {}).get("next")
            if not next_url:
                break
            response = self.session.get(next_url).json()


# Usage
client = AuraboxClient(api_key=os.environ["AURABOX_API_KEY"])
patients = client.list_patients(search="Jones")
```

### TypeScript Client

```typescript
const AURABOX_BASE_URL = "https://au.aurabox.cloud";

class AuraboxClient {
  private baseUrl: string;
  private headers: Record<string, string>;

  constructor(apiKey: string) {
    this.baseUrl = AURABOX_BASE_URL;
    this.headers = {
      Authorization: `Bearer ${apiKey}`,
      "Content-Type": "application/json",
      Accept: "application/json",
    };
  }

  private async request<T>(
    method: string,
    path: string,
    options?: { body?: unknown; params?: Record<string, unknown> },
  ): Promise<T> {
    let url = `${this.baseUrl}${path}`;
    if (options?.params) {
      const searchParams = new URLSearchParams();
      for (const [key, value] of Object.entries(options.params)) {
        if (value !== undefined && value !== null) {
          searchParams.set(key, String(value));
        }
      }
      const qs = searchParams.toString();
      if (qs) url += `?${qs}`;
    }
    const response = await fetch(url, {
      method,
      headers: this.headers,
      body: options?.body ? JSON.stringify(options.body) : undefined,
    });
    if (!response.ok) {
      throw new Error(`Aurabox API error: ${response.status} ${response.statusText}`);
    }
    if (response.status === 204) return {} as T;
    return response.json();
  }

  // Patients
  listPatients(params?: ListParams) {
    return this.request<PaginatedResponse<Patient>>("GET", "/api/v1/patients", { params });
  }
  getPatient(id: string) {
    return this.request<{ data: Patient }>("GET", `/api/v1/patients/${id}`);
  }
  createPatient(patient: CreatePatient) {
    return this.request<{ data: Patient }>("POST", "/api/v1/patients", { body: patient });
  }
  updatePatient(id: string, fields: Partial<CreatePatient>) {
    return this.request<{ data: Patient }>("PUT", `/api/v1/patients/${id}`, { body: fields });
  }
  deletePatient(id: string) {
    return this.request<void>("DELETE", `/api/v1/patients/${id}`);
  }

  // Cases
  listCases(params?: ListParams) {
    return this.request<PaginatedResponse<Case>>("GET", "/api/v1/cases", { params });
  }
  getCase(id: string) {
    return this.request<{ data: Case }>("GET", `/api/v1/cases/${id}`);
  }
  createCase(caseData: CreateCase) {
    return this.request<{ data: Case }>("POST", "/api/v1/cases", { body: caseData });
  }

  // Studies
  listStudies(patientId: string) {
    return this.request<PaginatedResponse<Study>>("GET",
      `/api/v1/patients/${patientId}/studies`);
  }
  getStudy(patientId: string, studyId: string) {
    return this.request<{ data: Study }>("GET",
      `/api/v1/patients/${patientId}/studies/${studyId}`);
  }
}

// Types
interface Patient {
  id: string;
  given_names: string;
  family_name: string;
  date_of_birth: string;
  sex: "male" | "female" | "other" | "unknown";
  address?: Address;
  status: string;
  archived: boolean;
  created_at: string;
  updated_at: string;
}

interface Case {
  id: string;
  label: string;
  status: "deidentified";
  archived: boolean;
  created_at: string;
  updated_at: string;
}

interface Study {
  id: string;
  [key: string]: unknown; // Study fields vary
}

interface Address {
  street?: string;
  city?: string;
  region?: string;
  postcode?: string;
  country?: string;
}

interface CreatePatient {
  given_names: string;
  family_name: string;
  date_of_birth: string;
  sex: "male" | "female" | "other" | "unknown";
  address?: Address;
}

interface CreateCase {
  label: string;
  sex?: "male" | "female" | "other" | "unknown";
  date_of_birth?: string;
}

interface ListParams {
  per_page?: number;
  search?: string;
  sort?: string;
  direction?: "asc" | "desc";
  archived?: boolean;
}

interface PaginatedResponse<T> {
  data: T[];
  links: { first: string; last: string; prev: string | null; next: string | null };
  meta: { current_page: number; from: number; last_page: number; per_page: number; to: number; total: number };
}
```

### curl

```bash
# List patients
curl -s "https://au.aurabox.cloud/api/v1/patients" \
  -H "Authorization: Bearer $AURABOX_API_KEY" \
  -H "Accept: application/json" | jq

# Create a case
curl -s -X POST "https://au.aurabox.cloud/api/v1/cases" \
  -H "Authorization: Bearer $AURABOX_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{"label": "STUDY-2025-001"}' | jq

# Get studies for a patient
curl -s "https://au.aurabox.cloud/api/v1/patients/{patient_id}/studies" \
  -H "Authorization: Bearer $AURABOX_API_KEY" \
  -H "Accept: application/json" | jq
```

---

## Error Handling

The API returns standard HTTP status codes:

| Code | Meaning |
|------|---------|
| `200` | Success |
| `201` | Created |
| `204` | Deleted (no content) |
| `401` | Unauthorized (bad or missing API key) |
| `404` | Not found |
| `422` | Validation error (check response body for details) |
| `429` | Rate limited |
| `500` | Server error |

Error responses include a `message` field:

```json
{
  "message": "Not found"
}
```

Validation errors include field-level detail:

```json
{
  "message": "The given data was invalid.",
  "errors": {
    "given_names": ["The given names field is required."],
    "family_name": ["The family name field is required."]
  }
}
```

---

## Common Patterns

### Iterate All Patients

```python
# Python: paginate through all patients
all_patients = list(client.paginate(client.list_patients))
```

### Search and Filter

```python
# Find active patients named "Smith"
results = client.list_patients(search="Smith", archived=False)
```

### Bulk Case Creation (Research)

```python
# Create cases for a research cohort
import csv

with open("cohort.csv") as f:
    for row in csv.DictReader(f):
        client.create_case(
            label=row["study_id"],
            sex=row.get("sex"),
            date_of_birth=row.get("dob"),
        )
```

---

## Gotchas

- **IDs are UUIDs** -- not sequential integers. Always use the `id` field from API responses.
- **Patient vs Case** -- Patients have PII (names, addresses). Cases are de-identified (label only). Use cases for research data.
- **Studies are nested** -- Study endpoints require a `patient_id` in the URL path. You cannot query studies globally.
- **Dates must be before today** -- The API rejects future dates for `date_of_birth`.
- **Country is 2-char ISO** -- The `address.country` field accepts ISO 3166-1 alpha-2 codes only (e.g., `au`).

---

## Resources

- [Aurabox Documentation](https://docs.aurabox.cloud)
- [API Docs](https://au.aurabox.app/docs)
- [OpenAPI Spec](https://au.aurabox.app/docs/openapi.yaml)
- [Postman Collection](https://au.aurabox.app/docs/collection.json)
