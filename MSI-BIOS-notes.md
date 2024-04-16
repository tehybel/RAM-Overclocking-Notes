Tips and tricks for MSI's BIOS
---------------------
- To find out the range of numbers a setting supports, you can put 99999 in and press enter. The BIOS will change the value to the highest supported right away.
- You can press the + and - keys to increase and decrease values by one. This also lets you find the lowest possible value, by holding down minus.
- You can press left/right to quickly jump downwards one category at a time
- The favorites feature is very handy and worth setting up. Press F2 anywhere to favorite a setting. Press F3 to enter favorites.
- Also, under F3, you can set things up to start at your preferred favorites page each time you enter the BIOS!
- You can force the BIOS to do a full poweroff reboot by quickly pressing F9 and then F8 to save your settings, then load them immediately. Anytime a profile is loaded this way, the BIOS will do a full cold start, rather than a soft reboot.
- During RAM OC you can put the RTLs and IOLs in your favorites menu to show their trained values right away after POSTing
- The Memory Fast Boot option is important during RAM overclocking
  - Set Fast Boot to Disabled to force the board to **initialize and train** memory anew on every POST. This is what you want during RAM OC
  - Set Fast Boot to Slow Training to force the board to **train** memory anew on every POST. However, memory initialization is skipped when possible. Source: [MSI manual](https://download.msi.com/archive/mnu_exe/mb/Intel500BIOS.pdf)
- 

Fixing when MOBO gets stuck during POST but keeps trying forever
-------------------------
- Sometimes during RAM OCing, your board gets stuck in the training process -- it tries to train repeatedly but gets stuck after perhaps 10 seconds, requiring a hard poweroff. The board has then 'forgotten' that this happened on the next boot and will try training again forever. In this situation you cannot get into the BIOS anymore. You would then normally use JBAT1 to reset the BIOS settings. However, there's an alternative way to get out of this looping that will keep your settings.
  - start with your PC powered fully off, then hit the power button
  - when the power button light comes on, count out loud, "one mississippi, two mississippi". Sounds silly, but it gets the timing right.
  - as soon as you are done counting, start holding down the power button until your machine shuts off fully
  - repeat the above steps one more time, then boot normally. You should get to the Overclock Failed! menu and be able to restore your settings from there.
  - this process works because it lets the POST process get to a point where the board knows it's tried booting, but hasn't gotten stuck yet. When this happens a few times, the motherboard will reset itself. 
- 

Bugs and workarounds
--------------------
- If your BIOS freezes or gets stuck when you are about to save your settings and reboot, disable Intel Turbo Boost Max Technology 3.0, which can be found under Advanced CPU Configuration in the OC menu.
- You cannot trust the search feature, as it does not look inside all sub-menus. For example, there is a feature called CPU VRM Over Temperature Protection under the DigitALL Power menu. Searching for VRM, however, gives zero results!
- If you cannot POST with a high System Agent Voltage (VCCSA) then try setting CPU Over Temperature Protection to a higher value, or to Auto. At 96C, I cannot POST above 1.17v VCCSA.
- Setting the Intel Adaptive Thermal Monitor to disabled might cause your system to no longer POST
- TODO document ring throttling
- Loading an overclocking profile from a USB drive may bug out in the selection menu if there are too many files in the same folder. As a workaround, move your old MsOcFiles into a sub-directory in your USB drive.
- If your system POSTs fine the first time, but hangs as soon as you try to reboot it with the top EZ debug LED lit, then try lowering VCCSA. Above 1.17v my system will only POST once.
