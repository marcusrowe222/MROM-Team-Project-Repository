# Spinlog: Requirements Document 

# Team Members: Owen Mueller and Marcus Rowe 

## **1. Version and change log:** 

|**Version**|**Date**|**Change**|**Author**|
|---|---|---|---|
|0.1|7 Nov 2026|First draft, consolidated from the user stories, Do, D1, D2, and the Constraints Essay.|Owen Mueller<br>Marcus Rowe|



## **2. Purpose and scope** 

Spinlog helps vinyl collectors catalog their records, see what their collection is worth over time, and log every play. This document states what the first release must do and for who. The Technical Specification Document says how it will be built. 

In scope for this release: 

- A phone app for the collection 

   - Adding records by hand. 

   - Filtering and searching. 

   - Editing, removing. 

   - Value over time. 

- Play logging by tapping an NFC tag on a dedicated reader, including: 

   - Tag linking 

   - Reader setup 

   - Offline taps 

- Pricing from the Discogs API, taken once a night 

Out of scope for this release: 

- Real time play tracking that identifies songs from audio 

- Sharing collections between users 

- Selling records 

- Publishing app to an app store 

## **3. Stakeholders and user goals** 

|**Stakeholder**|**Category**|**User goal**|**Source**|
|---|---|---|---|
|**Vinyl Collector**|Primary|Keep a digital library and filter it by artist, genre, and period.|US-01|
|**Vinyl Collector**|Primary|See what the collection is worth over time, to decide what to keep<br>or sell.|US-02|
|**Vinyl collector who plays often**|Primary|Log a listen without opening the app or typing.|US-03|
|**Data provider (Discogs)**|Secondary|Requests stay within limits, and data is used under the providers<br>term.|US-04|
|**Record stores and sellers, NFC tag**<br>**suppliers, app stores**|Secondary|Noted for context. No requirement in this release depends on<br>them directly|Week 4<br>map|
|**Later student team**|Hidden|Code, data model, and setup steps are documented and<br>runnable.|US-05|
|**Collectors who will not adopt the**<br>**app**|Hidden|Noted for context. Not designed for in this release.|Week 4<br>map|



## **4. Glossary** 

|**Term**|**Meaning**|
|---|---|
|**Collector**|A signed in user who owns records and tracks them in spinlog.|
|**Record**|One physical copy owned by a collector. It links to a release and to the<br>collector.|
|**Release**|One Discogs release, or a hand entered albym with no Discogs ID. Shared<br>across collectors.|
|**Tag**|An NFC tag, stuck to a record’s sleeve, that the reader can read.|
|**Tag link**|The connection between a tag and a record, set up in a link session.|
|**Link session**|A two minute window in the app in which the next tap on the reader links a tag.|
|**Play**|One logged listen to a record, from a tap or entered by hand.|
|**Duplicate play**|A second tap of the same record within 10 seconds, or a resend of the same<br>app, that is not counted again.|
|**Unassigned tap**|A tap on a tag that is not linked to any record. Nothing is saved.|
|**Reader**|The ESP32 device with an NFC module that reads tags and reports taps.|
|**Pairing token**|A secret the reader stores to prove which collector it belongs to.|
|**Session token**|A short lived (15 minute) token that the app sends with each request.|
|**Refresh token**|A long lived, 30 day token that gets a new session token. It rotates on each use.|
|**Identify provider**|The external service that checks who the collector is.|
|**Snapshot**|One saved lowest listed price for a release on one day.|
|**Collection Value**|The total of the newest snapshot prices for the records a collector owns, on a<br>given day.|
|**Soft remove**|Marking a record as removed, without deleting its row, so past totals stay the<br>same.|
|**Team computer**|The computer that runs the backend and database for this demo.|
|**Restricted Data**|Discogs data that the Discogs terms allow only for a limited use.|



## **5. Functional and Non-Functional Requirements** 

### **5.1 - Functional Requirements:** 

|**ID**|**Statement**|**MoSCoW**|**Source**|**Verification**|
|---|---|---|---|---|
|**REQ-01**|The system shall add a record from a barcode only after<br>the collector picks one of the Discogs matches for that<br>barcode, or by hand entry with artist and title.|Must|US-01|Test: scan a known barcode,<br>confirm matches are listed, pick<br>one, confirm one record is saved.<br>Repeat with a manual entry.|
|**REQ-02**|The system shall list the collector's records, filtered by<br>artist, genre, and decade. And by keyword across artist,<br>title, and label. Shall show the total count.|Must|US-01, UC-<br>01, AC-01.1<br>to AC-01.4|Acceptance tests AC-01.1 to AC-<br>01.4 with the stated sample data.|
|**REQ-03**|The system shall let the collector edit condition and<br>notes, and shall soft remove a record so that it leaves<br>the list while past value totals are unchanged.|Should|US-01, US-<br>02|Test: remove a record, confirm it<br>leaves the list, and confirm the<br>value total for an earlier date is<br>unchanged.|
|**REQ-04**|The system shall show the collection value for each day<br>in a selected date range, using each release's newest<br>snapshot dated on or before that day.|Must|US-02|Test: load known snapshots for<br>two records, confirm each day's<br>total matches the expected sum.|
|**REQ-05**|When a linked tag is tapped on the reader, the system<br>shall record exactly one play for that tag’s record. And<br>shall not record a second play for a retry of the same tap<br>or a second tap of the same record within 10 seconds.|Must|US-03|Test: tap a linked tag once, retry<br>the same tap, and tap again<br>within 10 seconds. Confirm one<br>play is stored.|
|**REQ-06**|While the reader cannot reach the server, the reader<br>shall store up to 100 unsent taps in flash and resend<br>them with their original event IDs. When the queue is<br>full, it shall drop the oldest tap.|Should|US-03|Test: disconnect the reader for 10<br>minutes, tap 20 times, reconnect,<br>and confirm 20 plays are stored<br>with no duplicates.|
|**REQ-07**|The system shall link a tag to a record only during a two-<br>minute link session that the collector opens in the app.<br>A tap that links a tag shall not be counted as a play|Must|US-03|Test: open a session, tap an<br>unlinked tag, confirm the link is<br>saved and no play is counted.<br>Wait out a session and confirm a<br>tap is refused.|
|**REQ-08**|The system shall pair a reader from the phone over<br>Bluetooth, shall send the Wi-Fi settings and a pairing<br>token to it, and shall reject its taps after the collector<br>revokes it.|Must|US-03, D0<br>decision log|Test: pair a reader, log a play,<br>revoke it, and confirm the next tap<br>is rejected.|
|**REQ-09**|The system shall show each record's play count and<br>last-played date.|Should|US-03|Test: log three plays for a record<br>and confirm the count is 3 and the<br>last-played date is the latest.|
|**REQ-10**|The system shall let the collector log a play by hand<br>from the record screen.|Should|US-03 (D0 I8)|Test: log a play by hand and<br>confirm it appears in the play<br>count.|
|**REQ-11**|The system shall sign the collector in through an identity<br>provider, keep the session for 15 minutes, renew it with<br>a refresh token that lasts 30 days and rotates on each<br>use, and sign the collector out on request.|Must|D0 I4, I9|Test: sign in, wait past 15 minutes<br>and confirm the refresh works,<br>sign out and confirm the refresh<br>token is refused.|
|**REQ-12**|The system shall show and change only the signed-in<br>collector's data. A request for another collector's data<br>shall be refused and logged.|Must|Security (D2<br>FR-12), US-<br>01|Test: sign in as two collectors,<br>request one collector's record<br>with the other's session, and<br>confirm a refusal and a log entry.|



### **5.2 - Non-Functional Requirements:** 

|ID|Statement|MoSCoW|Source|Verification|
|---|---|---|---|---|
|**NFR-01**|The system shall light the reader's result within<br>2 seconds of a tap.|Must|D0<br>(performance)|Measure 20 test taps with a<br>timestamp and confirm each is<br>within 2 seconds.|
|**NFR-02**|The system shall support up to 1,000 records<br>per user and fewer than 20 users on one team<br>computer.|Must|Constraints<br>Essay (Social),<br>D0 (scalability)|Load test with 20 accounts and one<br>account at 1,000 records; confirm<br>the 1,001st add is refused with 409<br>limit_reached.|
|**NFR-03**|The system shall encrypt data in transit with<br>TLS 1.2 or higher.|Should|Constraints<br>Essay (Security)|Check each HTTPS and database<br>connection with a TLS scan.|
|**NFR-04**|The system shall stay within Discogs request<br>limits and terms.|Must|US-04|Check the nightly job log against<br>Discogs' published limits, and<br>confirm one snapshot per release<br>per day.|
|**NFR-05**|A stolen database copy shall not expose<br>credentials. Only hashes of refresh and pairing<br>tokens shall be stored.|Must|D2 (security<br>constraint)|Inspect the token tables after a<br>sign-in and a pairing and confirm<br>only hashes are present.|
|**NFR-06**|Failures shall be visible to the user and to the<br>team. Light and app messages come from<br>endpoint error codes, and 401, 403, database<br>retries, and unassigned taps are logged.|Should|D2|Trigger each error in the Technical<br>Specification and confirm the light,<br>the app message, and the log entry.|
|**NFR-07**|A later team shall be able to run, recover, and<br>extend the project from the written steps and<br>the D0,D1,and D2 documents.|Should|US-05|Have a team member who did not<br>build the setup run the app from the<br>written steps alone.|
|**NFR-08**|The reader plus tag hardware shall cost less<br>than $20.|Should|Constraints<br>Essay<br>(Economic)|Price the parts for one reader and a<br>set of tags, and confirm the total is<br>under $20.|



## **6. MoSCoW Summary** 

#### **Must:** 

- REQ-01: Add a record by barcode or by hand. 

- REQ-02: Filter and search the collection, with a count. 

- REQ-04: Show collection value over time. 

- REQ-05: Log one paly per tap on a linked tag. 

- REQ-07: Link a tag to a record in a two minute link session. 

- REQ-08: Pair a reader from the phone, and reject its taps after revocation. 

- REQ-11: Sign in through an identity provider, with session and refresh tokens. 

- REQ-12: Show and change only the signed in collector’s data. 

- NFR-01: Light the reader’s result within 2 seconds. 

- NFR-02: Support up to 1,000 records per user and fewer than 20 users. 

- NFR-04: Stay within Discogs request limit and terms. 

- NFR-05: Keep no usable credentials in a copy of the database. 

#### **Should:** 

- REQ-03: Edit condition and notes, and soft remove a record. 

- REQ-06: Keep up to 100 unsent taps in the reader and resend them. 

- REQ-09: Show each record’s play count and last played date. 

- REQ-10: Log a play by hand from the record screen. 

- NFR-06: Make failures visible to the user and the team. 

- NFR-07: Let a later team run, recover, and extend the project. 

- NFR-08: Keep the reader and tag hardware under $20. 

- NFR-03: Encrypt data in transit with TLS 1.2 or higher. 

#### **Could:** 

- Password sign in, as an alternative to the identity provider. 

#### **Won’t (for this release):** 

- Real time play tracking that identifies songs from audio. 

- Audio recognition as a play logging method. 

- Logging plays by tapping the phone’s NFC reader. 

- Barcode or QR scans as a way to log plays. 

- Custom dedicated hardware for play tracking. 

- Microservices, since the backend is one deployable service. 

- Publishing the app to an app store. 

- Sharing collections or play history between collectors. 

- Selling records or showing listings for sale. 

- Push notifications, email, or text messages. 

## **7. Inherited Constraints:** 

|**ID**|**Constraint**|**Source**|
|---|---|---|
|**CON-01**|Each user should be able to keep track of at least 1,000 records.|Constraints Essay,<br>Social|
|**CON-02**|The reader and tags should cost less than $20.|Constraints Essay,<br>Economic|
|**CON-03**|All user data is encrypted in transit with TLS 1.2 or higher.|Constraints Essay,<br>Security|
|**CON-04**|Development is paid for from the team’s personal funds.|Constraints Essay,<br>Economic|



## **8. Assumptions:** 

- Collectors have a phone with a camera and Bluetooth, and a home Wi-Fi network. 

- Collectors own NFC tags and attach them to their records. 

- The team computer stays on and connection while the app is in use. 

- Discogs keeps providing release metadata, images, and lowest listed prices through its API. 

- The chosen identity provider supports the OAuth 2.0 authorization code flow with PKCE. 

## **9. Open Questions:** 

|**ID**|**Question**|**Owner**|**Proposed Date**|
|---|---|---|---|
|**OQ-01**|Which identity provider will sign collectors in?|Owen<br>Marcus|20 Nov 2026|
|**OQ-02**|Does the demo meet TLS, or is it deferred until production?|Owen<br>Marcus|20 Nov 2026|
|**OQ-03**|Does tag linking happen only at the reader, or also on the phone?|Owen<br>Marcus|20 Nov 2026|
|**OQ-04**|How is a manual play logged? The endpoint is not yet designed.|Owen<br>Marcus|27 Nov 2026|
|**OQ-05**|Which Bluetooth services and characteristics does the reader<br>expose for setup?|Owen<br>Marcus|27 Nov 2026|



