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

The year can be short or full.

```
0: unset
1...100: short year 0 to 99
256...65535: full year -55280 to 9999
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
65...72: day 1 to 7 of week (Monday to Sunday)
81...87: Monday to Sunday (week of)
129...220: day 1 to 92 of quarter
257...439: day 1 to 183 of semester
63: last day of month
255: last day of quarter
511: last day of semester
```

If the full year is set:

```
0: unset
1...31: day 1 to 31 of month
65...72: day 1 to 7 of week (Monday to Sunday)
81...87: Monday to Sunday (week of)
129...494: day 1 to 366 of year
63: last day of month
511: last day of year
```

A note about values between 81 and 87: when that is selected, the week in the Week field is only counted when it contains the selected day.
In practice that allows for example to describe the _first, second_, etc up to _last Tuesday_ of a month.

A Moment cannot encode both the _day of week_ and the _day of month_. When both are given, the moment is assumed to be on a specific month and year which must be guessed based on the current date (closest matching date in the future or in the past) and _full year_, _month of year_ must be encoded in the Moment.

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
