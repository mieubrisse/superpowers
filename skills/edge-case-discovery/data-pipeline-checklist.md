# Data Pipeline Edge Case Checklist

Additional edge cases specific to data processing, ETL, and streaming systems.

## Schema and Format

- Schema changes upstream (new fields, removed fields, type changes)
- Mixed schema versions in the same batch (gradual rollout upstream)
- Encoding mismatches (UTF-8 vs. Latin-1, BOM presence)
- Delimiter conflicts in CSV (commas in quoted fields, newlines in values)
- Nested or recursive data structures exceeding expected depth
- Numeric precision loss (floating point, large integers in JSON)

## Ordering and Delivery

- Out-of-order messages (event timestamps vs. arrival timestamps)
- Duplicate messages (at-least-once delivery semantics)
- Missing messages (gaps in sequence numbers)
- Late-arriving data (event arrives after its processing window closed)
- Poison messages (single bad record blocks entire batch)

## Volume and Scale

- Empty batch (zero records to process)
- Single record batch (boundary for batch-size assumptions)
- Extremely large batch (memory pressure, timeout risk)
- Sustained high throughput (backpressure, queue depth growth)
- Data skew (one partition has 99% of records)

## Failure and Recovery

- Process crashes mid-batch — can it resume without reprocessing?
- Idempotent reprocessing — rerunning produces same result?
- Partial writes to destination (some rows committed, then failure)
- Destination unavailable during write (retry? buffer? drop?)
- Checkpoint/offset management (what is the committed position after failure?)

## Time and Windowing

- Time zone handling in timestamps (UTC vs. local, offset changes)
- Daylight saving time transitions (23-hour and 25-hour days)
- Clock skew between producer and consumer
- Processing windows that span midnight, month boundaries, year boundaries
- Backfill of historical data (reprocessing old data through current pipeline)

## Data Quality

- Null values in required fields
- Default/sentinel values that look like real data (0, -1, "N/A", epoch timestamp)
- Referential integrity failures (foreign key points to nonexistent record)
- Data that is technically valid but semantically wrong (future dates, negative ages)
