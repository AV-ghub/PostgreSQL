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

* [man](https://www.postgresql.org/docs/current/pgbench.html) [pro](https://postgrespro.ru/docs/postgresql/16/pgbench) [recap](https://github.com/AV-ghub/PostgreSQL/blob/main/004%20%D0%9E%D0%BF%D1%82%D0%B8%D0%BC%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D1%8F/%D0%9F%D1%80%D0%B0%D0%BA%D1%82%D0%B8%D0%BA%D0%B0%20%D0%BE%D0%BF%D1%82%D0%B8%D0%BC%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D0%B8/%D0%9C%D0%BE%D0%BD%D0%B8%D1%82%D0%BE%D1%80%D0%B8%D0%BD%D0%B3%2C%20%D0%BF%D1%80%D0%BE%D1%84%D0%B8%D0%BB%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5%20%D0%B8%20%D0%BB%D0%BE%D0%B3%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5/pgbench.md)   
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















