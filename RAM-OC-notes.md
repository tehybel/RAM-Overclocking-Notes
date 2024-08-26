

For now, this document is a mix of findings, tips, notes, and advice for overclocking memory - specifically DDR4 B-die memory on Intel platforms.

I hope to eventually turn it into a full-fledged guide. 

Until then, sections may be reordered and I'm still adding information, but feel free to skim through it and take whatever you find useful with you. If you found it useful, feel free to leave a Github Star, or link others here when they ask questions :-)



Basic Memory Overclocking
====================


Stability testing
-----
TODO write me


Overall process
-------------
- The following steps are based on the linked comment with [an order of which timings to optimize when](https://github.com/integralfx/MemTestHelper/issues/87#issuecomment-2119254780)
- turn PowerDown mode off before you start
- turn Memory Fast Boot off (or set it to Slow Training on MSI) while overclocking
- core/cache OC: get your processor core- and cache overclocks stable before you start on RAM. Alternatively, leave them until after you're done with RAM overclocking. Mixing them up gets very confusing fast.
- Memory Frequency CR2: see how far up in frequency your memory will go with Command Rate set to 2.
- Memory Frequency CR1: see how far up in frequency your memory will go on CR1
- choose CR1 or CR2 depending on results, prioritizing CR2 if it's 200+ MT/s higher than CR1, as a rule of thumb
- then do tRRDS/L + tFAW as it speeds up memory testing and is largely independent from all the other parameters
- tCL should go next as many of the following timings depend on it
- find good RTL/IOLs and lock them
- find the lowest possible tRCD/tRP
- lower tWR/tRTP together, keeping tWR as twice the value of tRTP
- lower tCWL; this goes after tCL since tCWL depends on it 
- after tCWL is tightened, lower tWRRD-sg/dg. Note that tCWL and tWRRD are inter-dependent: tightening one makes it harder to tighten the other.
- tRDRD, tWRWR
- tRFC, tREFI can be optimized here. Note that they are very temperature sensitive, so stress test your GPU while testing RAM to raise chassis temperature to a realistic one (unless you have your GPU/RAM watercooled)
- stop here if satisfied, or try advanced techniques documented below if you want to take your OC further

- TODO further document order of which advanced parameters to optimize, like RTLs-IOLs first, slopes near-last..



Recommended tools
-----------------

- [GSAT](https://github.com/stressapptest/stressapptest) is among the best tools for stability testing. You can run it on Windows via WSL.
- TestMem5 (TM5) with [these configs](https://github.com/integralfx/MemTestHelper/tree/oc-guide/TM5-Configs). TM5 can sometimes find errors in minutes that other tools take hours to discover. At other times TM5 finds nothing for an hour, and then VST errors after five seconds. Thus it's worth running as a supplement to other tools, to quickly find some types of errors.
- [y-cruncher](http://www.numberworld.org/y-cruncher/)
  - The VST, VT3 and FFT stress tests are excellent for testing stability.
  - Y-cruncher's Pi calculation can be timed as a good benchmark. It is sensitive to most memory changes, including loaded latency.
- [Intel Memory Latency Checker (MLC.exe)](https://www.intel.com/content/www/us/en/developer/articles/tool/intelr-memory-latency-checker.html) is good for measuring latency and bandwidth
- PyPrime is a good, latency-sensitive benchmark







More advanced topics
====================


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


RTLs and IOLs
-------
RTLs and IOLs are values that get set on each boot. You will want to find good values for them and set them to these fixed values inside the BIOS. This way, your board trains those RTLs and IOLs on every boot. This decreases variance and increases stability.

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



ODT RTTs
-------------

- Setting ODT RTTs (Wr, Nom, Park) correctly will *massively* impact how far you can take your OC
- Many cheap boards can actually OC surprisingly well once you set RTTs, they just set bad RTTs on Auto!
- RTTs can be found under the Skews menu in the Asus BIOS, and are also known as ODT Skews.


RTTs (Termination Resistances) are the values for Nom, Wr, Park that can be manually set in your BIOS for each memory channel. They strongly affect stability at higher frequencies. Setting correct values for them is crucial in order for your DIMM's ODT (On-Die Termination) to work well. This ensures signal stability even at high frequencies, letting you OC your sticks much further.

Recommended reading for setting RTTs: [overclock.net/threads/the-importance-of-skew-control-for-memory-overclocking.1774358/](The importance of skew control for memory overclocking)

Here's a quote by munternet, the author of that thread:

> I can see now that it would be easy to think you were unlucky in the silicon lottery or your IMC isn't any good just because your skews aren't set correctly


Background info on ODT RTTs
-----------------------------

- The unit for RTT is Ohms. The RTTs are resistances.
- Nom is short for Nominal[^polarfire]
- Wr is short for Write; it is the termination resistance used during a write operation
- Park is a default, or "parked" value when nothing else is going on[^samsung]

[^polarfire]: https://microchip.my.site.com/s/article/Use-of-the--ODT-Rtt-Nominal-Value--and--Dynamic-ODT--Rtt-WR---parameters-in-the-PolarFire-DDR-memory-controller
[^samsung]: https://download.semiconductor.samsung.com/resources/device-operation-timing-diagram/DDR4_Device_Operations_Rev11_Oct_14-0.pdf

See also:
- https://en.wikipedia.org/wiki/On-die_termination


Setting the RTTs in practice
-------------
- Optimal values can vary depending on other variables
  - IO voltage, RAM voltage, frequency, and CAS latency affect optimal ODTs. There are likely more variables too.

Optimal RTTs depend on your memory frequency, but the frequency you can reach also depends on having set at least "good enough" RTTs! This creates a problem when you first want to discover which frequency to take a new set of sticks to. Therefore, you may have to iterate a few times: raise frequency, improve ODT RTTs, then raise frequency a few hundred MHz further, then tweak the ODTs more, etc.

Each of the variables Wr, Nom and Park can only take specific values: 0, 34, 40, 48, 60, 80, 120, 240, 255. So you cannot set, for example, Wr=94. Only Wr=80 or Wr=120. (In MSI BIOS you can fine-tune ODT RTTs, though. This is likely not worth fiddling with until much later in the overclocking process.)

Which values should you use? If you haven't played with RTTs before, try a few of these settings as a starting point; they're in MSI format (wr-nom-park):
- 80-34-48
- 80-40-60
- 80-0-34
- 80-0-60

You can then pick one of Wr, Park, and Nom, and move it up/down by one, then see if your system got more stable. Once you have your starting point, make sure to only tweak one variable at a time or your results will get too confusing.

Other notes on RTTs
-------------
- when your RTLs and IOLs fail to train well on auto, it's usually because your RTTs are wrong (or you set a timing too tight)
- if your RTLs/IOLs only go very high/bad on one channel, but not the other, it's a strong hint that your RTTs for that channel are wrong!
  - for example, if your auto training often lands on 57-64-7-14, rather than around 57-57-7-7, then the high numbers are on Channel B, and you could try tweaking CHB ODT RTTs a bit up or down to see if that helps.
  - Tweak one of Park/Nom at a time, moving it one step up or down. Then do 3-5 reboots and see if your RTLs and IOLs on Auto become better.
  - If you just changed frequency and your skews no longer fit, it can be useful to switch one of ChA/ChB Park/Nom to Auto, then back again. Do all four parameters one at a time to identify which one is wrong. 
- Wr is usually fixed at 80. At very high frequencies (4500+) it can sometimes be better at 120.
- Asus and MSI use a different order for these, making it easy to get confused!
  - Asus uses the order Wr-Park-Nom
  - MSI uses the order Wr-Nom-Park
- when communicating with others, try to use explicit, unambiguous notation
  - for example, write: wr=80, park=0, nom=60. This is clearer than writing 80-0-60, whose meaning depends on the relevant board manufacturer
- you can get a quick feeling for approximate RTT values by setting your IOLs and RTLs all on auto and rebooting a few times, then seeing whether the RTLs/IOLs train well
- About scaling: as you increase demands on your memory controller (e.g. setting higher frequency, lower CAS latency) then the optimal values will change.
  - Wr usually remains static at 80; very high frequencies may benefit from wr=120
  - Nom drops slightly (e.g., 80-40-40 optimal at 4200C15, 80-40-34 optimal at 4300C15)
  - (Previously I found that Park drops towards 0 and Nom increases towards Wr, but I cannot replicate this, perhaps due to using different sticks?)
- it is very possible that memory channels A and B prefer slightly different RTTs.
  - for example, CHA might prefer 80-0-48 while CHB should be 80-0-60 for optimal stability.
  - Start by optimizing both channels together, changing values for both channels at once. Then finish the process by tweaking the RTTs slightly up and down one channel at a time, to look for better values.
- You cannot generally assume that Wr, Nom and Park are independent and optimize each one at a time
   - however on some boards the assumption may hold and be a useful shortcut



Approach for optimizing each variable
----------------------
- use GSAT; anything else I tried gave inconsistent results and muddy data
- you will need to temporarily cause instability. The means for doing this is crucial.
- always test inter-run variance before optimizing!
- TODO document this.
- custom OS installation that runs a test, saves data to storage, then auto reboots -- this is great for collecting data!
- On ASUS, disable Memory Scrambler while optimizing, then re-enable it once you want stability. Otherwise errors become inconsistent and it is hard to spot the patterns in your collected data



Misc findings
-----------
- ODT Write Duration and ODT Write Delay are independent of each other
  - This means you can optimize one of them first while the other is on auto, then lock your found value and search for the remaining part of the pair. This will discover the optimal values; there is no need to check every pair combined.
- The MSI Memory Try It! feature changes various settings behind the scenes, for example enabling or disabling some training algorithms that are set to Auto. Therefore it's worth testing different values of this feature, even if you override all the timings it sets for you.






Tweaking memory training algorithms
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


Links
---------------
- [Perfect's RAM OC Guide](https://docs.google.com/document/d/15qsrwUxGbKtqjeyEhmCear2kN9hCt2noKK5z14Webhg/edit?pli=1)
- [Integralfx's DDR4 OC guide](https://github.com/integralfx/MemTestHelper/blob/oc-guide/DDR4%20OC%20Guide.md)


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

