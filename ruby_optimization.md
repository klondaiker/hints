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
Ruby не может отдать память ОС после разбухивания изза фрагментации памяти

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
