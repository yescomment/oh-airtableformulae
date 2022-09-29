# AirTable formulae for OHNY

Copy and paste into fields based on filenames.

## Line-by-line annotation

Using `WEB- Day 2 End Times` as example

**NOTE:** This is _not_ JavaScript, but that seems to be closest syntax to [AirTable formuae](https://support.airtable.com/docs/formula-field-reference) syntax.

```js
IF({Saturday End Time (Open Access)},
  // if event is Open Access, we assume just one time range per day
  // (this is **NOT** always true, but it was quicker to just manually update the exceptions
  // however, this may be an issue now that we are syncing live from AirTable and cannot override
  // if so, you'll need to loop this next block similar to how the Advance Reservation blocks do
  DATETIME_FORMAT(
    // we only need times (in h:mma formatstring, see below)
    SET_TIMEZONE(
      // AirTable super annoyingly stores all datetimes in some arcane timezone
      // (it's like EST+13 ??)
      // so every time must be explicitly parsed as EST
      {Saturday End Time (Open Access)},
      "America/New_York"
    ),
    "h:mma"
  ),
  // else, we loop through each AR time & pull in each start time for a given day
  // again, parse as America/New_York, then append `9999`, then format as `YYYY-MM-DD`, then compare to hardcoded relevant OHNY datestring
  // the `9999` is a dumb hack to prevent blank AirTable fields from throwing an `#ERROR` and breaking the whole formula:
  // on blank fields, it parses as date `9998-12-31`
  // on valid datetimes, it harmlessly seems to just add milliseconds?
  // either way, it's fine because all we're doing is testing if the date === `2022-10-22` or not.
  // Times are irrelevant at this point (_unless_ they cause the date to rollover, which could be an issue if we have near- or cross-midnight events)
  IF(  DATETIME_FORMAT(SET_TIMEZONE({End Time 1 (Advance Reservations)} & "9999", "America/New_York"), "YYYY-MM-DD")  = "2022-10-22",
    DATETIME_FORMAT(
      SET_TIMEZONE(
        {End Time 1 (Advance Reservations)},
        "America/New_York"
      ),
      // comma after each so final string forms ACF-valid CSV strings for WordPress import (trailing comma is okay)
      "h:mma, "
    ),
    // otherwise, return empty string (remember, we're in an IF within the ELSE of a previous IF)
    BLANK()
  )
  // then keep concatenating! (`&` is string concatenation in AirTable)
  & IF(  DATETIME_FORMAT(SET_TIMEZONE({End Time 2 (Advance Reservations)} & "9999", "America/New_York"), "YYYY-MM-DD")  = "2022-10-22",
    DATETIME_FORMAT(
      SET_TIMEZONE(
        {End Time 2 (Advance Reservations)},
        "America/New_York"
      ),
      "h:mma, "
    ),
    BLANK()
  )
  // loop through all 22 start/end fields
  ⋮
  // nothing unique about last record
  // I'd previously omitted the final comma, but leaving in for easier expandability
  // Remember, only like 1 or 2 AR Sites actually get to this point in the loop
  & IF(  DATETIME_FORMAT(SET_TIMEZONE({End Time 22 (Advance Reservations)} & "9999", "America/New_York"), "YYYY-MM-DD")  = "2022-10-22",
    DATETIME_FORMAT(
      SET_TIMEZONE(
        {End Time 22 (Advance Reservations)},
        "America/New_York"
      ),
      "h:mma, "
    ),
    BLANK()
  )
// ENDIF to end all ENDIFs
)
```

Yes, you can safely just RegEx `End Time` to `Start Time` to convert between.

Don't hesitate to email me if you chance yourself upon these —[Jacob](//jacobford.com/card)
