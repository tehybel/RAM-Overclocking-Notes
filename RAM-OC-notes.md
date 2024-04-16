

Findings
--------
- Setting ODT skews (Wr, Nom, Park) correctly *massively* impacts how far you can take your OC
- ODT skews depend on other variables: I've personally observed IO and RAM voltage, and frequency. Therefore, fix these *before* optimizing ODT skews.
- 

Misc tips and tricks
------------------
- You can time how long memory training takes with a stopwatch
- 

Process for optimizing
----------------------
- use GSAT; anything else I tried gave inconsistent results and muddy data
- TODO document this.



About internet content
----------------------
TODO: write about trusted sources only, give some usernames?


DLL Bandwidth setting
---------------------
I found these values:

  Setting | Data
  | ------------ | --- |
  BW 0           | 504.00s 49072.32MB/s, with 38 hardware incidents,
  BW 1           | 554.00s 49404.96MB/s, with 53 hardware incidents
  BW 2           | 563.01s 49352.69MB/s, with 66 hardware incidents
  BW 3           | (way more errors than BW2, found in a previous test)
  auto           | 312.00s 49129.21MB/s, with 11 hardware incidents,

I've retried the experiment thrice now, to be sure. It seems that setting DLL BW to 'auto' simply performs better than any of the manually set options! 

Perhaps on auto, the board uses some mechanism to find a precisely tuned bandwidth, whereas 0/1/2/.../7 just correspond to specific frequencies that are spaced far apart? 


Memory training algorithms
--------------------------

You can sometimes find stability gains by moving specific trainings from 'Auto' to enabled/disabled. 

For example, I found:

  Early | Late | Data
  | ------------ | --- | --- |
  Early command training  ENABLED | Late command training DISABLED        | 312.00s 49129.21MB/s, with 11 hardware incidents
  Early command training DISABLED | Late command training  ENABLED        | 602.00s 49303.27MB/s, with 13 hardware incidents

Your frequency/board might prefer the opposite, so make sure to test both.


Interesting quotes
------------------




Recommended tools
-----------------

- [GSAT](https://github.com/stressapptest/stressapptest)
- TestMem5 (TM5) with [these configs](https://github.com/integralfx/MemTestHelper/tree/oc-guide/TM5-Configs)
- [y-cruncher](http://www.numberworld.org/y-cruncher/), especially the VST and VT3 stress tests



Useful links
------------
https://www.overclock.net/threads/the-importance-of-skew-control-for-memory-overclocking.1774358/post-28662268

