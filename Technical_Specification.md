# Spinlog: Technical Specification Document 

# Team Members: Owen Mueller and Marcus Rowe 

## **1. Version and change log** 

|**Version**|**Date**|**Change**|**Author**|
|---|---|---|---|
|0.1|7 November 2026|First draft, written against Requirements Document v0.1 and fixes differences between D0, D1, and D2 requirements|Marcus Rowe<br>Owen Mueller|



## **2. Architecture summary** 

Spinlog has seven components (D0 diagram: D0_Block_Diagram.png). Two are clients **:** the React Native phone app (C1) and the ESP32 NFC reader (C2). Four are server modules: Plays (C3), Valuation (C4), Collection (C5) and Authentication (C6). The seventh is the PostgreSQL database (C7). Three external systems are involved: the NFC tag (E1), the identity provider (E2) and Discogs (E3). 

**Deployment model:** One team computer runs two containers: the FastAPI backend under Uvicorn, and PostgreSQL. The phone and the reader reach the backend over the home or lab network. The nightly price job (C4) runs inside the backend process on a timer. Clients never talk to the database, and external services are called only from the server, except the one-time sign-in redirect (I4). 

**Data flow: Add a record** _-_ C1 scans a barcode, C5 asks Discogs (I14), the collector picks a match, C5 saves the record to C7. **Value -** every night C4 reads owned releases from C7, fetches the lowest price from Discogs (I15) and stores one snapshot per release per day. **Play -** the reader sends the tag UID to C3 (I3); C3 authenticates the reader, checks for duplicates, writes the play to C7 and returns a status that the reader shows as a light color. Every phone request carries a session token from C6. Flow diagrams: A, B, C. 

## **3. Interfaces and data formats** 

### **3.1 - Component boundaries** 

### **3.1.1 – Boundary table** 

Full table is also in D0 section 4, error behavior is further detailed in D2 section 3. JSON schemas are requests and responses with the inputs and outputs specified in this table (example shown below in 3.1.3), database schemas are specified below in 3.1.2. 

|**ID**|**Boundary**|**Inputs**|**Outputs**|**Format/Protocol**|**Error Behavior**|
|---|---|---|---|---|---|
|**I1**|User → C1|Touch, camera frames|Screens, field<br>errors|OS touch and<br>camera APIs|Invalid field highlighted. Camera denied or<br>unreadable barcode: retry or manual<br>entry.|
|**I2**|E1 → C2|NFC read command|7-byte tag_uid (14<br>hex)|NFC-A (ISO 14443-A)|Read fails or times out: red light, nothing<br>sent (EC-12). Non-7-byte UID: red (EC-13).|
|**I3**|C2 → C3|pairing_token,<br>tag_uid, played_at,<br>event_id|status, play_id,<br>record_id|JSON, HTTP POST|400: drop and red. 401: red, stop sending.<br>500/503 or no network: keep in flash, retry<br>same event_id, 5 s doubling to 5 min (EC-<br>14).|
|**I4**|C1 → E2|client_id, redirect_uri,<br>state, code_challenge|code, state|OAuth 2.0 code +<br>PKCE in system<br>browser|Cancel or state mismatch: restart sign-in.<br>Provider down: "try again later".|
|**I5**|C1 → C2|server_url, ssid,<br>password,<br>pairing_token, commit|device_id, status|JSON over BLE|Non-connected status shown to user. BLE<br>drop: reader discards partial settings (EC-<br>16).|
|**I6**|C1 → C6|Code and<br>code_verifier, or<br>refresh_token|session_token,<br>refresh_token|JSON, HTTPS REST|400 missing field. 401 bad or reused token.<br>503 provider down.|
|**I7**|C1 → C5|session_token, filters,<br>record fields,<br>barcode, date range|Records,<br>candidates, value<br>history|JSON, HTTPS REST|400 names the field. 401 sign in again. 403<br>refuse and log. 404. 409 limit_reached or<br>conflict. 503 after one retry.|
|**I8**|C1 → C3|session_token,<br>record_id, optional<br>played_at|play_id|JSON, HTTPS POST|400, 401, 403 (refuse and log), 404, 503.|
|**I9**|C6 → E2|code, code_verifier,<br>client_id,<br>client_secret|id_token,<br>refresh_token|Form-encoded<br>request, JSON reply,<br>HTTPS|Invalid code: 401. Bad id_token: reject and<br>log. No answer in 5 s: 503, no retry (EC-<br>45).|
|**I10–**<br>**I13**|C6, C5, C4,<br>C3 → C7|SQL scoped by<br>user_id|Result rows|SQL over<br>PostgreSQL protocol|Connection lost: retry once, then 503<br>(reader and nightly job keep their work).<br>Constraint violation: 409. Duplicate play:<br>return stored play as duplicate.|
|**I14**|C5 → E3|barcode (8 to 14<br>digits)|Candidate<br>releases|JSON, HTTPS GET|Rate limited or down: 503 lookup_busy. No<br>match: empty list.|
|**I15**|C4 → E3|release_id|lowest_price,<br>num_for_sale|JSON, HTTPS GET|429: slow down; skip release after 3 slow-<br>downs. Down: stop run after 5 failures in a<br>row. No price: save nothing (EC-24, 29,<br>30).|
|**I16**|C2 → C6|X-Pairing-Token<br>header|device_id,<br>server_time|JSON, HTTP GET|401 revoked or unknown: red, stop<br>sending.|



### **3.1.2 – Database schemas** 

Refer to Design_Diagrams/images/D1_Data_Model.png (D1 Figure 1). 

|**Table**|**Columns**|**Keys and Rules**|
|---|---|---|
|**USERS**|user_id int, idp_subject text, email text, created_at<br>timestamptz|PK user_id; UK idp_subject|
|**REFRESH_TOKENS**|token_id int, user_id int, token_hash text, expires_at,<br>revoked_at|PK; FK user; UK token_hash; hash only; 30-day<br>expiry|
|**DEVICES**|device_id text, user_id int, pairing_token_hash text,<br>last_seen_at, revoked_at|PK; FK user; UK hash; max 5 active per user|
|**RELEASES**|release_id int, discogs_release_id int, created_by_user_id<br>int, artist, title, label, year int, genres text[],cover_url|PK; UK discogs_release_id; no Discogs ID<br>means hand-entered and private|
|**RECORDS**|record_id int, user_id, release_id, added_at, removed_at,<br>condition, notes|PK; FKs; soft remove via removed_at; max<br>1,000 active per user|
|**TAGS**|user_id int, tag_uid char(14), record_id int|PK (user_id, tag_uid); UK record_id|
|**PLAYS**|play_id int, user_id, record_id, device_id (nullable), event_id<br>text, played_at timestamptz|PK; unique (device_id, event_id)|
|**PRICE_SNAPSHOTS**|release_id int, snapshot_date date, lowest_price<br>numeric(10,2), num_for_sale int|PK (release_id, snapshot_date)|



### **3.1.3 – JSON payload example** 

POST /taps (I3):

Request:

```json
{"pairing_token":"dk_9f2c7a41b6e8","tag_uid":"04A35B12C87780","played_at":"2026-10-27T22:18:04Z","event_id":"reader-01-000412"}
```

Reply (status is one of logged, duplicate, unassigned, linked):

```json
{"status":"logged","play_id":1873,"record_id":212}
```

### **3.2 – Endpoint table** 

Authentication: **S** = Session token, **P** = pairing token, **none** = public 

|**#**|**Path**|**Owner**|**Auth**|**Success**|**Errors**|**Req**|
|---|---|---|---|---|---|---|
|**1**|POST /taps|C3|P (body)|200 {status, play_id, record_id}|400 invalid_field, 401 invalid_token, 503 database_unavailable, 500 server_error|REQ-05,<br>06, 07|
|**2**|POST /plays *(proposed, OQ-04)*|C3|S|201 {play_id, record_id,<br>played_at}|400, 401, 403, 404, 503|REQ-10|
|**3**|GET /releases/lookup?barcode= *(proposed, Q1)*|C5|S|200 {items:[{release_id, artist, title, year}]}, empty list if none|400, 401, 503 lookup_busy|REQ-01|
|**4**|POST /records|C5|S|201 saved record (body<br>has discogs_release_id)|400, 401, 409 limit_reached|REQ-01,<br>NFR-02|
|**5**|POST /records/manual|C5|S|201 saved record (artist,<br>title required)|400, 401, 409 limit_reached|REQ-01,<br>NFR-02|
|**6**|GET /records?artist=&genre=<br>&decade=&q=&limit=&offset=|C5|S|200 {items, total}, items<br>carry play_count,<br>last_played|400, 401, 403|REQ-02, 09|
|**7**|PATCH /records/{id}|C5|S|200 updated record<br>(condition, notes only)|400, 401, 403, 404|REQ-03|
|**8**|DELETE /records/{id}|C5|S|204(soft remove)|401,403,404|REQ-03|
|**9**|GET /value-<br>history?from=&to=|C5|S|200 [{date, total_usd,<br>priced_count,<br>owned_count}]|400 bad dates, 401, 503|REQ-04|
|**10**|POST /link-sessions|C5|S|201{record_id, expires_at}|401,404,409|REQ-07|
|**11**|GET /link-sessions/current|C5|S|200 session|401,404|REQ-07|
|**12**|DELETE /link-sessions/current|C5|S|204|401|REQ-07|
|**13**|DELETE /tags/{tag_uid}|C5|S|204|401,404|REQ-07|
|**14**|POST /auth/token|C6|none|200 {session_token,<br>expires_in, refresh_token}|400, 401, 503|REQ-11|
|**15**|POST /auth/refresh|C6|none|200 new tokens; old<br>refresh token stops<br>working|401 invalid_refresh|REQ-11,<br>NFR-05|
|**16**|POST /auth/logout|C6|none|204|none (always 204)|REQ-11|
|**17**|POST /devices|C6|S|201 {device_id,<br>pairing_token} (shown<br>once)|401, 409 device_limit (5)|REQ-08|
|**18**|GET /devices|C6|S|200 {items:[{device_id,<br>last_seen_at, revoked_at}]}|401|REQ-08|
|**19**|DELETE /devices/{device_id}|C6|S|204|401,403,404|REQ-08,12|
|**20**|GET /devices/me|C6|P<br>(header)|200 {device_id,<br>server_time}|401|REQ-08, 06|



## **4. Technology stack** 

|**Layer**|**Choice**|**Justification**|
|---|---|---|
|**Phone app**|React Native (MIT); scanner and BLE<br>libraries TBD|<br>Camera barcode (REQ-01); lists and chart (REQ-02, REQ-04, REQ-<br>09); manual play (REQ-10); BLE setup (REQ-08); free (CON-04)|
|**Reader firmware**|ESP32, C++|Wi-Fi and BLE built in (REQ-08); 7-byte UIDs (REQ-05, REQ-07); 2 s<br>tap path (NFR-01); about $10 (NFR-08,CON-02)|
|**Reader storage**|ESP32 flash queue|Offline taps (REQ-06); survives power cut (EC-11)|
|**Backend**|Python, FastAPI, Uvicorn|One process for plays, records, auth (REQ-05, REQ-07, REQ-11,<br>REQ-12); error codes (NFR-06); free (CON-04)|
|**Database**|PostgreSQL|Exactly-once plays (REQ-05); value history joins (REQ-04); soft remove (REQ-03); filters and counts (REQ-02, REQ-09); 1,000 records × 20 users (NFR-02, CON-01); free (CON-04)|
|**DB access**|SQLAlchemy, psycopg 3|user_id on every query (REQ-12); TLS-capable (NFR-03,CON-03)|
|**Authentication**|OAuth 2.0 + PKCE; provider TBD (OQ-<br>01)|Sign-in and tokens (REQ-11); reader pairing tokens (REQ-08); no<br>passwords (NFR-05)|
|**Token hashing**|hashlib, secrets|Hash-only tokens (NFR-05, REQ-08, REQ-11); no extra dependency<br>(CON-04)|
|**Scheduler**|In-process timer|Nightly snapshots (REQ-04); one per release per day (NFR-04);<br>catch-up (EC-27)|
|**External data**|Discogs API, personal token|Barcode lookup (REQ-01); prices (REQ-04); 60 requests per minute<br>(NFR-04)|
|**Hosting**|Docker on a team computer|Free (CON-04); under 20 users (NFR-02); nightly pg_dump (NFR-07)|
|**Transport security**|TLS 1.2+ in production; demo on plain<br>HTTP|NFR-03, CON-03 (in production)|
|**Logging**|Plain-text console log|401,403,retries, unassigned taps (NFR-06); refusals (REQ-12)|



## **5. Known Constraints** 

From Requirements Document v0.1 section 7. 

|**ID**|**Constraint**|**Rules out**|**Forces**|
|---|---|---|---|
|**CON-01**|Each user can track at<br>least 1,000 records.|Loading the whole collection into<br>the phone at once|Paged lists (100 per page), scrolling list on phone|
|**CON-02**|Reader and tags cost<br>less than $20.|Raspberry Pi class boards, extra<br>parts such as a clock chip, custom<br>hardware.|A bare ESP32 plus one NFC module and NTAG tags|
|**CON-03**|All user data is encrypted in transit with TLS 1.2 or higher.|Plain HTTP and unencrypted database connections in a deployed system.|TLS on I3, I6, I7, I8, I16 and on I10–I13 (in production)|
|**CON-04**|Development is paid for from the team's personal funds.|Paid hosting, paid identity providers, paid data sources, paid app-store accounts.|Self-hosting on a team computer (it must stay on while the app is used), free-tier identity provider, free Discogs API, open-source libraries only, no app-store release.|



## **6. Traceability matrix** 

|**Req ID**|**Description**<br>|**Design Component (D0/D1/D2)**|**Status**|**Test Case**|
|---|---|---|---|---|
|**REQ-01**|Add a record from a Discogs barcode<br>match or by hand|D0 Section 5; D2 Section 2.4; Spec 3.2|Designed|TC-01|
|**REQ-02**|List, filter by artist/genre/decade, keyword<br>search, total count|D2 Section 2.4; Spec 3.2|Designed|TC-02|
|**REQ-03**|Edit condition and notes; soft remove<br>keeps past totals|D1 Section 2; D2 Section 2.4|Designed|TC-03|
|**REQ-04**|Daily collection value from newest<br>snapshot on or before each day|D0 Section 5; D1 Section 2; D2<br>Sections 2.3,2.4|Designed|TC-04|
|**REQ-05**|Exactly one play per tap; retries and 10 s<br>double taps not counted|D1 Sections 2, 3, 5; D2 Section 2.2|Designed|TC-05|
|**REQ-06**|Reader stores 100 unsent taps in flash and<br>resends with same IDs|D2 Sections 2.2, 3|Designed|TC-06|
|**REQ-07**|Link a tag only in a 2-minute link session;<br>link tap is not a play|D1 Section 2; D2 Sections 2.2, 2.4|Designed|TC-07|
|**REQ-08**|Pair a reader over Bluetooth; reject its taps<br>after revoke|D1 Section 2; D2 Sections 2.2, 2.5|Designed|TC-08|
|**REQ-09**|Show play count and last-played date per<br>record|D1 Section 2; D2 Section 2.4|Designed|TC-09|
|**REQ-10**|Log a play by hand from the record screen|D1 Section 2; D2 Section 2.1; Spec 3.2|Planned|TC-10|
|**REQ-11**|Sign in via identity provider; 15-min<br>session; 30-day rotating refresh; sign out|D1 Section 2; D2 Sections 2.1, 2.5|Designed (identity<br>provider not yet<br>chosen)|TC-11|
|**REQ-12**|Show and change only the signed-in<br>collector's data; refuse and log others|D1 Section 2; D2 Section 3; Spec 3.2|Designed|TC-12|
|**NFR-01**|Reader light within 2 s of a tap|D0 Section 6; D1 Section 3|Designed|TC-13|
|**NFR-02**|1,000 records per user, fewer than 20 users<br>on one computer|D1 Section 2; D2 Section 3|Designed|TC-14|
|**NFR-03**|TLS 1.2+ on all connections|D0 Section 6; D2 Section 2.6|Planned (deferred<br>for demo)|TC-15|
|**NFR-04**|Stay within Discogs limits and terms|D2 Sections 2.3,2.4|Designed|TC-16|
|**NFR-05**|Only hashes of refresh and pairing tokens<br>stored|D1 Section 2; D2 Section 2.5|Designed|TC-17|
|**NFR-06**|Failures visible to user and team; codes<br>and logs|D2 Section 2.6; Spec 3.2|Designed|TC-18|
|**NFR-07**|Later team can run, recover, extend from<br>written steps|D2 Sections 2.6, 3|Planned (setup<br>steps not yet<br>written)|TC-19|
|**NFR-08**|Reader plus tags under $20|D0 Section 6|Planned (parts list<br>not yet written)|TC-20|
|**CON-01**|At least 1,000 records per user|D2 Section 3|Designed|TC-21|
|**CON-02**|Reader and tags under $20|D2 Section 2.2|Planned (same<br>cost sheet as<br>NFR-08)|TC-22|
|**CON-03**|TLS 1.2+ for all user data in transit|D0 Section 6; D2 Section 2.6|Planned<br>(deferred)|TC-23|
|**CON-04**|Paid from personal funds|D0 Section 6; D1 Sections 4,6|Designed|TC-24|



