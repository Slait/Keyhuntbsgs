Инструкция по режиму -m FastECDLP
=================================

Этот файл описывает экспериментальный режим FastECDLP, добавленный в keyhunt.
Режим предназначен для проверки алгоритма FastECDLP/BSGS-подобного поиска на
тех же публичных ключах и диапазонах, с которыми работает обычный BSGS.

Важно:
FastECDLP в текущей реализации является экспериментальным режимом для сравнения
с -m bsgs. Он уже умеет строить baby-step таблицу T1, использовать sorted или
cuckoo lookup backend, распараллеливать giant loop по -t, использовать batch32
для ускорения giant points, а также сохранять/загружать cache T1/T2.

Базовый запуск
--------------

Для примера файл test.txt содержит 037d5b40e63c41a0b70d499a41c8f256569bd4bf169b5e02d668bb0498c7178cea

Пример:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16

То же самое можно писать строчными буквами:

  keyhunt.exe -m fastecdlp -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16

Основные параметры:

  -m FastECDLP
    Включает экспериментальный режим FastECDLP.

  -f test.txt
    Файл с публичными ключами. Формат такой же, как для BSGS:
    compressed public key длиной 66 hex-символов или uncompressed public key
    длиной 130 hex-символов.

  -r START:END
    Диапазон приватных ключей в hex.

  -t N
    Количество потоков. В FastECDLP giant loop делится между потоками.

  -s 0
    Отключает периодический вывод статистики обычного keyhunt. Для FastECDLP
    итоговые метрики печатаются в конце запуска.


Что делает FastECDLP
--------------------

Для каждого публичного ключа Q и диапазона [start, end] режим считает:

  Q' = Q - start*G

Затем ищет offset x, такой что:

  Q' = x*G

Если offset найден, приватный ключ:

  privkey = start + x

Алгоритм делит поиск на:

  T1 / baby steps
    Таблица маленьких шагов i*G.

  giant loop
    Последовательные проверки Q' - j*B, где B = 2^l1 * G.

Для совпадений по x-координате проверяются оба кандидата:

  j*B + i
  j*B - i

Финальная проверка всегда делается через пересчет public key, поэтому ложное
совпадение в lookup table не должно давать ложный найденный ключ.


--fe-l1 N
---------

Ручной выбор размера baby-step таблицы T1.

Пример:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16 --fe-l1 20

Допустимый диапазон:

  10..40

Что означает l1:

  baby_count = 2^l1

Чем больше l1:

  больше T1 в памяти и на диске;
  дольше создание T1;
  меньше giant steps;
  потенциально выше скорость поиска, если хватает RAM и cache не начинает
  сильно тормозить.

Примеры размеров для sorted backend:

  --fe-l1 20
    baby_count = 1,048,576
    T1 примерно 16 MB

  --fe-l1 24
    baby_count = 16,777,216
    T1 примерно 256 MB

  --fe-l1 27
    baby_count = 134,217,728
    T1 примерно 2 GB

Пример для таблицы около 1.8-2.0 GB:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16 --fe-backend sorted --fe-l1 27


--fe-t1-mb N
------------

Автоматически выбирает l1 под примерный размер T1 в мегабайтах.

Пример:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16 --fe-backend sorted --fe-t1-mb 1800

Что делает:

  программа оценивает, какой l1 даст T1 примерно указанного размера;
  выбранный l1 печатается в начале запуска;
  если одновременно указан --fe-l1, то --fe-l1 имеет приоритет.

Важно:
--fe-t1-mb задает размер именно T1 lookup table FastECDLP, а не bloom-фильтра.
В режиме FastECDLP сейчас нет bloom-фильтра как в -m bsgs. Здесь T1 является
таблицей baby steps.

Пример:

  --fe-backend sorted --fe-t1-mb 1800

Обычно выберет l1 около 30, потому что sorted T1 хранит примерно 16 байт на
элемент.


--fe-backend sorted|cuckoo
--------------------------

Выбирает backend для T1 lookup.

Пример sorted:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16 --fe-backend sorted --fe-l1 20

Пример cuckoo:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16 --fe-backend cuckoo --fe-l1 20

sorted:

  T1 хранится как массив key/index;
  после создания сортируется;
  поиск идет через binary search;
  память примерно 16 байт на элемент;
  надежный fallback backend.

cuckoo:

  T1 строится как cuckoo hash table;
  поиск обычно быстрее, чем binary search;
  память в текущей реализации примерно 22 байта на элемент;
  если cuckoo table не построилась, программа автоматически откатывается на
  sorted backend и печатает предупреждение.

Практический смысл:

  sorted обычно экономнее по памяти;
  cuckoo может быть быстрее на lookup, но использует больше RAM.


--fe-t2-limit N
---------------

Задает лимит giant steps, при котором FastECDLP будет заранее строить T2 cache
и использовать batch32 по T2.

Пример:

  keyhunt.exe -m FastECDLP -f test.txt -r 1:5000 -t 4 --fe-l1 10 --fe-t2-limit 100

Если:

  giant_count <= --fe-t2-limit

то программа строит/загружает T2:

  filters/FastECDLP/t2_l*_g*.bin

Если:

  giant_count > --fe-t2-limit

то программа работает в T2=stream режиме, без хранения всей T2 в памяти.

Значение по умолчанию:

  1048576

Отключить T2 cache:

  --fe-t2-limit 0

Пример:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16 --fe-l1 20 --fe-t2-limit 0

Важно:
Даже при T2=stream текущая реализация использует batch32 для giant points.
То есть stream mode уже ускорен и не делает одиночный AddDirect на каждый шаг.


--fe-max-giants N
-----------------

Ограничивает количество giant steps. Используется для benchmark-тестов, чтобы
не ждать полный проход огромного диапазона.

Пример:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16 --fe-backend cuckoo --fe-l1 20 --fe-t2-limit 0 --fe-max-giants 100000

Что делает:

  программа посчитает только первые N giant steps;
  если ключ не находится в этой части, программа завершится;
  итоговые метрики покажут скорость lookup/generation на выбранном отрезке.

Это полезно для сравнения вариантов:

  sorted vs cuckoo;
  разные --fe-l1;
  разные -t;
  T2=batch32 vs T2=stream.

Важно:
Для реального полного поиска не указывайте --fe-max-giants.


--fe-profile
------------

Включает подробные метрики горячего пути FastECDLP.

Пример:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16 --fe-backend cuckoo --fe-l1 20 --fe-t2-limit 0 --fe-max-giants 100000 --fe-profile

Дополнительно печатает строку:

  FastECDLP profile:

Основные поля:

  add32_ns_call
    Среднее время одного AddDirect32 batch-вызова.

  offset_ns_lookup
    Среднее время расчета offset на один lookup.

  xkey_ns_lookup
    Среднее время извлечения 64-bit xkey на один lookup.

  backend_ns_lookup
    Среднее время lookup в sorted/cuckoo backend.

  check_key_ns_lookup
    Среднее суммарное время check_key на один lookup.

  *_thread_share
    Доля thread-time относительно wall-time. В многопоточном запуске может быть
    больше 100%, это нормально. Используйте это как индикатор горячих мест, а
    не как обычный процент.

Важно:
--fe-profile добавляет overhead из-за таймеров и atomic-счетчиков. Для финальной
скорости запускайте benchmark без --fe-profile, а профиль включайте только для
анализа узких мест.


Cache T1/T2
-----------

FastECDLP cache создается автоматически в папке:

  filters/FastECDLP/

T1 cache:

  filters/FastECDLP/t1_l10_b1024.bin
  filters/FastECDLP/t1_l20_b1048576.bin
  filters/FastECDLP/t1_l27_b134217728.bin

Формат имени:

  t1_l{l1}_b{baby_count}.bin

T2 cache:

  filters/FastECDLP/t2_l10_g21.bin
  filters/FastECDLP/t2_l20_g100000.bin

Формат имени:

  t2_l{l1}_g{giant_count}.bin

Когда cache создается:

  если файла нет;
  если параметры l1/baby_count/giant_count отличаются;
  если файл не читается или не совпадает по ожидаемому размеру.

Когда cache загружается:

  если файл уже есть и соответствует текущим параметрам.

В выводе будет видно:

  [+] FastECDLP T1 cache saved: ...
  [+] FastECDLP T1 cache loaded: ...
  [+] FastECDLP T2 cache saved: ...
  [+] FastECDLP T2 cache loaded: ...


Метрики FastECDLP
-----------------

Пример вывода:

  [+] FastECDLP l1=20 l2=17 baby=1048576 giant=100000/1529008357377 T1=16.00 MB backend=cuckoo T2=stream 0.00 MB
  [+] FastECDLP metrics: precompute=0.155s search=0.007s lookups=100000 lookup_ns=72.69 table_hits=0 candidates=0 verified=0 hit_rate=0.000000% candidate_rate=0.000000% batch32=3040
  [+] FastECDLP estimated speed: 0.014 Pkeys/s (14425312972899 keys/s)

Поля:

  l1
    Размер baby-step части. baby_count = 2^l1.

  l2
    Приблизительный размер giant-step части в битах.

  baby
    Количество baby steps.

  giant=A/B
    A = сколько giant steps будет реально проверено в этом запуске.
    B = сколько giant steps нужно для полного диапазона.
    Если указан --fe-max-giants, A может быть меньше B.

  T1
    Примерный размер T1 в памяти.

  backend
    sorted или cuckoo.

  T2
    batch32 или stream.

  precompute
    Время подготовки T1/T2/cache/backend.

  search
    Время поиска по giant loop.

  lookups
    Количество обращений к T1.

  lookup_ns
    Среднее время на один giant lookup с учетом генерации giant points.

  table_hits
    Количество совпадений в T1.

  candidates
    Количество кандидатов, прошедших до финальной проверки.

  verified
    Количество подтвержденных кандидатов.

  hit_rate
    table_hits / lookups.

  candidate_rate
    candidates / lookups.

  batch32
    Количество batch32-вызовов для генерации giant points.

  estimated speed
    Оценка скорости в keys/s и Pkeys/s.


Сравнение с BSGS bloom/fuse
---------------------------

Обычный BSGS:

  keyhunt.exe -m bsgs -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -k 128 -S -s 1 -x bloom -t 16

  keyhunt.exe -m bsgs -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -k 128 -S -s 1 -x fuse -t 16

В BSGS bloom/fuse фильтры создаются и загружаются через -S.

Пути:

  filters/bloom/k128n0x100000000000/
  filters/FUSE/k128n0x100000000000/

Размер bloom/fuse зависит от:

  -n
    Определяет N для BSGS. По умолчанию N = 0x100000000000.

  -k
    Множитель M. Чем больше -k, тем больше фильтры и RAM, но меньше giant work.

  -x
    Тип backend: bloom, fuse, xor, blocked.

Для bloom в текущем коде используется false-positive rate 0.000001.

FastECDLP:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16 --fe-backend cuckoo --fe-l1 20 --fe-t2-limit 0 --fe-max-giants 100000

Для полного поиска убрать:

  --fe-max-giants


Как получить T1 около 1.8 GB
----------------------------

Вариант 1, автоматический:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16 --fe-backend sorted --fe-t1-mb 1800

Вариант 2, ручной:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16 --fe-backend sorted --fe-l1 27

Для sorted:

  l1=27 примерно 2.0 GB.

Для cuckoo:

  l1=26 примерно 1.4 GB;
  l1=27 примерно 2.8 GB в текущей реализации.

Если RAM ограничена, лучше начинать с:

  --fe-l1 24
  --fe-l1 25
  --fe-l1 26


Важно про "чем больше фильтр, тем быстрее"
-----------------------------------------

Идея частично верная, но с оговорками.

В BSGS:

  больший -k увеличивает M и фильтры;
  giant loop становится короче;
  но растет RAM и время подготовки/загрузки.

В FastECDLP:

  больший T1 увеличивает baby_count;
  full_giant_count уменьшается;
  lookup table становится больше;
  cache/RAM pressure растет;
  слишком большой T1 может начать тормозить из-за памяти.

Поэтому оптимум нужно искать тестами.

Рекомендуемый порядок тестов FastECDLP:

  1. Проверить корректность на маленьком диапазоне:

     keyhunt.exe -m FastECDLP -f test.txt -r 1:5000 -t 4 --fe-backend sorted --fe-l1 10

  2. Проверить cuckoo:

     keyhunt.exe -m FastECDLP -f test.txt -r 1:5000 -t 4 --fe-backend cuckoo --fe-l1 10

  3. Benchmark на большом диапазоне без полного прохода:

     keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16 --fe-backend cuckoo --fe-l1 20 --fe-t2-limit 0 --fe-max-giants 100000

  4. Увеличивать l1:

     --fe-l1 22
     --fe-l1 24
     --fe-l1 26
     --fe-l1 27

  5. Сравнить estimated speed, lookup_ns, precompute и RAM.


Практические команды
--------------------

Маленький тест sorted:

  keyhunt.exe -m FastECDLP -f test.txt -r 1:5000 -s 0 --fe-backend sorted --fe-l1 10 -t 4

Маленький тест cuckoo:

  keyhunt.exe -m FastECDLP -f test.txt -r 1:5000 -s 0 --fe-backend cuckoo --fe-l1 10 -t 4

Benchmark sorted:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -s 0 --fe-backend sorted --fe-l1 20 --fe-t2-limit 0 --fe-max-giants 100000 -t 16

Benchmark cuckoo:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -s 0 --fe-backend cuckoo --fe-l1 20 --fe-t2-limit 0 --fe-max-giants 100000 -t 16

Пример T1 около 1.8 GB:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -s 0 --fe-backend sorted --fe-t1-mb 1800 --fe-t2-limit 0 --fe-max-giants 100000 -t 16

Полный запуск без benchmark-ограничителя:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -s 1 --fe-backend sorted --fe-l1 27 --fe-t2-limit 0 -t 16
