# User Stories, Use Cases, and Acceptance Criteria 
**Team members:** Owen Mueller, Marcus Rowe  



## 1. Stakeholder Map

| Category | Stakeholders |
|----------|--------------|
| **Primary** | Vinyl collectors (core users) <br> Project team (Owen and Marcus) |
| **Secondary** | Record stores and sellers <br> Data providers / APIs <br> NFC tag suppliers <br> App stores |
| **Hidden** | Later student teams who may inherit this project <br> Collectors who cannot or will not adopt the app |

---

## 2. User Stories

| ID | Stakeholder Category | Story |
|----|----------------------|-------|
| **US-01** | Primary: Vinyl Collector | **As a** vinyl collector with a growing collection, <br>**I want to** keep a digital library of my records, <br>**so that** I can see which records I own by artist, genre, and time period. |
| **US-02** | Primary: Vinyl Collector | **As a** vinyl collector, <br>**I want to** see what my collection is worth over time, <br>**so that** I can decide which records to keep or sell. |
| **US-03** | Primary: Vinyl Collector | **As a** vinyl collector who plays my records often, <br>**I want to** log a listen without opening the app or typing anything, <br>**so that** I can see how often I play each album. |
| **US-04** | Secondary: Data Provider | **As a** provider of record pricing or song data, <br>**I want** the app to stay within my request limits and use my data under my terms, <br>**so that** my service stays available and this project can keep using it. |
| **US-05** | Hidden: Later Student Team | **As a** student on a later team who inherits this project, <br>**I want** the code, data model, and setup steps to be clearly documented and runnable, <br>**so that** I can add to the app within days instead of reverse engineering it. |

**Coverage:** three primary stories (US-01 to US-03), one secondary (US-04), and one hidden (US-05).

---

## 3. INVEST Self-Check

Each story was checked against **I**ndependent, **N**egotiable, **V**aluable, **E**stimable, **S**mall, and **T**estable.

| Story | I | N | V | E | S | T | Notes |
|-------|:-:|:-:|:-:|:-:|:-:|:-:|-------|
| **US-01** | YES | YES | YES | YES | YES | YES | Foundation story that stands alone and states the need, not a screen. Verified by AC-01.1 through AC-01.4. |
| **US-02** | YES | YES | YES | YES | YES | YES | Needs records (US-01) and a price source (US-04), but can be built and tested with sample data. Testable by checking that known records and prices give the expected total. |
| **US-03** | YES | YES | YES | YES | YES | YES | Reworded to remove the "tapping" interaction, so NFC is one option rather than a requirement. Testable by logging a known number of listens and checking the count. |
| **US-04** | YES | YES | YES | YES | YES | YES | Enforcement method (caching, throttling) is left open. Testable by comparing request counts to each provider's published limit. |
| **US-05** | YES | YES | YES | YES | YES | YES | Scope is limited to a README, data model description, and setup steps. Testable by having a new developer run the app from the written steps alone. |

---

## 4. Use Cases

### UC-01: Filter Collection

| Field | Detail |
|-------|--------|
| **ID** | UC-01 |
| **Expands** | US-01 |
| **Name** | Filter collection by artist, genre, and decade |
| **Primary actor** | Vinyl collector (primary stakeholder) |
| **Secondary actors** | None |

**Preconditions** (each can be verified by a tester)

1. The collector is signed in.
2. The collection contains at least 1 record.
3. Every record has an artist, a genre, and a release year.
4. The collection view is reachable from the main page.

**Main success flow**

| Step | Actor action | System response |
|:----:|--------------|-----------------|
| 1 | The collector opens the collection view. | The system displays all records sorted by date added, along with a total record count. |
| 2 | The collector chooses to filter. | The system offers filter options for artist, genre, and decade. |
| 3 | The collector selects Genre = "Rock". | The system displays only records with the genre "Rock" and updates the count. |
| 4 | The collector selects Artist = "AC/DC". | The system narrows the list to records that match both Genre = "Rock" and Artist = "AC/DC", and updates the count. |

**Alternate flow: UC-01-A1 (keyword search instead of preset filters)**

At step 2, the collector enters "AC/DC" as a search term instead of choosing a filter option. The system lists every record whose artist, album title, or label contains "AC/DC". The use case then ends at the postcondition.

**Exception flow: UC-01-E1 (no records match the filter combination)**

At step 3 or 4, no record matches the combined filters. The system displays the message "No records match your filters," keeps all active filters visible, and offers a way to clear them. When the collector clears the filters, the system removes all of them and shows the full collection, as in step 1.

**Postcondition**

The collector sees a list containing only the records that match every active filter. No collection data has been changed.

---

## 5. Acceptance Criteria (Given / When / Then)

### UC-01: Filter Collection

**AC-01.1: Main success flow (filter by genre and artist)**

- **Given** a signed-in collector with 500 records, of which 30 are Rock and 10 of those Rock records are by AC/DC,
- **When** the collector selects Genre = "Rock" and Artist = "AC/DC",
- **Then** exactly 10 records are shown, each with Genre "Rock" and Artist "AC/DC", and the count shows 10.

**AC-01.2: Exception flow (no matching records)**

- **Given** a signed-in collector with 500 records, none of which are both Rock and by Miles Davis,
- **When** the collector selects Genre = "Rock" and Artist = "Miles Davis",
- **Then** 0 records are shown, the message "No records match your filters" is displayed, and both active filters remain visible.

**AC-01.3: Exception flow recovery (clear filters)**

- **Given** the no-match state from AC-01.2,
- **When** the collector clears the filters,
- **Then** all 500 records are shown, the count shows 500, and no filters remain active.

**AC-01.4: Alternate flow (keyword search)**

- **Given** a signed-in collector with 500 records, of which 10 have "AC/DC" in the artist, album title, or label,
- **When** the collector searches for "AC/DC",
- **Then** exactly 10 records are shown, and each contains "AC/DC" in its artist, album title, or label.
