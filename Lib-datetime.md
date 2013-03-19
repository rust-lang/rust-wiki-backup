## 1. Announcement to mailing list

  - Proposed editor: _your name_
  - Date proposed: _date of proposal_
  - Link: _link to email_

###  Notes from discussion on mailing list

  - _note_
  - _note_
  - _note_

## 2. Research of standards and techniques

  1. Standard: Coordinated Universal Time (UTC)
    - http://en.wikipedia.org/wiki/Coordinated_Universal_Time
    - [Leap second](http://en.wikipedia.org/wiki/Leap_second)

### Summary of research on standards and leading techniques

#### Relevant standards and techniques exist?

* Concepts
  * **Time Point**: a location in the time continuum
  * **Duration**: a length of time
  * **Period**: a duration attached to a specific point in the time continuum
  * **Resolution**: is defined by the smallest representable duration
  * **Time System**:
  * **Calendar System**: http://en.wikipedia.org/wiki/Calendar#Calendar_systems
* Calendars
  * [Gregorian Calendar](http://en.wikipedia.org/wiki/Gregorian_calendar): internationally the most widely accepted and used civil calendar.
* Time Zones

#### Those intended to follow (and why)

#### Those intended to ignore (and why)

## 3. Research of libraries from other languages

  1. Language: C++/Boost
    - [Boost.Date_Time](http://www.boost.org/doc/libs/1_53_0/doc/html/date_time.html)
        - [Gregorian](http://www.boost.org/doc/libs/1_53_0/doc/html/date_time/gregorian.html)
        - [Posix Time](http://www.boost.org/doc/libs/1_53_0/doc/html/date_time/posix_time.html)
        - [Local Time](http://www.boost.org/doc/libs/1_53_0/doc/html/date_time/local_time.html)

  2. Language: Haskell
    - [Data.Time](http://www.haskell.org/ghc/docs/latest/html/libraries/time-1.4.0.1/index.html)
        - ...

  3. Language: Python
    - [datetime](http://docs.python.org/3.3/library/datetime.html)
        - [datetime.datetime](http://docs.python.org/3.3/library/datetime.html#datetime.datetime)
        - [datetime.timedelta](http://docs.python.org/3.3/library/datetime.html#datetime.timedelta)
        - [datetime.timezone](http://docs.python.org/3.3/library/datetime.html#datetime.timezone)
    - [calendar](http://docs.python.org/3.3/library/calendar.html#module-calendar)

  4. Language: Java
    - [java.util.Date](http://docs.oracle.com/javase/7/docs/api/java/util/Date.html): a time period with millisecond duration
    - [java.util.Calendar](http://docs.oracle.com/javase/7/docs/api/java/util/Calendar.html)
        - [java.util.GregorianCalendar](http://docs.oracle.com/javase/7/docs/api/java/util/GregorianCalendar.html)
    - [java.util.TimeZone](http://docs.oracle.com/javase/7/docs/api/java/util/TimeZone.html)
        - [java.util.SimpleTimeZone](http://docs.oracle.com/javase/7/docs/api/java/util/SimpleTimeZone.html)

### Summary of research from other languages:
#### Structures and functions commonly appearing
#### Variations on implementation seen
#### Pitfalls and hazards associated with each variant
#### Relationship to other libraries and/or abstract interfaces

## 4. Module writing

  - Pull request: _link to bug_

### Additional implementation notes

  - _note_
  - _note_
  - _note_