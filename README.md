## Overview

This project demonstrates an end-to-end data pipeline that automatically loads CSV sales data from a **Google Cloud Storage (GCS)** bucket into **Snowflake**, transforms it, and maintains a clean analytics-ready table using Snowflake-native features: **Snowpipe**, **Streams**, and **Tasks**.

## Architecture

```
┌─────────────────────────────────┐
│  GCS Bucket                     │
│  (daily-sales-south-1)          │
│  - Stores daily sales CSV files │
└───────────────┬─────────────────┘
                │  New file arrives
                ▼
┌─────────────────────────────────┐
│  GCP Pub/Sub                    │
│  - Sends a notification to      │
│    Snowflake saying "hey, a     │
│    new file just landed"        │
└───────────────┬─────────────────┘
                │  Event notification
                ▼
┌─────────────────────────────────┐
│  Snowpipe (AUTO_INGEST)         │
│  - Picks up the notification    │
│  - Reads the new CSV file       │
│  - Loads data into the raw      │
│    table automatically          │
│  - No manual COPY needed        │
└───────────────┬─────────────────┘
                │  Raw data loaded
                ▼
┌─────────────────────────────────┐
│  SALES_RAW (Landing Table)      │
│  - Stores data as plain text    │
│  - No transformation yet        │
│  - Acts as a staging area       │
└───────────────┬─────────────────┘
                │  Stream detects changes
                ▼
┌─────────────────────────────────┐
│  Stream (Change Data Capture)   │
│  - Watches SALES_RAW for any    │
│    new inserts, updates, or     │
│    deletes                      │
│  - Keeps a log of "what changed │
│    since last time we checked"  │
└───────────────┬─────────────────┘
                │  Task runs every 1 min
                ▼
┌─────────────────────────────────┐
│  Task (Scheduled MERGE)         │
│  - Runs automatically every     │
│    1 minute                     │
│  - Reads changes from stream    │
│  - Applies MERGE logic:         │
│    • New rows → INSERT          │
│    • Changed rows → UPDATE      │
│    • Deleted rows → DELETE      │
└───────────────┬─────────────────┘
                │  Clean data written
                ▼
┌─────────────────────────────────┐
│  SALES_CLEAN (Final Table)      │
│  - Proper data types (DATE,     │
│    NUMBER)                      │
│  - Always up-to-date            │
│  - Ready for analytics/reports  │
└─────────────────────────────────┘
```
The entire flow is **fully automated** — once set up, new files are ingested, transformed, and available for analysis without any manual work.


## Key Snowflake Features Used

| Feature | Purpose |
|---------|---------|
| Storage Integration | Secure, credential-free access to GCS |
| Notification Integration | Event-driven trigger via GCP Pub/Sub |
| Snowpipe | Continuous, serverless auto-ingest |
| Stream | Change Data Capture on raw table |
| Task | Scheduled incremental transformation |
| MERGE | Upsert/delete logic for clean table |
| Time Travel + CLONE | Point-in-time data recovery |

---



