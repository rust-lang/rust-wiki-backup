One or more core developers spend one day per week mopping up accumulated mess in the bug tracking system. This includes:

  - Initial triage: tagging, assigning, setting milestones, and eliminating incoming duplicates.
  - Source alignment: making FIXMEs in code correspond to bug numbers and vice versa.
      * In general, the tidy script enforces that FIXMEs go with bug numbers. It's still worth checking occasionally that the bug numbers refer to open bugs. It's also worth looking for the handiwork of people who defeated the tidy script by using something like "XXX" instead of "FIXME".
      * As of 2012-10-11, all FIXMEs in the Rust code correspond to valid, open bug numbers (tjc)
  - Reproduction: checking locally to see if bugs are still valid.
  - Followup: requesting clarification, testcases, etc. from reporters of unclear bugs. 
  - Actual fixing: if there's any time left, pick some work off that seems fixable in short order.
  - Documentation: perhaps leave a note on this wiki page about what you did, for the next janitor's benefit.

## Janitors by day:

  - Monday: @brson
  - Tuesday: @nikomatsakis
  - Wednesday: @graydon
  - Thursday: @catamorphism (tjc on IRC)
  - Friday: @pcwalton

## Log:
  - Thursday 11/8, tjc reviewed open issues 3900-3941
  - Thursday 11/1, tjc reviewed open issues 3861-3899 and 1801-2208
  - Thursday 10/25, tjc reviewed open issues 3807-3860 and 1051-1800
  - Thursday 10/18, tjc reviewed open issues 3724-3806
  - Thursday 10/11, tjc reviewed open issues 3403-3723 and 1-1050
  - Thursday 9/6, tjc reviewed open issues 3264-3402
  - Thursday 8/23, tjc reviewed open issues 3164-3263
  - Thursday 8/9, tjc reviewed open issues 3091-3163 and 2000-2250
  - Thursday 8/2, tjc reviewed open issues 1501-1999 and 3039-3090
  - Thursday 7/26, tjc reviewed open issues 2971-3038 and 808-1500
  - Thursday 7/19, tjc reviewed open issues 2882-2970 and 1-807
  - Thursday 7/12, tjc reviewed open issues 2807-2881
  - Thursday 7/5, tjc reviewed open issues 2736-2806
  - Thursday 6/28, tjc reviewed open issues 2658-2735.
  - Thursday 6/21, tjc reviewed open issues 2621-2657.
  - Thursday 6/14, tjc reviewed open issues 2527-2590.
  - Thursday 6/7, tjc reviewed open issues 2396-2526.
  - Thursday 5/17, tjc reviewed open issues 2376-2395.
  - Thursday 5/10, tjc reviewed open issues 2352-2374.
  - Thursday 5/3, tjc reviewed open issues 2304-2338.
  - Thursday 4/26, tjc reviewed open issues 2244-2303.
  - Thursday 4/19, tjc reviewed open issues 2001-2243.
  - Thursday 4/12, tjc reviewed open issues 1001-2000.
  - Thursday 3/15, tjc reviewed open issues 1-1000.