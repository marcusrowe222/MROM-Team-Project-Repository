# Tasklist.md

T1. Investigate the NFC-A read path for C2 NFC Reader Device (I2, E1), confirming 7-byte UID support and NTAG213/215 compatibility on the chosen ESP22 module before firmware work starts. Marcus Rowe

T2. Develop the tap queue and sender in C2 NFC Reader Device (I3), keeping taps in flash, retrying with the same event_id, and dropping the oldest tap once 100 are queued. Marcus Rowe

T3. Develop the Bluetooth setup flow in C2 NFC Reader Device (I5) and the reader setup wizard in C1 Client App Frontend (S6, S7), covering setup mode, Wi-Fi join, paring token, and factory reset. Marcus Rowe

T4. Develop the POST /api/v1/taps endpoint in C3 Plays (I3, I13) using the D1 Play Tracking Algorithm: input validation, device lookup, duplicate check, unassigned taps, and the 10 second double tap rule. Marcus Rowe

T5. Develop sign in, token refresh, and sign out in C6 Authentication (I6, I9, I10), including refresh-token rotation and reuse detection, plus the sign in screen in C1 Client App Frontend (S1, I4). Marcus Rowe

T6. Test the tap path through C2 NFC Reader Device and C3 Plays against the 2 second tap to LED target and edge cases EC-09 to EC-14 and EC-17 and EC-22. Marcus Rowe

T7. Refine the C7 Database model (Figure 1) by adding the field length limits, numeric(10, 2) price type, not null rules, and unique constraints that EC-48 to EC-52 depend on. Owen Mueller

T8. Develop the record service in C5 Collection (I7, I11, I14): list, filter, search, add by barcode or by hand, edit, and soft remove, enforcing the 1000 record limit. Owen Mueller

T9. Develop the value history and link tag sessions in C5 Collection (I7, I11), computing per day totals from price snapshots and expiring link sessions after two minutes. Owen Mueller

T10. Develop the nightly job in C4 Valuation (I12, I15) that fetches Discogs prices, saves one snapshot per release per day, and handles rate limits and Discogs outages. Owen Mueller

T11. Develop the library, record detail, value chart, and add record screens in C1 Client App Frontend. Owen Mueller

T12. Test C4 Valuation and C5 Collection against edge cases EC-24 to EC-38 for boundary values, timing, bad input, and external failures. Owen Mueller
