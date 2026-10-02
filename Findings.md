| Request | Status | Expected| Correct? | Comment |
|---|---|---|---|---|
| GET /ping | 201 Created | 200 OK | X | GET is not creating a resource; health-check tools expect 200 and would report this server unavailable|

|GET /booking|200 OK |200 OK|✅| {"bookingid": 3}
| GET /booking/3 (nepostojeći ID) | 404 Not Found | 404 Not Found | ✅ | Ispravno: nepostojeći resurs vraća 404 |
| GET /booking/3 | 200 OK (ranije 404) | 200 OK | ✅ | Podaci na deljenom serveru se menjaju/resetuju; testovi ne smeju koristiti fiksne ID-jeve |
|GET /booking|200 OK |200 OK|✅| 222544dce5c9aa3 token added
|GET /booking|200 OK |200 OK|✅| bookingid: 3869
| DELETE /booking/{id} | 201 Created | 204 No Content (ili 200 OK) | ❌ | 201 znači "kreirano", a resurs je obrisan |
| GET /booking/{obrisani id} | 404 Not Found | 404 Not Found | ✅ | Potvrda da DELETE stvarno briše, uprkos pogrešnom 201 statusu |
