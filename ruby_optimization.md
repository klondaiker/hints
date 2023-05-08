[Performance Dashboard](https://docs.google.com/spreadsheets/d/1eNysCmQsca_F2U6O27WxiEBx0HhNS_Rxu-hVMxh_PGg/edit#gid=1564412399)

# Оптимизация
Целесообразность оптимизации: \
Экономия на железе должно быть больше времени работы программиста на оптимизацию

Порядок действии:
1) Определение метрики
2) Определение порога метрики
3) Написание теста производительности(rspec-benchmark)
4) Локализация главной точки роста
5) Устранение главной точки роста

# Оптимизация CPU
## Помощи от железа нет
1) Частота процессоров не растет
2) Паралеллизма нет (MRI/GIL)

GIL не обеспечивает потокобезопасноть (делает атомарными только некоторые операции (puts, Array#push)), \
но не дает интерпретатору разрушиться при нескольких тредах

**Поэтому оптимизируем сами!**

## Локализация и устранение главной точки роста CPU:
1) Использование benchmark инструментов для замера времени работы
2) Отключение GC при профилировании
3) Использование профилировщиков для локализации точки роста
4) Устранение точки роста путем оптимизации алгоритма медленного кода

## Benchmark инструменты:
1) [progressbar](https://github.com/jfelchner/ruby-progressbar)
2) [benchmark](https://ruby-doc.org/stdlib-2.5.3/libdoc/benchmark/rdoc/Benchmark.html)
3) [benchmark-ips](https://github.com/evanphx/benchmark-ips)

## Инструменты для профилирования
### Сэмплирующие профилировщики
a) берут текущее состояние стека несколько раз в секунду \
b) смотрят, что выполнялось в данный момент \
c) считают, какой метод какую долю сэмплов выполнялся \
d) не очень сильно замедляют программу

1) [rbspy](https://github.com/rbspy/rbspy) - подключение к текущему процессу
2) [stackprof](https://github.com/tmm1/stackprof)
  * может выдавать исходники медленных методов \
`stackprof stackprof.dump --method Object#select_session_from_user`
  * экспорт в flamegraph для [speedscope](https://www.speedscope.app)
### Трассирующие профилировщики
a) Полностью отслеживают все-все инструкции программы \
b) Работают более точно \
c) Сильно замедляют программу

[ruby-prof](https://github.com/ruby-prof/ruby-prof)
  * `RubyProf::FlatPrinter` - медленные методы
  * `RubyProf::GraphHtmlPrinter` - медленные части кода
  * `RubyProf::CallStackPrinter` - медленные пути исполнения
  * `RubyProf::CallTreePrinter`(для qcachegrind)- мощный визуальный инструмент

# Оптимизация Memory
## Слои работы с памятью
1) Ruby-код
2) Ruby runtime (VM, GC)
3) Allocator
4) OS
5) Actual Memory (RAM/SWAP)

## Garbage collector
1) Mark and sweep
2) Lazy sweep (1.9.3)
3) Generational (2.1) <-- большой буст в производительности Memory и CPU
4) Incremental (lazy mark) (2.2)
5) Compacting (2.7)
6) No parallel
7) No real-time

## Memory profilers
1) Запрос количества памяти у ОС
```ruby
"%d MB" % (`ps -o rss= -p #{Process.pid}`.to_i / 1024)
```
2) GC.stat - статистика сборщика мусора
3) ObjectSpace - позволяет проходить по всем живым текущим объектам
4) Гем [memory_profiler](https://github.com/SamSaffron/memory_profiler) - аллокация и память, один детальный отчет
5) Гем [stackprof](https://github.com/tmm1/stackprof) - только аллокация
6) Гем [ruby-prof](https://github.com/ruby-prof/ruby-prof) - аллокация и память
  * `RubyProf::ALLOCATIONS`
  * `RubyProf::MEMORY`
7) Valgrind Massif. Valgrind Visualizer

## Разбухание памяти (Bloat)
Ruby не может отдать память ОС после его разбухивания изза фрагментации памяти

## Утечка памяти (Leak)
1) Утечка объектов
2) Утечка памяти в C-extensions
3) Утечка в ruby VM

## Аллокаторы памяти
1) glibc - дефолт
2) jemalloc
```bash
gem install jemalloc
CONFIGURE_OPTS="--with-jemalloc" rbenv install 2.6.3
```
3) tcmalloc

# Оптимизация БД
## Инструменты мониторинга
1) PG Hero - находим главную точку роста в бд
2) New Relic (Gmonit) - помогает найти экшены и дает вид с точки зрения приложения
3) PG badger - анализ логов в определенный период времени (не-online, полезен в анализе инцидентов, отчеты)
4) pgadmin - дашбоард текущих процессов (кол-во сесии и тд)
5) graphana + prometheus postgresq_exporter - более круче pgadmin
6) Рельсовые логи

## Инструмены для разработки
1) postico - mac-клиент для подключения к postgresql
2) rails panel - dev-extension для хрома
3) rack-mini-profiler (спроектирован и для прода, используя авторизацию)

## Порядок действии оптимизации запроса
1) Находим больной запрос через pghero / NewRelic / RMP
2) Строим план EXPLAIN ANALYZE, визуализируем через explain.tensor.ru: \
Seq scan - плохо, Index-only scan - идеально
3) Смотрим куда уходит время
4) Пытаемся понять можно ли помочь индексом (многоколоночный? функциональный? частичный?)
5) Строим индекс и сравниваем результаты
6) https://use-the-index-luke.com/
P.S. Можно ли закешировать результаты ?

## Индексы
1) Btree index \
Сбалансированная структура данных, поиск за O(1) (изза роста в ширину, а не в глубину) \
Сильно ветвистая, глубина <= 5 для огромных таблиц (в каждой страницe сотни элиментов) \
Данные упорядочены, есть горизонтальные связи \
Дефолтный индекс в PG \
Может выдавать значения без обращения к таблице \
Поддерживает многоколоночность (порядок колонок очень важен)
2) Hash index \
Менее полезен и распространен, чем btree \
Не сохраняет порядок \
Не надежен при сбоях \
Интересен скорее ради примера
3) GIST index \
Обощенное дерево поиска \
Framework для создания индекса \
Обобщение Btree: вместо больше меньше(<>) - произвольные предикаты и функции согласованости \
Например: Rtree - индекс для точек на плоскости
4) GIN index \
Сами данные - составные, строки содержат элементы \
Индексируются элементы \
Элемент сылается на строки, где он встречается \
Пример: индексация массивов, индексация jsonb, полнотекстовый поиск

## EXPLAIN
1) cost=0.00..458 \
0.00 - Оценка стоимости до начала возврата первых результатов \
458 - оценка стоимости полного завершения выполнения узла
2) rows=100000 - оценочное количество строк в результате
3) width=244 - оценочный средний размер строки в байтах \
Стоимость измеряется в условных единицах \
Стоимость внешних узлов включает в себя стоимость внутренних \
EXPLAIN ANALYZE реально выполняет запрос и добавляет actual_time - время в ms, по каждому узлу плана \

## Тюнинг настроек постгреса
Вкладка Tune в PgHero \
Книга: https://postgresql.leopard.in.ua/

## Предотвращение проблем с БД
https://github.com/ankane/strong_migrations

## N + 1
https://github.com/flyerhzm/bullet \
https://github.com/nepalez/rspec-sqlimit

## VACUUM

# Оптимизация Backend
## Закон Литтла
I = L * T \
Instances, Load(req/sec), Response Time (sec/req) \

## Rules of thumb from Nate Berkopec
95th percentile  <= 4 * avg для каждого endpoint \
endpoint avg <= 4 * app_avg для каждого endpoint

## Сервисы мониторинга
Saas
1) Skylight.io
2) NewRelic (Gmonit)
3) Scout
4) Datadog

DIY
1) ELK (Elastic, logstash, Kibana)
2) Prometheus + graphana (gem prometheus_exporter, yabeda)

Можно и нужно настраивать алерты - эфективная обратная связь от системы мониторинга

## Local production
gem ankane/pgsync

## Load-test и Benchmark
Удобны для получения метрики и бюджета, к которому мы стремимся
1) siege
2) ab (apache framework)
3) wrk, wrk2

## Инструменты
1) Обязательно использовать local_production
2) rack-mini-profiler
3) ruby-prof в любом интересующем месте
4) rbspy к работающему app-серверу
