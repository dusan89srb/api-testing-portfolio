# API Testing Portfolio – Restful-booker

Exploratory and negative testing of the [Restful-booker](https://restful-booker.herokuapp.com/apidoc/index.html) REST API, a public practice API for testers. The goal of this project is to show how I approach API testing: understanding the contract, checking both the response and the actual effect on the data, and documenting defects clearly.

**Current status:** manual API testing with Postman (done) → automated test suite with Python + pytest (in progress).

## Key results

25 checks executed: 12 passed, **13 defects found**, 3 of them high severity.

| Severity | Finding |
|---|---|
| High | Wrong data type (`"totalprice": "Sto"`) is silently converted to `null` and saved (data corruption) |
| High | Missing required field on `POST /booking` crashes the server (500) instead of returning 400, while `PUT` handles the same case correctly |
| High | Business rule not validated: a booking with checkout before checkin is accepted and saved |
| Medium | Authentication failures return 403 instead of 401 (no credentials, invalid or expired token) |
| Medium | 404 and 405 are swapped (non-existent ID → 405, missing ID → 404) |
| Low | Wrong success codes (`GET /ping` and `DELETE` return 201, `POST` returns 200) and `418 I'm a teapot` for unsupported formats |

Full list with expected vs. actual results: **[Findings.md](Findings.md)**

## What is covered

| Area | Endpoints / checks |
|---|---|
| Health check | `GET /ping` |
| Authentication | `POST /auth` (token), Basic Auth, missing / invalid / expired credentials |
| CRUD | `GET /booking`, `GET /booking/{id}`, `POST /booking`, `PUT /booking/{id}`, `PATCH /booking/{id}`, `DELETE /booking/{id}` |
| Negative tests | Missing required fields, wrong data types, invalid business rules, malformed JSON, invalid IDs, unsupported methods, content negotiation (`Accept` header) |
| Data verification | Every create / update / delete is confirmed with a follow-up `GET` |

## Testing approach

- **Verify the effect, not only the status code.** A `DELETE` returning 201 is a status bug, but the booking is really deleted; a `POST` returning 200 with `null` price is a data bug. These are reported separately.
- **Equivalence partitioning and boundary values** for input fields (valid number, zero, negative, wrong type).
- **No hardcoded test data.** The shared server resets its data, so each test creates its own booking and uses the returned ID (stored as a variable).
- **No secrets in the repository.** Base URL and credentials are environment variables; the exported collection contains only `{{baseUrl}}`, `{{token}}`, `{{username}}` and `{{password}}`.

## Repository structure

```
api-testing-portfolio/
├── postman/
│   └── Restful-booker.postman_collection.json   # Postman collection (v2.1), incl. "Negative tests" folder
├── Findings.md                                   # Test results and defect list
└── README.md
```

## How to run the Postman collection

1. Import `postman/Restful-booker.postman_collection.json` into Postman.
2. Create an environment with these variables:

   | Variable | Value |
   |---|---|
   | `baseUrl` | `https://restful-booker.herokuapp.com` |
   | `username` | `admin` |
   | `password` | `password123` (public demo credentials from the API docs) |
   | `token` | value returned by **Create token** |
   | `bookingID` | value returned by **Create booking** |

3. Select the environment, run **Create token**, then **Create booking**, and continue with the other requests.

## Roadmap

- [x] Manual API testing in Postman (CRUD, authentication, negative tests)
- [ ] Postman test scripts and automatic variable handling; run the collection with Newman
- [ ] Python + pytest + requests test suite (API client layer, fixtures, schema validation)
- [ ] CI with GitHub Actions and HTML test reports
- [ ] API tests in Robot Framework (RequestsLibrary)

## Author

**Dušan Đorđević** – Senior QA Automation Engineer, ISTQB Advanced Level Test Automation Engineer
[LinkedIn](https://www.linkedin.com/in/du%C5%A1an-djordjevic-39b96720b/)
