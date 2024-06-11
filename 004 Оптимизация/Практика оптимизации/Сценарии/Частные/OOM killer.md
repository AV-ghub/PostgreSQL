## Настройка OOM killer
[Чрезмерное выделение памяти в Linux](https://postgrespro.ru/docs/postgresql/16/kernel-resources#LINUX-MEMORY-OVERCOMMIT)   
[Linux Memory Overcommit](https://www.postgresql.org/docs/current/kernel-resources.html#LINUX-MEMORY-OVERCOMMIT)   
[Respite from the OOM killer](https://lwn.net/Articles/104179/)   
[overcommit handling modes](https://www.kernel.org/doc/Documentation/vm/overcommit-accounting)   
[Starting the Database Server](https://www.postgresql.org/docs/current/server-start.html)   

Включаем режим [строгого выделения памяти](https://www.kernel.org/doc/Documentation/vm/overcommit-accounting)
```bash
sysctl -w vm.overcommit_memory=2
```
Либо напрямую поместив соответствующую запись в /etc/sysctl.conf.

[Настраиваем overcommit_memory](https://serverfault.com/questions/606185/how-does-vm-overcommit-memory-work)
> When **vm.overcommit_memory = 0**, the vm.overcommit_ratio value is irrelevant.   
> The kernel will use a heuristic algorithm to overcommit memory.   
> When **vm.overcommit_memory 2**, the vm.overcommit_ratio value becomes relevant.   
> By **default, this value is set to 50**, which means the system would only allocate up to 50% of your RAM (plus swap).   
> When set it to 300, you are allowing the system to allocate up to 300% of your RAM (plus swap, if any),   
> which is why the **CommitLimit value in /proc/meminfo** is so high.   
> Although vm.overcommit_memory = 2 is usually used to prevent overcommitment, here, you're using it to cap the amount that can be overcommitted.   
> Setting it to 300 is dangerous as your system don't have 5171884 kB of memory, and so, **depending on how much swap space you have**,   
> the system will use swap (which is slow), or will run out of memory altogether.

Вместе с изменением vm.overcommit_memory исключаем процесс postmaster из числа возможных жертв при нехватке памяти.   
Для этого задаем приоритет OOM для этого процесса значение -1000
```bash
echo -1000 > /proc/self/oom_score_adj
```   
в скрипте запуска PostgreSQL непосредственно перед тем, как запускать postgres.    
Делать это нужно под root, в стартовом скрипте root.    
Также в скрипте должны быть установлены переменные окружения
```bash
export PG_OOM_ADJUST_FILE=/proc/self/oom_score_adj
export PG_OOM_ADJUST_VALUE=0
```   
С этими параметрами дочерние процессы будут запускаться с обычным, нулевым приоритетом OOM, чтобы при необходимости OOM смог уничтожать их.    
PG_OOM_ADJUST_VALUE также можно опустить, в этом случае подразумевается нулевое значение.    
Если не установить PG_OOM_ADJUST_FILE, дочерние процессы будут работать с тем же приоритетом OOM, что у главного процесса, что неразумно,    
так всё это делается как раз для того, чтобы только главный процесс оказался в привилегированном положении.





