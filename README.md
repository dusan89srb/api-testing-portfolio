# API Testing Portfolio – Restful-booker

Exploratory, negative and automated testing of the [Restful-booker](https://restful-booker.herokuapp.com/apidoc/index.html) REST API, a public practice API for testers. The goal of this project is to show how I approach API testing: understanding the contract, checking both the response and the actual effect on the data, documenting defects clearly and turning manual checks into automated tests.

**Current status:** manual exploratory testing (done) → automated Postman test suite with 52 checks (done) → Newman CLI runs and Python + pytest suite (in progress).

## Key results

- **25 manual checks**, **13 defects found**, 3 of them high severity → [Findings.md](Findings.md)
- **52 automated checks** in Postman, running in about 6 seconds: 41 pass, **11 fail – each failure is a documented defect**, with 0 script errors

| Severity | Finding |
|---|---|
| High | Wrong data type (`"totalprice": "Sto"`) is silently converted to `null` and saved (data corruption) |
| High | Missing required field on `POST /booking` crashes the server (500) instead of returning 400, while `PUT` handles the same case correctly |
| High | Business rule not validated: a booking with checkout before checkin is accepted and saved |
| Medium | Authentication failures return 403 instead of 401 (no credentials, invalid or expired token) |
| Medium | 404 and 405 are swapped (non-existent ID → 405, missing ID → 404) |
| Low | Wrong success codes (`GET /ping` and `DELETE` return 201, `POST` returns 200) and `418 I'm a teapot` for unsupported formats |

## Automated Postman tests

The collection is organised by purpose and runs top to bottom:

| Folder | What it does |
|---|---|
| `1. Happy path` | Health check, token, create, read, full update (token and Basic Auth), partial update, list |
| `2. Negative tests` | Missing / invalid credentials, missing fields, wrong data types, business rules, malformed JSON, invalid IDs, unsupported methods and formats |
| `3. Cleanup` | Deletes the booking created in the run and verifies it is really gone |

**Techniques used:**

- **Request chaining** – the auth call stores the token and the create call stores the new booking ID, so the whole flow runs without manual steps or hardcoded IDs
- **Dynamic test data** – unique names, prices and future dates are generated for every run, and the response is compared with exactly what was sent
- **JSON schema validation** – the booking response is validated against a schema, which catches structural defects such as the `null` price
- **Effect verification** – after `DELETE`, a follow-up request confirms the booking returns 404; after `PATCH`, unchanged fields are checked too
- **Spec-based assertions** – tests assert the correct HTTP behaviour, so known defects stay visible as failing tests instead of being hidden
- **No secrets in the repository** – base URL and credentials are environment variables

## Testing approach

- **Verify the effect, not only the status code.** A `DELETE` returning 201 is a status bug, but the booking is really deleted; a `POST` returning 200 with `null` price is a data bug. These are reported separately.
- **Equivalence partitioning and boundary values** for input fields (valid number, zero, negative, wrong type).
- **Change one thing per negative test.** Everything else in the request stays valid, so the failure is caused by the input under test and not, for example, by an expired token.

## Repository structure

```
api-testing-portfolio/
├── postman/
│   └── Restful-booker.postman_collection.json   # Postman collection (v2.1) with test scripts
├── Findings.md                                   # Manual test results and defect list
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

   `token`, `bookingID` and `bookingFirstname` are set automatically by the test scripts.

3. Select the environment and run the collection with the **Collection Runner** (Run → Start run).
4. Expected result: 52 tests, 11 failing. Each failing test corresponds to a defect in [Findings.md](Findings.md).

## Roadmap

- [x] Manual API testing in Postman (CRUD, authentication, negative tests)
- [x] Postman test scripts, request chaining, dynamic data and schema validation
- [ ] Run the collection with Newman and generate HTML reports
- [ ] Python + pytest + requests test suite (API client layer, fixtures, schema validation)
- [ ] CI with GitHub Actions
- [ ] API tests in Robot Framework (RequestsLibrary)

## Author

**Dušan Đorđević** – Senior QA Automation Engineer, ISTQB Advanced Level Test Automation Engineer
[LinkedIn](https://www.linkedin.com/in/dusan-djordjevic-seniorqa/)
