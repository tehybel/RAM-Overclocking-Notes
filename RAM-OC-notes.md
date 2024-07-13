

For now, this document is a mix of findings, tips, notes, and advice.

I hope to eventually turn it into a full-fledged guide. 

Until then, the order of sections is a jumbled mess - but feel free to skim through it and take whatever you find useful with you.




About Memory Training
-------------------
Each time you boot your computer, it goes through a process known as memory training. During memory training, your system goes through a long lineup of algorithms to tune timings and parameters that are specific to your memory, motherboard, processor, and even the current heat level of your RAM sticks. This training must complete before your system can use its RAM. 

Thus memory training occurs in the time between when you press the power button and the time when you are able to enter your BIOS. If your motherboard has a [seven-segment display](https://i.ytimg.com/vi/bEjH775UeNg/maxresdefault.jpg) with numeric codes, you can follow along as it trains. Sadly only expensive motherboards have this feature nowadays.

You can find a list of memory training algorithms inside your BIOS's advanced settings, where you can enable or disable individual algorithms. 

<details>
  <summary>Example List of Memory Training Algorithms</summary>
<code>
Early Command Training 
SenseAmp Offset Training 
Early ReadMPR Timing Centering 2D 
Read MPR Training 
Receive Enable Training 
Jedec Write Leveling 
Early Write Time Centering 2D 
Early Write Drive Strength / Equalization
Early Read Time Centering 2D 
Write Timing Centering 1D 
Write Voltage Centering 1D 
Read Timing Centering 1D 
Dimm ODT Training* 
DIMM RON Training* 
Write Drive Strength/Equalization 2D* 
Write Slew Rate Training* 
Read ODT Training* 
Read Equalization Training* 
Read Amplifier Training* 
Write Timing Centering 2D 
Read Timing Centering 2D 
Command Voltage Centering 
Write Voltage Centering 2D 
Read Voltage Centering 2D 
Late Command Training 
Round Trip Latency 
Turn Around Timing Training 
Rank Margin Tool 
Memory Test 
DIMM SPD Alias Test 
Receive Enable Centering 1D 
Retrain Margin Check 
Write Drive Strength Up/Dn independently 
CMD Slew Rate Training 
CMD Drive Strength and Tx Equalization 
Cmd Normalization
</code>
</details>

After booting, if parameters were found that led to your RAM being stable and error-free, we say that your memory "trained well". Otherwise we call it a "bad training".

A common situation in RAM overclocking is that one boot, your system is rock solid and stable.. and then the next boot you get tons of memory errors and crashes. 

Knowing about memory training is useful because it can explain what's going on in this situation: your memory trained well the first time, and poorly the second. To stabilize the system, you will need to find settings that consistently train well.

If you enable "fast boot" under your BIOS, the old training will be saved and reused on each boot. You generally want to **turn fast boot off** when you are RAM overclocking to avoid confusion and ensure stability. This way you get to see many different trainings of your memory as you reboot, and you can confirm that all of them are stable.

Even with fast boot enabled, if your system crashes, it will re-train memory on the next boot. You can tell this is happening as the boot takes far longer than usual.



RTLs and IOLs
-------
RTLs and IOLs are values that get set on each boot. They affect stability. You will want to find good values for them and set them to these fixed values inside the BIOS. This way, your board trains those RTLs and IOLs on every boot. This decreases variance and increases stability.

Additionally, RTLs and IOLs, when left on auto, can sometimes provide a hint about whether your memory training was good or bad.

- If RTLs are more than two apart, or IOLs are more than one apart across channels, then it is likely a bad memory training.
  - Note that this is only a rule of thumb, to be used for quick sanity checks.
  - Some online discussions talk about this as an absolute rule, but I have seen plenty of exceptions
- format: `RTL CHA` - `RTL CHB` - `IOL CHA` - `IOL CHB`
- example: 56-57-6-7 is a comparatively "good" training, because the RTLs 56 and 57 are only 1 apart, and the IOLs 6 and 7 are only 1 apart as well.
- example: 56-58-6-8 is unlikely to be stable by our rule of thumb, because the IOLs 6 and 8 are two apart.
- TODO talk about locking them in, and difference across manufacturers.
- ODT RTTs are strongly related to how well your RTLs and IOLs train
- When RTLs and IOLs won't train consistently well, it's likely because your RTTs are suboptimal
- RTLs vary by tCL. Raising tCL by one will require raising all RTLs by two.
- Higher memory frequencies may require slightly higher RTLs. For example:
  - at 4200 MHz my board likes 56-56-7-7
  - at 4300 MHz, it prefers 56-57-6-7 for stability
  - at 4400 MHz, 57-57-7-7 seems ideal.



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
- it is very possible that memory channels A and B prefer slightly different RTTs.
  - for example, CHA might prefer 80-0-48 while CHB should be 80-0-60 for optimal stability.
  - Start by optimizing both channels together, changing values for both channels at once. Then finish the process by tweaking the RTTs slightly up and down one channel at a time, to look for better values.


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

As it can be difficult to tell who's posting BS when you're new to RAM OC, I've provided my own list of people whose advice I've had success following.[^1] In no particular order:
- OLDFATSHEEP
- 7empe
- Falkentyne
- Ichirou
- PhoenixMDA
- Noreng

[^1]: One trick for deciding who to listen to online: check that their claims are backed up by results. Find a screenshot of their best memory overclock. If they have achieved outstanding results, their words deserve attention.

[^2]: Be especially wary with those who sound overly confident, and those who lay out their knowledge as "universal rules". Rules often depend on your board manufacturer, IMC, Intel vs. AMD, chipset generation, cooling situation, voltages, and a whole lot of other factors.




Memory training algorithms
--------------------------

You can sometimes find stability gains by moving specific trainings from 'Auto' to enabled/disabled.

The algorithms "Early Command Training" and "Late Command Training" are inter-dependent. You usually want to ensure only one of them is enabled at a time. Around 3200-4000 MHz, having only Early training enabled gives the best results. Once you push your RAM past a certain point, around CL16-CL17 and/or 4200 MHz, then disabling Early training and enabling only Late training gives better results.

Turn Around Timing Training: unless you disable this, some of your secondary timings may surreptitiously get changed across boots. Your settings for secondaries will only be taken as guidelines. I recommend always disabling this.

The 2D Centering and/or Voltage Centering trainings have been useful for me, but only in four-DIMM setups.

The first algorithms up to around Jedec Write Leveling are usually required to be Enabled in order to boot.

The "Early" versions of trainings are linked with the normal versions. I think only variant needs to run. For example, on MSI, enabling "Early Write Time Centering 2D" will disable "Write Time Centering 2D", assuming it is on Auto. 

I recommend testing each one of these, as I have had stability gains with them, but not many of the others:
- Early/Late Command Training (see above)
- Round Trip Latency: **Enabled**
- Turn Around Timing Training: **Disabled**
- Memory Test Training: **Disabled**
- 


DLL Bandwidth setting
---------------------

On MSI z490, I found that the Auto setting was consistently far more stable than any manually set bandwidth setting! Perhaps on auto, the board uses some mechanism to find a finely tuned value, whereas choosing 0/1/2/.../7 simply corresponds to specific values found in a lookup table?

On Asus, I've found that this setting is worth optimizing for small stability gains. For me, the ideal value could be found in the interval between 0 and 2. This value outperformed the 'Auto' setting, unlike on MSI.

The optimal value likely depends on DRAM frequency.




Automation and tricks to save effort
----------------------------------
- TODO write me




Interesting quotes
------------------

> [warm restart] causes post error 55. To avoid it I need to set Maximus Tweak Mode to 1 (from 2) and avoid fixing tRDRD_sg_training, tRDRD_sg_runtime, tRDRD_dg_training, tRDRD_dg_training. These have to be on auto, even with having the same 7 - 4 values after training. Setting them to fixed 7 - 4 gives me always post code 55 no matter how ridiculous VCCSA I would apply -- 7empe


Gotchas in practice
-------------------
- On ASUS, disable Memory Scrambler while optimizing, then re-enable it once you want stability. Otherwise errors become inconsistent and it is hard to spot the patterns in your collected data
- 





Recommended tools
-----------------

- [GSAT](https://github.com/stressapptest/stressapptest) is among the best tools for stability testing. You can run it on Windows via WSL.
- TestMem5 (TM5) with [these configs](https://github.com/integralfx/MemTestHelper/tree/oc-guide/TM5-Configs). TM5 can sometimes find errors in minutes that other tools take hours to discover. At other times TM5 finds nothing for an hour, and then VST errors after five seconds. Thus it's worth running as a supplement to other tools, to quickly find some types of errors.
- [y-cruncher](http://www.numberworld.org/y-cruncher/)
  - The VST, VT3 and FFT stress tests are excellent for testing stability.
  - Y-cruncher's Pi calculation can be timed as a good benchmark. It is sensitive to most memory changes, including loaded latency.
- {Intel Memory Latency Checker (MLC.exe)}(https://www.intel.com/content/www/us/en/developer/articles/tool/intelr-memory-latency-checker.html) is good for measuring latency and bandwidth
- PyPrime is a good, latency-sensitive benchmark


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
- write about stability testing, its importance, which tools work well
- 

Disclaimers
-----------
- I have only tested this information on B-die
- I have only used Intel systems
- I have only looked at DDR4
- I have only look at Asus and MSI boards so far

