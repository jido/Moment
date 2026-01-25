# Moment

Moment is a format that describes a partially defined date or time, such as the time of a coming appointment in common speech.

For example:

> Tuesday at 4:15

There is no year, month, half of the day specified. All parts are optional in a Moment.


## Format details

The format fits in a 64 bit word with the following fields:

```
yyyyyyyy yyyyyyyM MMMddddd ddddwwww wwDDDhhh hhmmmmmm ssssssSS SSSSSSSS

y: year (15 bit)
M: month (4 bit)
d: day (9 bit)
w: week (6 bit)
D: day of week (3 bit)
h: hour (5 bit)
m: minute (6 bit)
s: second (6 bit)
S: millisecond (10 bit)
```

The value 0 means the field is unset. There are other values that also don't set a value for the field but affect other fields.


### Year field

The year can be short or long. The Year field can also contain a _quarter_ or _semester_ number.

```
0: unset
1...100: short year 0 to 99
129...228: Q1 or S1 in year 0 to 99
257...356: Q2 or S2 in year 0 to 99
385...484: Q3 in year 0 to 99
513...612: Q4 in year 0 to 99
641...740: a quarter or semester in year 0 to 99
768...32767: long year -22000 to 9999
128: Q1 or S1
256: Q2 or S2
384: Q3
512: Q4
640: any quarter or semester
```


### Month field

If a _quarter_ or _semester_ is selected in the Year field.

```
0: unset
1...3: month 1 to 3 of quarter
9...14: month 1 to 6 of semester
8: semester only
```

Otherwise:

```
0: unset
1...12: month 1 to 12 of year
```

The _semester only_ option should be used when none of _month of semester_, _day of semester_ or _week of semester_ is selected.


### Day field

The Day field can contain a day of the month, the quarter, the semester or the year. It can also control how the week number in the Week field should be interpreted.

If a _quarter_ or _semester_ is selected in the Year field:

```
0: unset
1...31: day 1 to 31 of month
129...220: day 1 to 92 of quarter
257...439: day 1 to 183 of semester
63: last day of month
255: last day of quarter
511: last day of semester
64: week starting on day, week 1 contains day
96: week starting Monday, week 1 contains a Sunday
112: week starting Sunday, week 1 contains a Saturday
```

Otherwise:

```
0: unset
1...31: day 1 to 31 of month
129...494: day 1 to 366 of year
63: last day of month
511: last day of year
64: week starting on day, week 1 contains day
96: week starting Monday, week 1 contains a Sunday
112: week starting Sunday, week 1 contains a Saturday
```

The values 64, 96 and 112 have an effect on the Week field.


### Week field

If a _quarter_ or _semester_ is selected in the Year field:

```
0: unset
1...5: week 1 to 5 of month
17...30: week 1 to 14 of quarter
33...59: week 1 to 27 of semester
7: last week of month
31: last week of quarter
63: last week of semester
```

Otherwise:

```
0: unset
1...5: week 1 to 5 of month
9...61: week 1 to 53 of year
7: last week of month
63: last week of year
```

By default the week number follows the ISO 8601 standard: week starting Monday, week 1 contains a Thursday.

The Day field can be used to change it. The other options are _week starting on day, week 1 contains day_ (to use together with a _day of week_), _week starting Monday, week 1 contains a Sunday_ and _week starting Sunday, week 1 contains a Saturday_.


### Day of week

```
0: unset
1...7: day 1 to 7 of week
```

By default the week begins on Monday and ends on Sunday. The Day field offers an option to begin it on Sunday and end it on Saturday.

With the "week starting on day" option a week is only counted when it contains the selected day.

In practice that allows for example to describe the _first, second_, etc up to _last Tuesday_ of a month.


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
1023: twelve-hour clock
```

When milliseconds are given the hour is always assumed to be based on a 24-hour clock.


## Examples

* Tuesday at 4:15

```
0x00000000115003ff
```

Day of week = 2
: _day of week_, **Tuesday**
 
Hour = 5
: _hour (twelve-hour clock)_, **4**

Minute = 16
: _minute_, **15**

Millisecond = 1023
: _twelve-hour clock_

---

* The third Monday of the quarter

```
0x0500 0404 c800
```

Year = 640
: **any quarter or semester**

Day = 64
: _week starting on day, week 1 contains day_

Week = 19
: _week of quarter_, **3**

Day of week = 1
: _day of week_, **Monday**

---

* 7:01:33.239

```
0x00000000020288f0 
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
0xc1b2 c0b0 1800 0000
```

Year = 24793
: _long year_, **2025**

Month = 6
: _month of year_, **June**

Day = 11
: _day of month_, **11**

Day of week = 3
: _day of week_, **Wednesday**

---

* 23:59 on last day of 98

```
0x00c61ff0063c0000
```

Year = 99
: _short year_, **98**

Day = 511
: _day of year_, **last**

Hour = 24
: _hour_, **23**

Minute = 60
: _minute_, **59**
