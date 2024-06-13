[Settling the Myth of Transparent HugePages for Databases](https://www.percona.com/blog/settling-the-myth-of-transparent-hugepages-for-databases/)   

### Conclusion
I attained these results by running different benchmarking tools and evaluating different OLTP benchmarking standards.    
The results clearly indicate that for these workloads, ***THP has a negative impact on the overall database performance***.    
THP may be beneficial for various applications, but it certainly ***doesn’t give any performance gains when handling an OLTP workload***.

We can safely say that the “myth” is derived from experience and that the rumors are true.
