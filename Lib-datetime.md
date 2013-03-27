A library for date, time and date/time.

### Concepts

  * **Instant**: an infinitesimal location in the time continuum.
  * **Time Interval**: a part of the the time continuum limited by two instants.
  * **Time Scale**: a system of ordered marks of instants. One instant is chosen as the Origin, see Epoch.
  * **Epoch**: the origin of a particular era, e.g. the Incarnation of Jesus.
  * **Time Point**: an instant, defined by the means of a Time Scale.
  * **Duration**: a non-negative length of time.
  * **International Atomic Time (TAI)**: ?
  * **Coordinated Universal Time (UTC)**: is the primary time scale by which the world regulates clocks and time.
  * **Standard Time**: the result of synchronizing clocks in different geographical locations within a time zone to the same time.

See ISO 8601 which relies on IEC 60050-111, IEC 60050-713.

### Requirements

#### Things to be aware of
  * leap seconds
  * infinity
  * high resolution
  * network time sources
  * daylight saving time

#### Types

  * **Date** represents time points with a resolution of days
  * **Time** represents XXX
  * **DateTime** represents time points with a resolution of XXX
  * **Month** an enum: January, February, ...
  * **Weekday** an enum: Monday, Tuesday, ...

#### Conversion to/from other date representations

  * **Date -> year, month, day**
  * **Date <- year, month, day**
    * Feb 29 should be legal/illegal depending on leap years
  * **Date <-> Julian Day**
  * **Date -> year number**
  * **Date -> month number**
    * should we use an enum instead?
  * **Date -> week number**
  * **Date -> day number**
  * **Date -> day of year number**
  * **Date -> day of week number**
    * should we use an enum instead?

#### Calculations

  * **Instant +/- Duration** gives a Instant relative to the former
  * **Instant - Instant** gives the Duration between them
  * **Duration +/- Duration** gives an extended/reduced Duration
    * what about negative Durations?
  * **Period +/- Duration** gives an extended/reduced Period
    * what about negative Durations?

#### Conversion to/from strings

  * **Date <-> user formatted string**
  * **Date <-> ISO 8601 string**
  * **Date <-> localized string**

## 1. Announcement to mailing list

  - Proposed editor: _your name_
  - Date proposed: _date of proposal_
  - Link: _link to email_

###  Notes from discussion on mailing list

  - _note_
  - _note_
  - _note_

## 2. Research of standards and techniques

  1. [ISO 31-1](http://en.wikipedia.org/wiki/ISO_31-1) defines the units **day, hour, minute, second** for time, time interval and duration
  1. [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601)
    - uses the [Gregorian calendar](http://en.wikipedia.org/wiki/Gregorian_calendar) to identify calendar days
  1. [ITU-R TF.460-5](http://www.itu.int/dms_pubrec/itu-r/rec/tf/R-REC-TF.460-5-199710-S!!PDF-E.pdf)
  1. Standard: Coordinated Universal Time (UTC)
    - http://en.wikipedia.org/wiki/Coordinated_Universal_Time
    - [Leap second](http://en.wikipedia.org/wiki/Leap_second)
  1. Standard: Julian Day
    - http://en.wikipedia.org/wiki/Julian_day#Alternatives
  1. Standard: TAI (Temps Atomique International)
    - The underlying atomic time, non-leap seconds
    - [International Atomic Time](https://en.wikipedia.org/wiki/International_Atomic_Time)
    - [UTC vs. TAI discussion](http://cr.yp.to/proto/utctai.html)
  1. Network Time Protocol
    - http://en.wikipedia.org/wiki/Network_Time_Protocol


### Summary of research on standards and leading techniques

#### Relevant standards and techniques exist?
#### Those intended to follow (and why)

#### Those intended to ignore (and why)



## 3. Research of libraries from other languages

### Summary of research from other languages:

  * [Proposal to Add Date-Time to the C++ Standard Library](http://www.crystalclearsoftware.com/libraries/date_time/proposal_v75/date_time_standard.html)
  * https://github.com/ThreeTen/threeten/wiki/Multi-calendar-system

#### Simple immutable objects 
Used by: Boost, Cocoa, JSR-310

#### Gregorian Calendar

#### Date and Time Types

There are two possible approaches:

  * having separate types for Date, Time and Date/Time
    * Used by: Qt
    * Cons:
      * ?
    * Pros:
      * ?
  * having only one type for Date/Time 
    * Used by: Go
    * Cons:
      * ?
    * Pros:
      * ?
   
#### Underlying Representation

  * floats
    * Used by: ?
    * Cons:
      * ?
    * Pros:
      * ?
  * integers
    * Used by: Boost
    * Cons:
      * ?
    * Pros:
      * ?

#### Date and Time Types

#### Epochs

  * **Julian Day**: -4714-11-24T12:00:00Z
    * Used by: ?
  * **Modified Julian Day**: 1858-11-17T00:00:00Z
    * Used by: Haskell
  * **Unix Time**: 1970-01-01T00:00:00Z
    * Used by: Go, Unix 
  * **2001-01-01T00:00:00Z**
    * Used by: Apple Cocoa


#### Structures and functions commonly appearing

  * the primary calendar system is ISO 8601
  * other calendar systems focus on user input/output localization 

#### Variations on implementation seen

#### Pitfalls and hazards associated with each variant

#### Relationship to other libraries and/or abstract interfaces

#### Reference
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
        - [Unit tests](http://svn.boost.org/svn/boost/trunk/libs/date_time/test/)
    - Qt
        - [QDate](http://qt-project.org/doc/qt-5.0/qtcore/qdate.html)
            - uses Julian Day count internally
        - [QTime](http://qt-project.org/doc/qt-5.0/qtcore/qtime.html): a clock time since midnight
        - [QDateTime](http://qt-project.org/doc/qt-5.0/qtcore/qdatetime.html)
        - Unit tests:
            - https://qt.gitorious.org/qt/qt/blobs/4.8/tests/auto/qdate/tst_qdate.cpp
            - https://qt.gitorious.org/qt/qt/blobs/4.8/tests/auto/qtime/tst_qtime.cpp
            - https://qt.gitorious.org/qt/qt/blobs/4.8/tests/auto/qdatetime/tst_qdatetime.cpp
    - QuantLib
        - [Date](http://quantlib.org/reference/class_quant_lib_1_1_date.html)
        - [Period](http://quantlib.org/reference/class_quant_lib_1_1_period.html)
        - [Calendar](http://quantlib.org/reference/class_quant_lib_1_1_calendar.html)
  1. Language: Go
    - [time](http://golang.org/pkg/time/)
        - [Duration](http://golang.org/pkg/time/#Duration): a duration with an int64 nanosecond count.
        - [Location](http://golang.org/pkg/time/#Location): maps time instants to the zone in use at that time.
        - [Time](http://golang.org/pkg/time/#Time): an instant in time with nanosecond precision. Consists of:
            - Seconds (int64) since the Unix epoch
            - Nanoseconds (int32)
            - Location
  1. Language: Haskell
    - [Data.Time](http://www.haskell.org/ghc/docs/latest/html/libraries/time-1.4.0.1/index.html)
        - uses Modified Julian Day count internally
  1. Language: Java
    - [JSR-310](https://github.com/ThreeTen/threeten)
    - [Joda-Time](https://github.com/JodaOrg/joda-time)
    - java.util
        - [java.util.Date](http://docs.oracle.com/javase/7/docs/api/java/util/Date.html): a time period with millisecond duration
        - [java.util.Calendar](http://docs.oracle.com/javase/7/docs/api/java/util/Calendar.html)
        - [java.util.GregorianCalendar](http://docs.oracle.com/javase/7/docs/api/java/util/GregorianCalendar.html)
        - [java.util.TimeZone](http://docs.oracle.com/javase/7/docs/api/java/util/TimeZone.html)
        - [java.util.SimpleTimeZone](http://docs.oracle.com/javase/7/docs/api/java/util/SimpleTimeZone.html)
  1. Language: Objective-C
    - [Cocoa Date and Time Programming Guide](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/DatesAndTimes/DatesAndTimes.html)
      - [NSDate](https://developer.apple.com/library/mac/#documentation/Cocoa/Reference/Foundation/Classes/NSDate_Class/Reference/Reference.html)
      - [NSCalendar](https://developer.apple.com/library/mac/#documentation/Cocoa/Reference/Foundation/Classes/NSCalendar_Class/Reference/NSCalendar.html)
      - [NSDateComponents](https://developer.apple.com/library/mac/#documentation/Cocoa/Reference/Foundation/Classes/NSDateComponents_Class/Reference/Reference.html)
  1. Language: Python
    - builtin
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
    - [pytz](http://pytz.sourceforge.net/)
    - [dateutil](http://labix.org/python-dateutil)
  1. Language: Ruby
    - [Date](http://ruby-doc.org/stdlib-2.0/libdoc/date/rdoc/Date.html)
  1. Language: Scheme
    - [SRFI 19](http://srfi.schemers.org/srfi-19/srfi-19.html)

## 4. Module writing

  - Pull request: _link to bug_