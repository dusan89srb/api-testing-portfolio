# Findings – Restful-booker API

Manual exploratory testing with Postman. Base URL: https://restful-booker.herokuapp.com

## Summary

25 checks executed: 12 passed, 13 failed.

**Most important issues:**

- **High** – Wrong data type is silently converted to `null` and saved (`"totalprice": "Sto"`) – data corruption
- **High** – Missing required field on POST crashes the server (500 Internal Server Error) instead of returning 400
- **High** – Business rule not validated: a booking with checkout before checkin is accepted and saved
- **Medium** – Authentication errors return 403 instead of 401 (no credentials, invalid or expired token)
- **Medium** – 404 and 405 are swapped (non-existent ID returns 405; missing ID returns 404)
- **Low** – Wrong success codes (ping and DELETE return 201, POST returns 200) and a joke status (418) for unsupported formats

## Detailed results

| Request | Actual status | Expected status | Correct? | Comment |
|---|---|---|---|---|
| GET /ping | 201 Created | 200 OK | ❌ | GET does not create a resource; health-check tools expecting 200 would report the server as unavailable |
| GET /booking | 200 OK | 200 OK | ✅ | Returns a list of booking IDs |
| GET /booking/3 (non-existent ID) | 404 Not Found | 404 Not Found | ✅ | Non-existent resource correctly returns 404 |
| GET /booking/3 (later) | 200 OK (previously 404) | 200 OK | ✅ | Data on the shared server changes/resets; tests must not rely on hardcoded IDs |
| POST /auth | 200 OK | 200 OK | ✅ | Returns an auth token |
| POST /booking | 200 OK | 201 Created | ❌ | A new resource was created, so 201 Created is expected |
| DELETE /booking/{id} | 201 Created | 204 No Content (or 200 OK) | ❌ | 201 means "created", but the resource was deleted |
| GET /booking/{deleted id} | 404 Not Found | 404 Not Found | ✅ | Confirms DELETE actually removes the booking, despite the wrong 201 status |
| DELETE /booking/{id} with invalid/expired token | 403 Forbidden | 401 Unauthorized | ❌ | An invalid token means the user is not authenticated (401); 403 is for authenticated users without permission |
| DELETE /booking/{non-existent id} (valid token) | 405 Method Not Allowed | 404 Not Found | ❌ | DELETE is supported on this endpoint; a missing resource should return 404, not 405 |
| PUT /booking/{non-existent id} | 405 Method Not Allowed | 404 Not Found | ❌ | Same issue as DELETE: missing resource returns 405 instead of 404 |
| PUT /booking/{id} (valid token, full body) | 200 OK | 200 OK | ✅ | Booking fully updated; verified with GET |
| PATCH /booking/{id} (valid token, partial body) | 200 OK | 200 OK | ✅ | Only the sent fields were updated, other fields unchanged; verified with GET |
| PUT /booking/{id} with Basic Auth (full body) | 200 OK | 200 OK | ✅ | Basic Auth works as an alternative to the token; credentials stored as environment variables; verified with GET |
| PUT /booking/{id} without any authentication | 403 Forbidden | 401 Unauthorized | ❌ | No credentials sent at all: the user is unauthenticated, so 401 is expected; 403 implies a known user without permission |
| PUT /booking/{id} with invalid token | 403 Forbidden | 401 Unauthorized | ❌ | Invalid credentials: the user is unauthenticated, so 401 is expected |
| PUT /booking/{id} (valid token, missing required field) | 400 Bad Request | 400 Bad Request | ✅ | Correctly rejected; booking unchanged; verified with GET |
| POST /booking with missing required field | 500 Internal Server Error | 400 Bad Request | ❌ | **High severity**: unhandled server error instead of validation; reproducible with a fresh token. Inconsistent with PUT, which correctly returns 400 for the same case |
| POST /booking with wrong type ("totalprice": "Sto") | 200 OK, booking created with totalprice: null | 400 Bad Request, no booking created | ❌ | **High severity**: invalid input is silently converted to null and persisted (data corruption); verified with GET |
| POST /booking with checkout before checkin | 200 OK, booking created | 400 Bad Request or 422 Unprocessable Entity | ❌ | **High severity**: business rule not validated; a booking with a negative duration is persisted |
| POST /booking with malformed JSON (missing comma) | 400 Bad Request | 400 Bad Request | ✅ | Correctly rejected; improvement: error message does not say what or where the problem is |
| GET /booking/{non-numeric id} ("asdadads") | 404 Not Found | 400 or 404 | ✅ | Acceptable; 400 would be more precise since the ID format itself is invalid |
| DELETE /booking (no ID) | 404 Not Found | 405 Method Not Allowed (with Allow header) | ❌ | Inverse of the DELETE non-existent ID finding: 404 and 405 are swapped |
| POST /booking without explicit Accept header (Postman sends */*) | 200 OK, JSON returned | 2xx | ✅ | Content negotiation works with */* (200 vs 201 already reported above) |
| POST /booking with unsupported Accept (application/pdf) | 418 I'm a teapot | 406 Not Acceptable | ❌ | Joke status from RFC 2324 (April Fools'); a real client cannot handle it; 406 with supported formats is expected |
