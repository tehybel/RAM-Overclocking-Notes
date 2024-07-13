


RTLs and IOLs
-------
- TODO talk about memory training in general first
- If RTLs are more than two apart, or IOLs are more than one apart across channels, then it is likely a bad memory training.
  - Note that this is only a rule of thumb, to be used for quick sanity checks.
  - Some online discussions talk about this as an absolute rule, but I have seen plenty of exceptions
- format: (RTL CHA, RTL CHB, IOL CHA, IOL CHB)
- example: 56-57-6-7 is a comparatively "good" training, because the RTLs 56 and 57 are only 1 apart, and the IOLs 6 and 7 are only 1 apart as well.
- example: 56-58-6-8 is unlikely to be stable by our rule of thumb, because the IOLs 6 and 8 are two apart.
- TODO talk about locking them in, and difference across manufacturers.
- ODT RTTs are strongly related to how well your RTLs and IOLs train
- When RTLs and IOLs won't train consistently well, it's likely because your RTTs are suboptimal


About ODT RTTs
--------------
- Setting ODT RTTs (Wr, Nom, Park) correctly will *massively* impact how far you can take your OC
- These can be found under the Skews menu in the Asus BIOS, and are also known as ODT Skews.
- Optimal ODT skews can vary depending on other variables
  - IO voltage, RAM voltage, frequency, and CAS latency affect optimal ODTs. There are likely more variables too.
- when your RTLs and IOLs fail to train well on auto, it's usually because your ODTs are wrong (or you set a timing too tight) 
- Wr is usually fixed at 80. At very high frequencies (4500+) it can sometimes be better at 120.
- Asus and MSI use a different order for these, making it easy to get confused!
  - Asus uses the order Wr-Park-Nom
  - MSI uses the order Wr-Nom-Park
- when communicating with others, try to use explicit, unambiguous notation
  - for example, write: wr=80, park=0, nom=60. This is clearer than writing 80-0-60, whose meaning depends on the relevant board manufacturer
- you can get a quick feeling for approximate RTT values by setting your IOLs and RTLs all on auto and rebooting a few times, then seeing whether the RTLs/IOLs train well
- About scaling: as you increase demands on your memory controller (e.g. setting higher frequency, lower CAS latency) then the optimal values will change.
  - Park drops towards 0
  - Nom rises towards Wr
  - Wr usually remains static at 80
  - Very high frequencies may benefit from wr=120



Misc tips and tricks
------------------
- You can time how long memory training takes with a stopwatch and use it to gauge which settings are best
- 

Process for optimizing
----------------------
- use GSAT; anything else I tried gave inconsistent results and muddy data
- you will need to temporarily cause instability. The means for doing this is crucial.
- always test inter-run variance before optimizing!
- TODO document this.
- custom OS installation that runs a test, saves data to storage, then auto reboots -- this is great for collecting data!


Some findings
-----------
- ODT Write Duration and ODT Write Delay are independent of each other
  - This means you can optimize one of them first while the other is on auto, then lock your found value and search for the remaining part of the pair. This will discover the optimal values; there is no need to check every pair combined.

Overall process
-------------
- TODO document order of what to optimize, like RTLs-IOLs first, slopes near-last..


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



The difficulty of RAM overclocking research
----------------------

Memory training and variance across reboots
-----------------------------------
Imagine that you want to compare two different values for a setting to determine which is more stable. For example, you may ask yourself, "is setting an ODT RTT of 80-0-34 or 80-0-48 better on my system? Which one should I use for best results?"

Let's say you try to boot up both settings, one at a time, and run a stability test on each boot. You find that on the first boot, having set 80-0-34, the test fails after 20 minutes. On the second boot, with 80-0-48, it fails after 8 minutes. You may then conclude, "80-0-34 is better on my system than 80-0-48, because the stability test ran for 20 minutes -- far more than only 8 minutes." But then, the next day after a fresh boot, your system has destabilized and you're quickly getting memory errors!? What's going on?

The issue is that on each boot, your board trains its memory anew. This involves running a long list of algorithms to determine values for various hidden parameters and timings, many of which can hurt stability if the training goes wrong. As you increase your overclock, it gets more difficult for this memory training to consistently succeed. Additionally, the training results vary depending on DIMM temperature and a whole host of other, essentially random factors. 

Returning to our "80-0-34 vs 80-0-48" example, the problem came from drawing a conclusion based on only two data points. If you want to be sure which setting is best, you will need to do multiple reboots, run stability test(s) each time, and gather data in a spreadsheet. 

Analyzing your data, then, is no longer a simple comparison between two numbers - it becomes a matter of statistics. Since data needs statistical analysis, there are no longer any clear, black-and-white answers. All you get is a probability that one value is better than the other.


Online sources of information
----------

Due to the variability of memory training across boots, it is easy to draw the wrong conclusions. To be fully certain, you need to gather data via long stability tests across multiple reboots. Only then do you have a reasonable level of certainty that your conclusion is correct -- at least for your particular system.

For this reason, high-level RAM overclocking demands a level of rigor and meticulousness that most people frankly just don't have the patience for. 

Therefore, there is **a lot** of conflicting and just plain **wrong** information about RAM overclocking online. I doubt most people set out to mislead or cause confusion; people share information with an intention to be helpful. It's just that finding the truth requires so much careful effort, and it's so easy to misstep.

When I started learning, I did not realize any of this. I would collect and read through all the material I could find online. I assumed everyone knew more than me. However, it turned out that a lot of it was wrong, or at least did not hold true on my personal system. It made me feel confused and uncertain, cost me a lot of time, and it held back my progress.

Therefore I would strongly recommend keeping a small list of people who definitely know what they are doing, and ignoring advice from anyone else.[^2]

As it can be difficult to tell who's posting BS when you're new to RAM OC, I've provided my own list of people whose advice I've had success following:[^1]
- OLDFATSHEEP
- 7empe
- Falkentyne
- Ichirou
- PhoenixMDA
- Noreng


Interesting quotes
------------------

> [warm restart] causes post error 55. To avoid it I need to set Maximus Tweak Mode to 1 (from 2) and avoid fixing tRDRD_sg_training, tRDRD_sg_runtime, tRDRD_dg_training, tRDRD_dg_training. These have to be on auto, even with having the same 7 - 4 values after training. Setting them to fixed 7 - 4 gives me always post code 55 no matter how ridiculous VCCSA I would apply -- 7empe


Gotchas in practice
-------------------
- On ASUS, disable Memory Scrambler while optimizing, then re-enable it once you want stability. Otherwise errors become inconsistent and it is hard to spot the patterns in your collected data
- 





Recommended tools
-----------------

- [GSAT](https://github.com/stressapptest/stressapptest)
- TestMem5 (TM5) with [these configs](https://github.com/integralfx/MemTestHelper/tree/oc-guide/TM5-Configs)
- [y-cruncher](http://www.numberworld.org/y-cruncher/), especially the VST and VT3 stress tests



Useful links
------------
https://www.overclock.net/threads/the-importance-of-skew-control-for-memory-overclocking.1774358/post-28662268


Misc Findings
--------
- The MSI Memory Try It! feature changes various settings behind the scenes, for example enabling or disabling some training algorithms that are set to Auto. Therefore it's worth testing different values of this feature, even if you override all the timings it sets for you.

TODO and reminders
---------
- TODO idea: put stars out of five, indicating difficulty/advancedness, and potential gains from tweaking
- talk about cold vs. warm boot, full powerdown with power supply off, boot stability
- 

Disclaimers
-----------
- I have only tested this information on B-die
- I have only used Intel systems
- I have only looked at DDR4
- I have only look at Asus and MSI boards so far

Footnotes
------------
[^1]: One trick for deciding who to listen to online: check that their claims are backed up by results. Find a screenshot of their best memory overclock. If they have achieved outstanding results, their words deserve attention.

[^2]: Be especially wary with those who sound overly confident, and those who lay out their knowledge as "universal rules": Rules often depend on your board manufacturer, IMC, Intel vs. AMD, chipset generation, cooling situation, voltages, and a whole lot of other factors.
