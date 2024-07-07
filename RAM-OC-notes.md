

Findings
--------
- Setting ODT skews (Wr, Nom, Park) correctly *massively* impacts how far you can take your OC
- ODT skews depend on other variables: I've personally observed IO and RAM voltage, and frequency. Therefore, fix these *before* optimizing ODT skews.
- The Memory Try It! feature changes various settings behind the scenes, for example enabling or disabling some training algorithms that are set to Auto. Therefore it's worth testing different values of this feature, even if you override all the timings it sets for you.

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

On MSI z490, I found these values:

  Setting | Data
  | ------------ | --- |
  BW 0           | 504.00s 49072.32MB/s, with 38 hardware incidents,
  BW 1           | 554.00s 49404.96MB/s, with 53 hardware incidents
  BW 2           | 563.01s 49352.69MB/s, with 66 hardware incidents
  BW 3           | (way more errors than BW2, found in a previous test)
  auto           | 312.00s 49129.21MB/s, with 11 hardware incidents,

I've retried the experiment thrice to be sure: setting DLL BW to 'auto' simply performs better than any of the manually set options! 

Perhaps on auto, the board uses some mechanism to find a precisely tuned bandwidth, whereas 0/1/2/.../7 just correspond to specific frequencies that are spaced far apart? 


Memory training algorithms
--------------------------

You can sometimes find stability gains by moving specific trainings from 'Auto' to enabled/disabled. 

For example, I found:

  Early | Late | Data
  | ------------ | --- | --- |
  Enabled | Disabled        | 312.00s 49129.21MB/s, with 11 hardware incidents
  Disabled | Enabled        | 602.00s 49303.27MB/s, with 13 hardware incidents

Your frequency/board might prefer the opposite, so make sure to test both.


People confirmed to know what they're doing
-------------------------------------------
- 7empe
- Falkentyne
- Ichirou
- PhoenixMDA
- TODO add more

To decide who to listen to, check that their claims are backed up by results. Find a screenshot of their best memory overclock. If they have achieved outstanding results, their words deserve attention.

The better you get, the easier it will be to see through misinformation and vet people.



Interesting quotes
------------------

>  Restart causes post error 55. To avoid it I need to set Maximus Tweak Mode to 1 (from 2) and avoid fixing tRDRD_sg_training, tRDRD_sg_runtime, tRDRD_dg_training, tRDRD_dg_training. These have to be on auto, even with having the same 7 - 4 values after training. Setting them to fixed 7 - 4 gives me always post code 55 no matter how ridiculous VCCSA I would apply -- 7empe




Recommended tools
-----------------

- [GSAT](https://github.com/stressapptest/stressapptest)
- TestMem5 (TM5) with [these configs](https://github.com/integralfx/MemTestHelper/tree/oc-guide/TM5-Configs)
- [y-cruncher](http://www.numberworld.org/y-cruncher/), especially the VST and VT3 stress tests



Useful links
------------
https://www.overclock.net/threads/the-importance-of-skew-control-for-memory-overclocking.1774358/post-28662268

