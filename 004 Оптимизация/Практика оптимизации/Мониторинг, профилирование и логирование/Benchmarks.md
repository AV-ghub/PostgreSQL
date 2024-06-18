<details><summary><h5>Вводные положения</h5></summary>

##### Требования к бенчмарку
* открытие транзакции на каждый запрос
* открытие сессии на каждый запрос
* сетевые задержки

##### Факторы, сильно влияющие на результат теста
* объём данных
* характеристики инстанса
* особенности файловых систем
* прогрев данных

##### Профиль нагрузки
Классика 90% чтение 9% insert и 1% update

</details>

<details><summary><h5>Классические варианты бенчмарков для PostgreSQL</h5></summary>

* [pgbench](https://www.postgresql.org/docs/current/pgbench.html)
* [--Яндекс.Танк]()
* [--Apache Benchmark]()
* [--JMeter]()
* [--sysbench]()
* [--netperf]()

</details>

<details><summary><h5>Классические утилиты анализа производительности</h5></summary>

* [--top]()
* [--atop]()
* [--htop]()
* [--perf-top]()

</details>

<details><summary><h5>Что измерять?</h5></summary>

* TPS
* QPS
* Latency
* нагрузку на диск/CPU/сеть

</details>















