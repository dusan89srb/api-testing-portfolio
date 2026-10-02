# Findings – Restful-booker API

Manual exploratory testing with Postman. Base URL: https://restful-booker.herokuapp.com

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
| DELETE with invalid/expired token | 403 Forbidden | 401 Unauthorized | ❌ | An invalid token means the user is not authenticated (401); 403 is for authenticated users without permission |