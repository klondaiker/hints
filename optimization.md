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
Ruby не может отдать память ОС после его разбухивания изза фрагментации памяти (gem oink)

puma-worker-killer

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
Отличие дева от прода
1) Данные в бд (gem ankane/pgsync)
2) Сборка фронта
3) Перезагрузка кода
4) Логирование
5) HTTPS
6) Внешние сервисы

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
5) Middleware: Rack::RubyProf, свой кастомный.

## Как сделать быстрее
1) Не делать (а это точно нужно? продуктовое решение)
2) Пусть сделает кто-то другой (переложить работу на СУБД и тд)
3) Сделать и не переделывать (кеширование, мемоизация)
4) Сделать потом (вынести в фоновую задачу)
5) Сделать сейчас, но эффективнее (оптимизация)
6) Не проставивать во время ожидания (concurrency)
7) Параллелизм (процессорный)

## Виды кеширования
1) Fragment caching (cache от updated_at, touch)
2) Page caching - закешировать страницу целиком, сохранить в файл и отдавать nginx \
Не может в динамику (никакой авторизации) \
https://github.com/rails/actionpack-page_caching
3) Action caching - выполняются before-фильтры, может в авторизацию \
https://github.com/rails/actionpack-action_caching
4) Low-level caching (Rails.cache.fetch)
5) Custom caching (мемоизация)
6) HTTP caching (304 - stale?)

## Хорошие фоновые задачи
1) Идемпотентность (можно запускать дважды)
2) Код Job-а чем меньше тем лучше
3) Параметры Job-а чем проще тем лучше
4) Время исполнения Job-а чем меньше тем лучше
5) Формировать очереди, учитывая время работы и интенсивность потока задач
6) Можно проверять уникальность задачи перед постановкой
7) Ошибки из Job-ов всегда слать в сервис ошибок!
8) Timout, retry
9) Избегать memory bloat

# Оптимизация веба

## Мотивация
Медленный сайт теряет деньги \
Быстрые сайты зарабатывают больше
1) Вовлеченность
2) Удержание
3) Конверсия

## Как измерять
1) Не так как вам удобно, а так как это видит большая часть пользователей
2) Географически ближе к вашим юзерам
3) Технически ближе к вашим юзерам (Браузер, устройство)
4) Скорость доступа (3g и тд)

## Чем измерять
Saas
1) Google analytics, Яндекс аналитика
2) NewRelic \
DIY \
gem ahoy, yabeda + Grafana, Kibana

## Цели
1) Сравнить себя с конкурентами (должны быть на 20% быстрее)
2) Browser calories (браузер-расширения) (deprecated)
3) Вовлекать всю компанию
4) Меньше 300мс, хорошо - 100мс
5) 5с на TTI на телефонах с 3G
6) SpeedIndex < 1250
7) Critical file suze budget 170kb gzipped (400kb)
8) https://www.performancebudget.io/

Google tag manager - зло!

## Latency and bandwidth
Latency - время передачи пакета от источника до получателя (нужен чтобы быстро открывались сайты, состоящие из сотен ресурсов)
Bandwidth - максимальное кол-во информации, которое можно передать (нужен больше для стриминга/скачивания видео)

Как улучшить Latency? 
1) CDN
2) Не делать лишних roundtrip-ов

## TCP
1) Обновить Linux
2) Прочитать HBPN
3) CDN
4) http://bagder.github.io/I-D/httpbis-tcp/

## HTTP
1) HTTP COOKIES BLOAT
2) Переход на HTTP/2 (все ускорится в 2 раза)
`listen 443 ssl http2;`

## TLS
SSL (Netscape) - коммерческое название

Обеспечивает:
1) Encryption
2) Authenication
3) Integrity

1) Идентификация - показать паспорт
2) Аутентификация - проверить его подлинность
3) Авторизация - разрешить зайти в бизнес центр, если заказан пропуск на этот паспорт

Checklist
1) Проверить утилитой для теста
2) Проверить сертификаты
3) Обязательно настроить установку TLS-соединения за 1 Roundtrip
4) Terminate TLS closer to the user
5) Настроить HSTS

## Аssets
1) CDN для asset-ов (config.action_controller.asset_host = "assets%d.example.com", %d - обход ограничения параллелизма)
2) Инлайнинг дает больше минусов, чем плюсов

# Оптимизация для браузера

## Полный чеклист от Виталия Фридмана
https://www.smashingmagazine.com/2021/01/front-end-performance-2021-free-pdf-checklist/

##  Инструменты
1) WebPageTest \
https://www.webpagetest.org/ (https://www.webpagetest.org/result/230509_BiDcAS_B86/) \
Можно настройть свой webpagetest для прогонов! \
Строим таблицу прогонов и обновляем ее
2) LightHouse (Google Chrome dev tools)
3) PWMetrics
4) PageSpeed Insight

## Мониторинг
Saas
1) NewRelic Browser (Gmonit ?)
2) CRUX
Google собирает RUM данные с Chrome \
Можно получить к ним доступ через BigQuery Data Warehouse
3) LightHouse-keeper
4) Speedcurve
5) Calibre

DIY \
Sitespeed.io (и может собирать данные из всех перечисленных Saas-данных

## CI
1) Sitespeed.io \
https://github.com/spajic/performance-ci-demo/blob/master/.travis.yml
2) LightHouse CI
https://github.com/GoogleChrome/lighthouse-ci

## Оптимизация объема Javascript
JS может заблокировать main-thread \
Webpack-bundle-analyzer \
Уменьшение бандла в 10 раз уменьшает время отклика где-то в 4 раза \
и увеличивает количество мобильных пользователей

## SW.js (service-worker)
1) Перехватывает все сетевые запросы как proxy
2) Запускается в фоновом режиме без блокировки main thread
3) Можно использовать для кеширования

## Оптимизаця CSS
1) Аудит неиспользуемого CSS
2) Выделение critical css и максимальная быстрая доставка (<14kb TCP magic number, inline)

## Оптимизация Картинок
1) Поддержать WebP

## Оптимизация шрифтов
1) Победить FOIT 
2) Webfonts baseline

## Turbolinks, SPA

## Оптимизация порядка загрузки
Как браузер строит страницу
1) Скачать HTML
2) Начал парсить HTML
3) Встретил ресурс - запустил загрузку асинхронно, пошел дальше
4) Встретил <script> - остановился! скачал! выполнил! продолжаем 
5) CSS в head - блокирует рендеринг до завершения CSSOM

Async, defer

Resources-hints
1) prefetch
2) preload
3) HTTP/2 push

# Оптимизация Developer eXperience (DX) (оптимизация своего рабочего процесса)
Быстрый feedback-loop ускоряют разработку в 10x раз!

## Сбор DX-метрик
1) Время загрузки приложения у разработчиков
2) Время прогона CI
3) Время деплоя на прод
4) Время assets precompile
5) Количество коммитов в день
6) Удовлетворенность работой \
и тд

InfluxDB - #1 time-series DBMS

Tick Stack

## Загрузка приложения
Bootsnap > spring

`time bundle exec rake environment` \
No optimization -> 12s \
Spring -> 5s \
Bootsnap -> 5s

## Оптимизация тестов
Parallel tests
1) gem parallel_tests
2) В rails 6 из коробки - PARALLEL_WORKERS=15 bin/rails test \
Создаем N бд \
Каждый процесс работает со своей бд

Pitfalls
1) Sidekiq inline ~ сжирает много времени
2) Логирование в тестах - config.logger = Logger.new(nil), config.log_leverl = :fatal
3) Upload - проверить сколько генерируются файлы из FactoryBot (Rack::Test::UploadedFile.new лучше убрать)
4) DatabaseCleaner нужен ли?

Тест профайлинг
1) rspec --profile - находим самые медленные тесты
2) gem test-prof
SAMPLE=10 TEST_RUBY_PROF=1 rspec

Rspec-Dissect \
RD_PROF=1 rspec \
Total `let` time, Total before(:each) time \

before_all, let_it_be

FactoryProf \
FPROF=1 rspec \
FactoryProf Flamegraph Report \
Cascade Fix

Factory-Doctor \
FDOC=1 rspec

Event-Prof \
EVENT_PROF=factory.create rspec

Guard \
gem terminal-notifier-guard

## Покрытие кода тестами
Почему бы и не 100% ? \
PRONTO + UNDERCOVER + BUSFOR_BOT

## Frontend DX
Принцип Брета Виктора - творцам необходима немедленная обратная связь с тем, что они создают
https://github.com/JoshCheek/seeing_is_believing
gem guard-livereload

Ускорение сборки фронтенда \
1) Sprockets, Browserify - читаем офф гайды
2) sass-rails -> sassc-rails (ускорение в два раза)
3) uglifier (вызывать напрямую, а не через ExecJS)
4) Чистить пути и фильтры прекомпиляции в assets.rb

## CI DX
1) Кеширование node_modules по Yarn.lock
2) Кеширование гемов по Gemfile.lock
3) Кеш для webpack-cache-loader
4) Запуск тестов в два процесса
