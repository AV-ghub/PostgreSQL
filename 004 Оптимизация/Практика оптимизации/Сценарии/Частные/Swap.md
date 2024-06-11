## [Swappiness](https://www.kernel.org/doc/html/latest/admin-guide/sysctl/vm.html?highlight=swappiness#swappiness)
Defines relative IO cost of swapping and filesystem paging.   

Value between **0 and 200**.    

At **100**, the VM assumes **equal IO cost**    
**Lower** values signify **more expensive** swap IO, **higher** values indicates **cheaper**.   

The **default value is 60**.

Example
Swap IO 2x faster than filesystem IO, swappiness should be 
```
133 (x + 2x = 200, 2x = 133.33).
```

> At 0 not swap does not initiated.

### Check
```bash
# from file
$ cat /proc/sys/vm/swappiness
$ sudo sysctl -p

$ sysctl vm.swappiness
```
### Set
```bash
$ sudo sysctl vm.swappiness=0

# edit the file with the `nano` editor
$ sudo nano /etc/sysctl.conf

# my custom swappiness setting
vm.swappiness=0
```
> You can also clear your swap by running swapoff -a and then swapon -a as root instead of rebooting to achieve the same effect.
```
$ sudo swapoff -a && sudo swapon -a
```
> Применение параметра смотрим на Ubuntu через System Monitor -> Resources







