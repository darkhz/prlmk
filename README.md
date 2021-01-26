# prlmk

A custom low-memory-killer for Android, based on the process reclaim driver.
Primarily suitable for devices with 4GB ram or less.

# usage

 - Disable all other low-memory-killers, if previously enabled, in the defconfig, for example:
> CONFIG_ANDROID_LOW_MEMORY_KILLER=n

> CONFIG_ANDROID_SIMPLE_LMK=n

 Do not disable the oom killer.
 
 - Enable the following in the defconfig:
 > CONFIG_PROCESS_RECLAIM=y
 
 > CONFIG_ANDROID_PR_KILL=y
 
- Make sure ZRAM/SWAP is enabled, otherwise you may encounter serious problems when using the driver.A ZRAM disksize/SWAP size of 1GB or higher is preferred.

- Apply all the process_reclaim commits from the branch that correlates to your kernel version and compile.

# objective

- Keep important tasks like Android services running
  until they are closed by the user.

- Kill apps as little as possible, or specifically,killing apps based
  on the total time spent on it.This means that, apps with larger usage
  time get killed more rarely, and apps with lesser usage times get
  killed frequently.

# approach

- We keep reclaiming pages if they are above defined threshold
  (`free_swap_limit`).

- Once the swap goes below `free_swap_limit`, we start to collect tasks to kill.

- We then sort those tasks based on their last accessed `stime+utime`,
  and then start killing the tasks with the lowest `stime+utime`.We stop
  killing tasks once the number of swap pages are above the threshold
  (`free_swap_limit`).

  I specifically sorted according to `acct_timexpd`,see kernel/tsacct.c for
  more details(CONFIG_TASK_XACCT).
  
# additional info

The default settings in this driver are already suitable for 3GB and 4GB RAM devices, there is no need to extensively tune it.

Should you feel the need to tune it, however, pay careful attention to the free_file_limit
tunable, since this determines at what ACTIVE file page size (in KB), the driver will see the memory situation in the device as critical.

To find out what free_file_limit value is suitable for your device:

- Set free_file_limit to 0.
- You will have to create a situation where memory pressure is high and filepages, especially active filepages are continuously depleting, which you can do by launching heavy apps and switching between them constantly.
- While doing the above step, keep an eye on /proc/meminfo, especially at the `Active(file)` field.You can launch an adb instance, and execute `watch -n1 cat /proc/meminfo`, and redirect it to a file for reference.
- Under severe memory pressure, the device will freeze.When this happens, look at the `Active(file)` field at the exact time of the freeze. This will be your free_file_limit value. Increase the value a little bit just to be on the safe side.

This process, I will acknowledge, is indeed tedious, and I plan to improve debugging it in the future. If this is too difficult for you to debug, you can stick to the defaults, or switch to another low-memory-killer, like slmk for example.


# todo

- Cleanup the code. It is messy as of now. A rewrite may be required.
- Simplify the tunables and tunable names.
- Make it easier to debug.

# credits

- Minchan Kim and Vinayak Menon for the process_reclaim driver
- Sultan AlSawaf (kerneltoast) for mm-related commits and some useful codebits from his excellent Simple LMK.
https://github.com/kerneltoast/simple_lmk
- Diab Neiroukh (lazerl0rd) for his suggestions and advice to make the code and commits simple and neat.
https://github.com/lazerl0rd/
