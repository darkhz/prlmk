# prlmk

A custom low-memory-killer for Android, based on the process reclaim driver.
Primarily suitable for devices with 4GB ram or less.

# usage

 - Disable all other low-memory-killers, if previously enabled, in the defconfig, for example:
> CONFIG_ANDROID_LOW_MEMORY_KILLER=n

> CONFIG_ANDROID_SIMPLE_LMK=n

 - Enable the following in the defconfig:
 > CONFIG_PRLMK=y
 
- Make sure ZRAM/SWAP is enabled, otherwise you may encounter serious problems when using the driver. 
  A ZRAM disksize/SWAP size of 1GB or higher is preferred.

- Apply all the process_reclaim and prlmk commits from the branch that correlates to your kernel version and compile.

# objective

- Keep important tasks like Android services running
  until they are closed by the user.

- Kill apps as little as possible, or specifically, killing apps based
  on the total time spent on it. This means that, apps with larger usage
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

- If the number of active file pages go below `free_file_limit`, it means that a
  memory critical situtation has occured, and tasks will be killed more aggressively.

  I specifically sorted according to `acct_timexpd`, see kernel/tsacct.c for
  more details (CONFIG_TASK_XACCT).
  
# additional info

The default settings in this driver are already suitable for 3GB and 4GB RAM devices, there is no need to extensively tune it.

Should you feel the need to tune it, however, enable the DEBUG option in the code, like this:
> #define DEBUG 1

and compile and boot the kernel. Try to stress memory as much as possible, by loading lots of applications and/or using a memory stress-tester tool, and carefully analyze dmesg and the log messages that follow. This will help to fine-tune free_file_limit and free_swap_limit.

# credits

- Minchan Kim and Vinayak Menon for the process_reclaim driver
- Sultan AlSawaf (kerneltoast) for mm-related commits and some useful codebits from his excellent Simple LMK.
https://github.com/kerneltoast/simple_lmk
- Diab Neiroukh (lazerl0rd) for his suggestions and advice to make the code and commits simple and neat.
https://github.com/lazerl0rd/
