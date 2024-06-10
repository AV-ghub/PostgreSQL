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
