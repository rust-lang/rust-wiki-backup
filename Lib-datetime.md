A library for date, time and date/time.

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
  2. Standard: Julian Day
    - http://en.wikipedia.org/wiki/Julian_day
  3. Standard: TAI (Temps Atomique International)
    - The underlying atomic time, non-leap seconds
    - [International Atomic Time](https://en.wikipedia.org/wiki/International_Atomic_Time)
    - [UTC vs. TAI discussion](http://cr.yp.to/proto/utctai.html)

### Summary of research on standards and leading techniques

Things to be aware of (among others):
  * leap seconds
  * infinity
  * high resolution
  * network time sources
  * ...

#### Relevant standards and techniques exist?

* Concepts
  * **Time Point**: a location in the time continuum
  * **Duration**: a length of time
  * **Interval** / **Period** : a duration attached to a time point
  * **Resolution**: is defined by the smallest representable duration
  * **Time System**: ?
  * **Calendar System**: ?
  * **Time Zone**: ?
* Calendars
  * [Gregorian Calendar](http://en.wikipedia.org/wiki/Gregorian_calendar): internationally the most widely accepted and used civil calendar.
* Time Zones

#### Those intended to follow (and why)

#### Those intended to ignore (and why)

## 3. Research of libraries from other languages

  1. Language: C
    - [time.h](http://pubs.opengroup.org/onlinepubs/7908799/xsh/time.h.html)
    - [libtai](http://cr.yp.to/libtai.html)

  1. Language: C++
    - [Boost.Date_Time](http://www.boost.org/doc/libs/1_53_0/doc/html/date_time.html)
        - boost::date_time
            - [boost::date_time::date](http://www.boost.org/doc/libs/1_53_0/doc/html/boost/date_time/date.html)
            - [boost::date_time::period] (http://www.boost.org/doc/libs/1_53_0/doc/html/boost/date_time/period.html)
        - boost::date_time::gregorian
          - [Gregorian](http://www.boost.org/doc/libs/1_53_0/doc/html/date_time/gregorian.html)
            - valid date range: 1400-01-01 to 9999-12-03
        - boost::date_time::posix_time
            - [Posix Time](http://www.boost.org/doc/libs/1_53_0/doc/html/date_time/posix_time.html)
    - Qt
        - [QDate](http://qt-project.org/doc/qt-5.0/qtcore/qdate.html)
            - uses Julian Day count internally
        - [QTime](http://qt-project.org/doc/qt-5.0/qtcore/qtime.html)
        - [QDateTime](http://qt-project.org/doc/qt-5.0/qtcore/qdatetime.html)


  1. Language: Haskell
    - [Data.Time](http://www.haskell.org/ghc/docs/latest/html/libraries/time-1.4.0.1/index.html)
        - uses Modified Julian Day count internally
        - day 0 := 1858-11-17

  1. Language: Python
    - [datetime](http://docs.python.org/3.3/library/datetime.html)
        - distinguishes between "naive" and "aware" date/time objects
        - [datetime.date](http://docs.python.org/3.3/library/datetime.html#datetime.date)
        - [datetime.time](http://docs.python.org/3.3/library/datetime.html#datetime.time)
        - [datetime.datetime](http://docs.python.org/3.3/library/datetime.html#datetime.datetime)
        - [datetime.timedelta](http://docs.python.org/3.3/library/datetime.html#datetime.timedelta)
            - uses days, seconds and microseconds internally
        - [datetime.timezone](http://docs.python.org/3.3/library/datetime.html#datetime.timezone)
    - [time](http://docs.python.org/3.3/library/time.html)
    - [calendar](http://docs.python.org/3.3/library/calendar.html#module-calendar)

  4. Language: Java
    - java.util
        - [java.util.Date](http://docs.oracle.com/javase/7/docs/api/java/util/Date.html): a time period with millisecond duration
        - [java.util.Calendar](http://docs.oracle.com/javase/7/docs/api/java/util/Calendar.html)
        - [java.util.GregorianCalendar](http://docs.oracle.com/javase/7/docs/api/java/util/GregorianCalendar.html)
        - [java.util.TimeZone](http://docs.oracle.com/javase/7/docs/api/java/util/TimeZone.html)
        - [java.util.SimpleTimeZone](http://docs.oracle.com/javase/7/docs/api/java/util/SimpleTimeZone.html)
    - [Joda-Time](https://github.com/JodaOrg/joda-time)
    - [JSR-310](https://github.com/ThreeTen/threeten)

### Summary of research from other languages:
#### Structures and functions commonly appearing
#### Variations on implementation seen
#### Pitfalls and hazards associated with each variant
#### Relationship to other libraries and/or abstract interfaces

## 4. Module writing

  - Pull request: _link to bug_

### Requirements

#### Calculations

  * **Timepoint +/- Duration** gives a Timepoint relative to the former
  * **Timepoint - Timepoint** gives the Duration between them
  * **Duration +/- Duration** gives an extended/reduced Duration
    * what about negative Durations?
  * **Period +/- Duration** gives an extended/reduced Period
    * what about negative Durations?

#### Conversion to and from strings

  * **Date to ISO 8601 string** conversion
  * **ISO 8601 string to Date** conversion