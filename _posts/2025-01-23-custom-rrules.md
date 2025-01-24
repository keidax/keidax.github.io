---
layout: post
title: "Adding custom recurrences to Google Calendar"
date: 2025-01-23 15:55:00 -0500
tags: update
---

_This post mostly serves as documentation for my future self, but I hope it's useful for you too!_

Google Calendar supports recurring events, with rules like "Weekly on Wednesday" or "Monthly on the second Sunday".
But did you know there are other configurations for recurring events, which can't be set through the regular calendar UI?

Suppose your rent is due on the first of every month.
You want to set a calendar reminder for the _last_ day of the month, so you don't forget rent is due tomorrow.
But "last day of the month" isn't an option here:

<img
  src="/assets/images/custom-rrules/repeat_options.png"
  alt="Screenshot of a Google Calendar event showing the available options for scheduling a repeating event"
  width="50%">

Instead, you can create an event in [iCalendar format][rfc5545] with a custom RRULE, and import it into Google Calendar.

### ICS Template

Here's a simple template for a recurring event:

```
BEGIN:VCALENDAR
VERSION:2.0
PRODID:bespoke text file
CALSCALE:GREGORIAN
BEGIN:VEVENT
SUMMARY:<title of event>
DTSTART:<start time of first occurrence>
DTEND:<end time of first occurrence>
RRULE:<rrule>
END:VEVENT
END:VCALENDAR
```

Save this in a file like `my_recurring_event.ics`, then fill in the missing fields.
`SUMMARY` should just be the title of the event.

### Timestamps

`DTSTART` and `DTEND` are the start and end times of the first occurrence.

For an all-day event, these should just be the date, like `20250131`.

For an event with a specific time, these fields can be written as `<date>T<time>`, such as `20250131T120000`.
Google Calendar will assume these times are for your calendar's primary time zone.

Or, for time zone precision, you can specify the times in UTC with a `Z` suffix, like `20250131T170000Z`

### Recurrence Rule

Now we get to the juicy bit!
The full specification for what's possible in an `RRULE` is pretty involved (see [RFC 5545 Section 3.3.10][rfc5545-3.3.10] for the details).

In our example of the last day of the month, we would want:

```
RRULE:FREQ=MONTHLY;INTERVAL=1;BYMONTHDAY=-1
```

`FREQ=MONTHLY` means the rule happens on a monthly basis.
`INTERVAL=1` means it happens once per `FREQ`, so every month.
And `BYMONTHDAY=-1` means the last day of the month!

What if you wanted to represent the second-to-last day of every other month?
That's as easy as:

```
RRULE:FREQ=MONTHLY;INTERVAL=2;BYMONTHDAY=-2
```

There is quite a lot you could do with custom rules.
Another one I've used in the past is for the 1st, 3rd, and 5th Saturday of every month:
```
RRULE:FREQ=MONTHLY;BYDAY=1SA,3SA,5SA
```

What about the last Friday of every year?
```
RRULE:FREQ=YEARLY;BYDAY=-1FR
```

Here's a rule to match February 28, on leap years only:
```
RRULE:FREQ=YEARLY;BYMONTH=2;BYMONTHDAY=28,29;BYSETPOS=-2
```

### Importing

Now that you have a completed ICS file, you just need to import it.
Go to Google Calendar settings, find the "Import & export" section, and import the file.

Write a sufficiently complex `RRULE` and you'll be rewarded with this:

<img
  src="./assets/images/custom-rrules/unsupported_recurrence.png"
  alt="Screenshot of a Google Calendar event that says 'Unsupported recurrence'"
  width="50%">

<span class="emoji">😎</span>

---

[rfc5545]: https://datatracker.ietf.org/doc/html/rfc5545
[rfc5545-3.3.10]: https://datatracker.ietf.org/doc/html/rfc5545#section-3.3.10
