# Оптимизация
Целесообразность оптимизации: \
Экономия на железе > время работы программиста на оптимизацию

Порядок действии:
1) Определение метрики
2) Определение порога метрики
3) Написание теста производительности(rspec-benchmark)
4) Локализация точки роста
5) Устранение точки роста

# Оптимизация CPU
Помощи от железа нет:
1) Частота процессоров не растет
2) Паралеллизма нет (MRI/GIL)

GIL не обеспечивает потокобезопасноть (делает атомарными только некоторые операции (puts, Array#push)), \
но не дает интерпретатору разрушиться при нескольких тредах

**Поэтому оптимизируем самим!**

Локализация и устранение точки роста CPU:
1) Отключение GC при профилировании
2) Использование benchmark инструментов для замера времени работы и локализации точки роста
3) Использование профилировщиков для устранения точки роста

Benchmark инструменты:
1) [progressbar](https://github.com/jfelchner/ruby-progressbar)
2) [benchmark](https://ruby-doc.org/stdlib-2.5.3/libdoc/benchmark/rdoc/Benchmark.html)
3) [benchmark-ips](https://github.com/evanphx/benchmark-ips)

Инструменты для профилирования:
1) [rbspy](https://github.com/rbspy/rbspy) - подключение к текущему процессу
2) [ruby-prof](https://github.com/ruby-prof/ruby-prof) - RubyProf::FlatPrinter, RubyProf::GraphHtmlPrinter, RubyProf::CallStackPrinter
