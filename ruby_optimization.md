[Performance Dashboard](https://docs.google.com/spreadsheets/d/1eNysCmQsca_F2U6O27WxiEBx0HhNS_Rxu-hVMxh_PGg/edit#gid=1564412399)

# Оптимизация
Целесообразность оптимизации: \
Экономия на железе > время работы программиста на оптимизацию

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
1) Отключение GC при профилировании (но не в benchmark)
2) Использование benchmark инструментов для замера времени работы и локализации точки роста
3) Использование профилировщиков для устранения точки роста

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
  * `RubyProf::CallTrueePrinter`(для qcachegrind)- мощный визуальный инструмент

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
