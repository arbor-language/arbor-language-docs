# Dates, times, and timezones

Temporal types distinguish calendar values, elapsed time, fixed offsets, and geographic
timezone rules. Choose the type that expresses what the value means.

!!! note "Design documentation"
This chapter describes the language design. Examples are illustrative; implementation
availability and unresolved rules are explained in [Design status](../about/status.md).

## Literal forms

Temporal values are first-class, with distinct meanings and types:

| Literal                                          | Type             | Meaning                             |
| ------------------------------------------------ | ---------------- | ----------------------------------- |
| `date'2000-04-18'`                               | `LocalDate`      | Calendar date                       |
| `time'09:30:00'`                                 | `LocalTime`      | Clock time                          |
| `local'2026-10-11T09:30:00'`                     | `LocalDateTime`  | Local date and time                 |
| `instant'2026-10-11T09:30:00Z'`                  | `Instant`        | Timeline point written in UTC       |
| `offset'2026-10-11T09:30:00+02:00'`              | `OffsetDateTime` | Date/time and fixed UTC offset      |
| `zoned'2026-10-11T09:30:00+02:00[Europe/Paris]'` | `ZonedDateTime`  | Date/time and named geographic zone |
| `tz'Europe/Paris'`                               | `TimeZone`       | Named timezone rules                |
| `duration'PT24H'`                                | `Duration`       | Elapsed time                        |
| `period'P1D'`                                    | `CalendarPeriod` | Calendar adjustment                 |

## Zone resolution and literal validation

The bracketed zone is an extension to the ISO-style representation. Zoned literals
include both offset and zone; the compiler checks their agreement. The two offsets
at a daylight-saving overlap identify different occurrences of the local time.
Construction without an explicit offset requires a resolution policy or raises a
declared exception for ambiguous or nonexistent times.

Literal rules are deliberately narrow: Gregorian `YYYY-MM-DD`, `HH:MM:SS` with
optional fractional seconds, uppercase `T`/`Z`, and `±HH:MM` offsets. Invalid dates
and unsupported precision are compile-time errors. No omitted seconds or alternate
date formats in literals. Runtime parsing may offer additional explicit formats.

The initial proposal excludes leap-second literals and documents that timeline
arithmetic does not count leap seconds. Final range and precision limits remain
open. Timezone data is versioned and pinned for reproducible builds and evaluation.

## Duration and calendar arithmetic

Adding a calendar day can differ from adding 24 hours. Month arithmetic needs an
explicit invalid-date policy. Coordinate-to-timezone lookup uses separate versioned
geographic data; named zones do not require coordinate lookup. Exact lookup APIs
and their v0.1 scope remain open.

## Related reading

- [Numbers and constants](numbers.md)
- [Decorators, accessors, and settings](decorators.md)
