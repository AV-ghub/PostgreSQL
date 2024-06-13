[Benchmark PostgreSQL With Linux HugePages](https://www.percona.com/blog/benchmark-postgresql-with-linux-hugepages/)   
[Tune Linux Kernel Parameters For PostgreSQL Optimization](https://www.percona.com/blog/tune-linux-kernel-parameters-for-postgresql-optimization/)

## Benchmark Machine
> Supermicro server:  
> Intel(R) Xeon(R) CPU E5-2683 v3 @ 2.00GHz  
> 2 sockets / 28 cores / 56 threads  
> Memory: 256GB of RAM  
> Storage: SAMSUNG  SM863 1.9TB Enterprise SSD  
> Filesystem: ext4/xfs  
> OS: Ubuntu 16.04.4, kernel 4.13.0-36-generic  
> PostgreSQL: version 11  

## Linux Kernel Settings
**Transparent HugePages** are by default enabled, and allocate a page size that may not be recommended for database usage.   
For databases generally, fixed sized HugePages are needed, which Transparent HugePages do not provide.   
Hence, disabling this feature and defaulting to classic HugePages is always recommended.

## PostgreSQL Settings
### Postgresql.conf
```sql
shared_buffers = '64GB'
work_mem = '1GB'
random_page_cost = '1' 
maintenance_work_mem = '2GB'
synchronous_commit = 'on'
seq_page_cost = '1' 
max_wal_size = '100GB'
checkpoint_timeout = '10min'
synchronous_commit = 'on'
checkpoint_completion_target = '0.9'
autovacuum_vacuum_scale_factor = '0.4'
effective_cache_size = '200GB'
min_wal_size = '1GB'
wal_compression = 'ON'
```
### HugePages config
```bash
AnonHugePages:       0 kB
ShmemHugePages:      0 kB
HugePages_Total:     100
HugePages_Free:      97
HugePages_Rsvd:      63
HugePages_Surp:      0
Hugepagesize:        1048576 kB
```

[Huge Pages and Transparent Huge Pages](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/6/html/performance_tuning_guide/s-memory-transhuge#s-memory-configure_hugepages)
<details><summary><h5>Huge Pages and Transparent Huge Pages</h5></summary>
  
There are two ways to enable the system to manage large amounts of memory:
- Increase the number of page table entries in the hardware memory management unit
- Increase the page size
  
**Modern processor only supports hundreds or thousands of page table entries.**

Additionally, hardware and memory management algorithms that ***work well with thousands of pages (megabytes of memory)   
may have difficulty performing well with millions (or even billions) of pages***.   
This results in performance issues: when an application needs to use more memory pages than the memory management unit supports,   
the system ***falls back to slower, software-based memory management***, which causes the entire system to run more slowly.

</details>

[Huge pages in Linux](https://kerneltalks.com/services/what-is-huge-pages-in-linux/)

<details><summary><h5>Huge pages in Linux</h5></summary>

During system boot, you reserve your memory portion with huge pages for your application.    
This memory portion i.e. these memory occupied by huge pages is ***never swapped out of memory***.    
It will stick there ***until you change your configuration***.    
This increases application performance to a great extent like Oracle database with pretty large memory requirements.

In virtual memory management, the kernel maintains a table in which it has a ***mapping of the virtual memory address to a physical address***.    
For every page transaction, the kernel needs to load related mapping.    
If you have small size pages then you need to load more numbers of pages resulting kernel to load more mapping tables.    
This decreases performance.

### configure huge pages
Run below command to check current huge pages details
```bash
root@kerneltalks # grep Huge /proc/meminfo
AnonHugePages:         0 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
```

Run below script to get how much huge pages your system needs currently. The script is from Oracle and can be found.
```bash
#!/bin/bash
#
# hugepages_settings.sh
#
# Linux bash script to compute values for the
# recommended HugePages/HugeTLB configuration
#
# Note: This script does calculation for all shared memory
# segments available when the script is run, no matter it
# is an Oracle RDBMS shared memory segment or not.
# Check for the kernel version
KERN=`uname -r | awk -F. '{ printf("%d.%d\n",$1,$2); }'`
# Find out the HugePage size
HPG_SZ=`grep Hugepagesize /proc/meminfo | awk {'print $2'}`
# Start from 1 pages to be on the safe side and guarantee 1 free HugePage
NUM_PG=1
# Cumulative number of pages required to handle the running shared memory segments
for SEG_BYTES in `ipcs -m | awk {'print $5'} | grep "[0-9][0-9]*"`
do
   MIN_PG=`echo "$SEG_BYTES/($HPG_SZ*1024)" | bc -q`
   if [ $MIN_PG -gt 0 ]; then
      NUM_PG=`echo "$NUM_PG+$MIN_PG+1" | bc -q`
   fi
done
# Finish with results
case $KERN in
   '2.4') HUGETLB_POOL=`echo "$NUM_PG*$HPG_SZ/1024" | bc -q`;
          echo "Recommended setting: vm.hugetlb_pool = $HUGETLB_POOL" ;;
   '2.6' | '3.8' | '3.10' | '4.1' | '6.5' ) echo "Recommended setting: vm.nr_hugepages = $NUM_PG" ;;
    *) echo "Unrecognized kernel version $KERN. Exiting." ;;
esac
# End
```
You can save it in /tmp as hugepages_settings.sh and then run it like below :
```bash
# sh /tmp/hugepages_settings.sh
```

### Configure hugepages in kernel
Now last part is to configure the above-stated kernel parameter and reload it. Add below value in /etc/sysctl.conf and reload configuration by issuing sysctl -p command.
```bash
vm.nr_hugepages=126
```
Now, huge pages have been configured in the kernel but to allow your application to use them you need to increase memory limits as well. The new memory limit should be 126 pages x 2 MB each = 252 MB i.e. 258048 KB.

You need to edit below settings in ***/etc/security/limits.conf***
```bash
soft memlock 258048 
hard memlock 258048
```
You might to restart your application to make use of these new huge pages.

### How to disable hugepages
To check the current state of huge pages.
```bash
root@kerneltalks # cat /sys/kernel/mm/transparent_hugepage/enabled
[always] madvise never
```
[always] flag in output shows that hugepages are enabled on system.

> For RedHat based systems file path is /sys/kernel/mm/redhat_transparent_hugepage/enabled

If you want to disable huge pages then add ***transparent_hugepage=never*** at the end of kernel line in ***/etc/grub.conf*** and reboot the system.

</details>

Clearly you can see that the ***performance gain with HugePages increases as the number of clients and the database size increases***, as long as the size remains within the pre-allocated shared buffer.   
With HugePages set to 1GB, the higher the number of clients, the higher the comparative performance gain.   
The key observation here is that the ***performance with 1GB*** HugePages improves as the number of clients increases and it eventually ***gives better performance than 2MB HugePages or the standard 4KB page size***.   
As expected, ***when the database spills over the pre-allocated HugePages, the performance degrades significantly***.   

### Summary
One of my key recommendations is that we must ***keep Transparent HugePages off***.    
You will see the ***biggest performance gains when the database fits into the shared buffer with HugePages enabled***.    
Deciding on the size of huge page to use requires a bit of trial and error, but this can potentially lead to a significant TPS gain where the database size is large but remains small enough to fit in the shared buffer.














