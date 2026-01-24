# Moment

Moment is a format that describes a partially defined date or time, such as the time of a coming appointment in common speech.

For example:

> Tuesday at 4:15

There is no year, month, half of the day specified. All parts are optional in a Moment.

## Format details

The format fits in a 64 bit word with the following fields:

```
yyyyyyyy yyyyyyyy MMMMMwww wwwddddd ddddFhhh hhmmmmmm ssssssSS SSSSSSSS

y: year (16 bit)
M: month (5 bit)
w: week (6 bit)
d: day (9 bit)
F: full year flag (1 bit)
h: hour (5 bit)
m: minute (6 bit)
s: second (6 bit)
S: millisecond (10 bit)
```

The value 0 means the field is unset. As an exception, if the millisecond value is `0x3ff` it is also unset (that value has a special meaning).

### Year field

The year can be short or long.

```
0: unset
1...100: short year 0 to 99
256...65535: long year -55280 to 9999
```

### Month field

The Month field can also contain a _quarter_ or a _semester_ number.

```
0: unset
1...12: month 1 to 12 of year
14...15: semester 1 to 2
17...22: month 1 to 6 of semester
25...27: month 1 to 3 of quarter
28...31: quarter 1 to 4
```

### Week field

If the full year flag is unset, _week of quarter_ and _week of semester_ are available:

```
0: unset
1...5: week 1 to 5 of month
17...30: week 1 to 14 of quarter
33...59: week 1 to 27 of semester
7: last week of month
31: last week of quarter
63: last week of semester
```

If the full year flag is set:

```
0: unset
1...5: week 1 to 5 of month
9...61: week 1 to 53 of year
7: last week of month
63: last week of year
```

### Day field

If the full year is unset, _day of quarter_ and _day of semester_ are available:

```
0: unset
1...31: day 1 to 31 of month
65...72: Monday to Sunday (week of)
81...87: day 1 to 7 of week (ISO 8601)
97...103: day 1 to 7 of week (Monday to Sunday)
113...119: day 1 to 7 of week (Sunday to Saturday)
129...220: day 1 to 92 of quarter
257...439: day 1 to 183 of semester
63: last day of month
96: week starting Monday, week 1 contains a Sunday
112: week starting Sunday, week 1 contains a Saturday
255: last day of quarter
511: last day of semester
```

If the full year is set:

```
0: unset
1...31: day 1 to 31 of month
65...72: Monday to Sunday (week of)
81...87: day 1 to 7 of week (ISO 8601)
97...103: day 1 to 7 of week (Monday to Sunday)
113...119: day 1 to 7 of week (Sunday to Saturday)
129...494: day 1 to 366 of year
63: last day of month
96: week starting Monday, week 1 contains a Sunday
112: week starting Sunday, week 1 contains a Saturday
511: last day of year
```

By default the week number in the Week field follows the ISO 8601 standard: Monday to Saturday, week 1 contains a Thursday. The other options are week starting Monday, week 1 contains a Sunday (_values 96...103_) and week starting Sunday, week 1 contains a Saturday (_values 112...119_). Day values _96_ and _112_ don't set a day but they have an effect on week numbers.

With the "week of \<day>" option the week in the Week field is only counted when it contains the selected day (_values 65...72_).
In practice that allows for example to describe the _first, second_, etc up to _last Tuesday_ of a month.

A Moment cannot encode both the _day of week_ and the _day of month_. When both are given, the moment is assumed to be on a specific month and year which must be guessed based on the date at the time of writing and encoded in the Moment (closest future or past date).

Alternatively the _day of week_ can be encoded in a separate Moment, if space permits.

### Full year flag

This flag controls how to read the Week and Day fields.

```
0: unset
1: set
```

When it is set, the _day of year_ or the _week of year_ value is used.

### Hour field

If the millisecond value is other than special value `0x3ff`:

```
0: unset
1...25: hour 0 to 24
```

If the millisecond value is special value `0x3ff`:

```
0: unset
2..13: hour 1 to 12 (twelve-hour clock)
```

When the special value is used that means _AM_ or _PM_ is not given for the hour value, as in the introductory example.

### Minute field

```
0: unset
1...60: minute 0 to 59
```

### Second field

```
0: unset
1...60: second 0 to 59
```

### Millisecond field

```
0: unset
1...1000: millisecond 0 to 999
1023: unset (special meaning)
```

When milliseconds are given the hour is always assumed to be based on a 24-hour clock.

## Examples

* Tuesday at 4:15

```
0x00000004215003ff
```

Day = 66
: _week of Tuesday_, **Tuesday**
 
Hour = 5
: _hour (twelve-hour clock)_, **4**

Minute = 16
: _minute_, **15**

Millisecond = 1023
: **unset**, means hour from 12-hour clock

---

* The third Monday of the quarter

```
0x0000002641000000
```

Week = 19
: _week of quarter_, **3**

Day = 65
: _week of Monday_, **Monday**

Full year flag = 0
: **unset**

---

* 7:01:33.239

```
0x0000 0000 020288f0 
```

Hour = 8
: _hour_, **7**

Minute = 2
: _minute_, **1**

Second = 34
: _second_, **33**

Millisecond = 240
: _millisecond_, **239**

---

* Wednesday, June the eleventh, 2025

```
0xe0d93000b0000000
```

Year = 57561
: _long year_, **2025**

Month = 6
: _month of year_, **June**

Day = 11
: _day of month_, **11**

---

* 23:59 on last day of 98

```
0x0063001ffe3c0000
```

Year = 99
: _short year_, **98**

Day = 511
: _day of year_, **last**

Full year flag = 1
: **set**

Hour = 24
: _hour_, **23**

Minute = 60
: _minute_, **59**
