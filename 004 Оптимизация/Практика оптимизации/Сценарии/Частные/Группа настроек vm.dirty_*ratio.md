[Отвечает за порядок записи на диск](https://qna.habr.com/q/538906)   

**vm.dirty_background_bytes** - сколько байт можно занять под dirty pages до того, как эта память начнем автоматически синхронизироваться на диск.    
**vm.dirty_bytes** - сколько всего байт можно занять под dirty pages.   

<details><summary><h5>Пример</h5></summary>
  
Например у вас выставлен vm.dirty_background_bytes = 10, vm.dirty_bytes = 30, скорость записи на диск 2 байта в секунду, а скорость записи в ОЗУ 5 байт в секунду (все специально такое маленькое, что бы наглядно было понятно что происходит и не путаться в переводе значений из байт в мегабайты, байт в секунду в мегабиты в секунду и т.п.).
(Сделаем оговорку что dirty pages работает намного сложнее, чем описанно ниже. Он работает со страницами, а не с байтами, он имеет привязку к конкретным файлам и еще куча нюансов. Ниже очень упрощенный вариант объяснения, что бы получить примерное представление о том, что вообще происходит).

В Секунду 0 (начало примера) vm.dirty_background_bytes = 0, диск полностью простаивает и ничего не происходит (опустим момент что linux многопоточный и такая ситуация почти нереальна в реальной жизни).   

Вдруг какой-то процесс начинает писать что-то на диск, фактически он изменяет данные в ОЗУ и помечает страницу как dirty page, фактической записи на диск не происходит (если не происходит вызов fsync, но об этом ниже).   

То есть через секунду, в секунду 1 у нас расклад такой: vm.dirty_background_bytes = 5 (скорость записи в ОЗУ ведь равно 5, дальше будет предполагать что процесс всегда будет писать с этой скоростью, потому что например ему нужно многое записать и все данные к этому уже подготовленны), vm.dirty_bytes = 5 (vm.dirty_bytes - это доступный для dirty page объем памяти, vm.dirty_background_bytes - это часть от vm.dirty_bytes), скорость записи на диск = 0.   

Следующая, 2 секунда: vm.dirty_background_bytes = 10, vm.dirty_bytes = 10, скорость записи на диск = 0.   

Следующая, 3 секунда: vm.dirty_background_bytes = 15, vm.dirty_bytes = 15, скорость записи на диск = 2 (vm.dirty_background_bytes превысил установленные нами условные 10 байт и запускается процесс синхронизации, он выбирает на скорость 2 байта в секунду (как мы условились выше) данные и пишет их на диск) (тут можно спорить в какой именно момент запуститься процесс синхронизации dirty pages на диск, когда vm.dirty_background_bytes будет 9, 10 или 11, но в данном случае этот вопрос не принципиальный).   

Следующая, 4 секунда: vm.dirty_bytes = 18 (+5 записал процесс, -2 синхронизировал процесс синхронизации, итого 15+5-2=18). vm.dirty_background_bytes нас больше не интересует, потому что он синхронизатор уже запустился.   

5 секунда: vm.dirty_bytes = 18   
6 секунда: vm.dirty_bytes = 21   
7 секунда: vm.dirty_bytes = 24   
8 секунда: vm.dirty_bytes = 27   
9 секунда: vm.dirty_bytes = 30   
10 секунда: vm.dirty_bytes = 30    
(vm.dirty_bytes уперся в вышеусловленное значение в 30 байт, больше места под dirty page у нас нет, поскольку синхронизирует записывает по 2 байта/с то мы упираемся в 2 байта/с, пока процесс не запишет все свои данные).   

...
Энная секунда: (процесс записал все необходимые ему данные): vm.dirty_bytes = 28   
n+1: vm.dirty_bytes = 26 (и т.п.).

</details>

[Tune Linux Kernel Parameters For PostgreSQL Optimization](https://www.percona.com/blog/tune-linux-kernel-parameters-for-postgresql-optimization/)

<details><summary><h5>Kernel Parameters For PostgreSQL Optimization</h5></summary>

### vm.swappiness
vm.swappiness is another kernel parameter that can affect the performance of the database. This parameter is used to control the swappiness (swapping pages to and from swap memory into RAM) behavior on a Linux system. The value ranges from 0 to 100. It controls how much memory will be swapped or paged out. Zero means disable swap and 100 means aggressive swapping.

You may get good performance by setting lower values.

Setting a value of 0 in newer kernels may cause the OOM Killer (out of memory killer process in Linux) to kill the process. Therefore, you can be on the safe side and set the value to 1 if you want to minimize swapping. The default value on a Linux system is 60. A higher value causes the MMU (memory management unit) to utilize more swap space than RAM, whereas a lower value preserves more data/code in memory.

A smaller value is a good bet to improve performance in PostgreSQL.

### vm.overcommit_memory / vm.overcommit_ratio
Applications acquire memory and free that memory when it is no longer needed. But in some cases, an application acquires too much memory and does not release it.  This can invoke the OOM killer. Here are the possible values for vm.overcommit_memory parameter with a description for each:

Heuristic overcommit, Do it intelligently (default); based kernel heuristics
Allow overcommit anyway
Don’t over commit beyond the overcommit ratio.
Reference: https://www.kernel.org/doc/Documentation/vm/overcommit-accounting

**vm.overcommit_ratio** is the percentage of RAM that is available for overcommitment. A value of 50% on a system with 2 GB of RAM may commit up to 3 GB of RAM.

A value of 2 for vm.overcommit_memory yields better performance for PostgreSQL. This value maximizes RAM utilization by the server process without any significant risk of getting killed by the OOM killer process. An application will be able to overcommit, but only within the overcommit ratio, thus reducing the risk of having OOM killer kill the process. Hence a value to 2 gives better performance than the default 0 value. However, reliability can be improved by ensuring that memory beyond an allowable range is not overcommitted. It avoids the risk of the process being killed by OOM-killer.

On systems without swap, one may experience a problem when vm.overcommit_memory is 2.

https://www.postgresql.org/docs/current/static/kernel-resources.html#LINUX-MEMORY-OVERCOMMIT

### vm.dirty_background_ratio / vm.dirty_background_bytes
The vm.dirty_background_ratio is the percentage of memory filled with dirty pages that need to be flushed to disk. Flushing is done in the background. The value of this parameter ranges from 0 to 100; however, a value lower than 5 may not be effective and some kernels do not internally support it. The default value is 10 on most Linux systems. You can gain performance for write-intensive operations with a lower ratio, which means that Linux flushes dirty pages in the background.

You need to set a value of vm.dirty_background_bytes depending on your disk speed.

There are no “good” values for these two parameters since both depend on the hardware. However, setting vm.dirty_background_ratio to 5 and vm.dirty_background_bytes to 25% of your disk speed improves performance by up to ~25% in most cases.

### vm.dirty_ratio / dirty_bytes
This is the same as vm.dirty_background_ratio / dirty_background_bytes except that the flushing is done in the foreground, blocking the application. So vm.dirty_ratio should be higher than vm.dirty_background_ratio. This will ensure that background processes kick in before the foreground processes to avoid blocking the application, as much as possible. You can tune the difference between the two ratios depending on your disk IO load.
  
</details>

  
Dirty page позволяет нам "сгладить" небольший всплекс на запись, за счет того, что данные, которые нужно записать на диск будут временно храниться в памяти.    
Причем именно кратковременный всплекс, потому что на долговременном всплекске вы забиваете всю память, отведенную под dirty page и после этого скорость записи падает до скорости работы диска.   

Увеличивая размер vm.dirty_bytes вы с одной стороны увеличиваете размер пика, который получиться сгладить, с другой увеличиваете объем ОЗУ которое займете под этот пик и время, которое понадобиться на то, что бы разгрести эту ОЗУ.    

По дефолту в centos7 выставленны значения ***vm.dirty_background_ratio = 10 и vm.dirty_ratio = 30*** (то есть 30% от всей доступной памяти можно занять под dirty page, запускать синхронизацию если зянято более 10%), ***vm.dirty_expire_centisecs = 3000*** (сбрасывать на диск данные, которые находяться в dirty page больше 30 секунд вне зависимости от того, сколько в данный размер занято под dirty page) и ***vm.dirty_writeback_centisecs = 500*** (синхронизатор должен просыпаться каждые 5 секунд для обработки данных, подподающих под vm.dirty_expire_centisecs).
На ubuntu (16.04 и 18.04) vm.dirty_ratio = 20, все остальные параметры точно такие-же как у centos.   

Логика тут очень простая: под dirty page память не резервируется. Если она нужна и есть свободная память - она используется (максимум 30% или 20% от всей доступной). Причем доступная память, это не "всего установленно", это именно available память в выводе "free".   

То есть мы получаем "некую автоматическую балансировку" системы по данному параметру в зависимости от "Total RAM - Used RAM", чем доступной памяти больше, тем больше может быть dirty page. Мой опыт говорит что на моих нагрузках большинство серверов чувствуют себя отлично со значениями по умолчанию.
