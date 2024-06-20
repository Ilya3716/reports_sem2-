# Лабораторная работа 2. Потоки. Процессы. Асинхронность.

---

## Цель работы

Понять отличия потоками и процессами и понять, что такое ассинхронность в Python.

## Задание

### Задача 1. Различия между threading, multiprocessing и async в Python

**_Задача_**: напишите три различных программы на Python, использующие каждый из подходов: threading, multiprocessing и async. Каждая программа должна решать считать сумму всех чисел от 1 до 1000000. Разделите вычисления на несколько параллельных задач для ускорения выполнения.

**_Подробности задания:_**
напишите программу на Python для каждого подхода: threading, multiprocessing и async.
Каждая программа должна содержать функцию calculate_sum(), которая будет выполнять вычисления.
Для threading используйте модуль threading, для multiprocessing - модуль multiprocessing, а для async - ключевые слова async/await и модуль asyncio.
Каждая программа должна разбить задачу на несколько подзадач и выполнять их параллельно.
Замерьте время выполнения каждой программы и сравните результаты.

### Задача 2. Параллельный парсинг веб-страниц с сохранением в базу данных

**_Задача:_** напишите программу на Python для параллельного парсинга нескольких веб-страниц с сохранением данных в базу данных с использованием подходов threading, multiprocessing и async. Каждая программа должна парсить информацию с нескольких веб-сайтов, сохранять их в базу данных.

### Подробности задания

Напишите три различных программы на Python, использующие каждый из подходов: threading, multiprocessing и async.
Каждая программа должна содержать функцию parse\*and_save(url), которая будет загружать HTML-страницу по указанному URL, парсить ее, сохранять заголовок страницы в базу данных и выводить результат на экран.
Используйте базу данных из лабораторной работы номер 1 для заполенния ее данными. Если Вы не понимаете, какие таблицы и откуда Вы могли бы заполнить с помощью парсинга, напишите преподавателю в общем чате потока.
Для threading используйте модуль threading, для multiprocessing - модуль multiprocessing, а для async - ключевые слова async/await и модуль aiohttp для асинхронных запросов.
Создайте список нескольких URL-адресов веб-страниц для парсинга и разделите его на равные части для параллельного парсинга.
Запустите параллельный парсинг для каждой программы и сохраните данные в базу данных.
Замерьте время выполнения каждой программы и сравните результаты.
Дополнительные требования:
\*\*\*Сделайте\_\*\* документацию, содержащую описание каждой программы, используемые подходы и их особенности.
Включите в документацию таблицы, отображающие время выполнения каждой программы.
Прокомментируйте результаты сравнения времени выполнения программ на основе разных подходов.

## Ход работы

### Задача 1

Сложить числа от 1 до миллиона <br>
Задача разбивается на 4 подзадачи – складываем числа в 4 итерации по 250.000 чисел<br>

**_async:_** создаем эти подзадачи, добавляем их в массив и складываем в gather.<br>

```python
async def async_calculate_sum():
    num_tasks = 4
    n = 1000000
    step = n // num_tasks
    tasks = []

    for i in range(num_tasks):
        s = i * step + 1
        e = (i + 1) * step if i != num_tasks - 1 else n
        tasks.append(asyncio.create_task(async_calculate_part(s, e)))

    results = await asyncio.gather(*tasks)
    total_sum = sum(results)
    return total_sum
```

**_multiprocessing:_** создаем менеджера, создаем и запускаем процессы для наших подзадач.<br>

```python
def mp_calculate_sum():
    num_tasks = 4
    n = 1000000
    tasks_list = []
    step = n // num_tasks

    manager = multiprocessing.Manager()
    res = manager.list([0] * num_tasks)

    for i in range(num_tasks):
        s = i * step + 1
        e = (i + 1) * step if i != num_tasks - 1 else n
        process = multiprocessing.Process(target=mp_calculate_part, args=(s, e, res, i))
        tasks_list.append(process)
        process.start()

    for process in tasks_list:
        process.join()

    _sum = sum(res)
    return _sum
```

**_threading:_** раскидываем эти подзадачи на разные потоки.<br>

```python
def thread_calculate_sum():
    num_tasks = 4
    n = 1000000
    tasks_list = []
    result = [0] * num_tasks
    step = n // num_tasks

    for i in range(num_tasks):
        start = i * step + 1
        end = (i + 1) * step if i != num_tasks - 1 else n
        thread = threading.Thread(target=thread_calculate_part, args=(start, end, result, i))
        tasks_list.append(thread)
        thread.start()

    for thread in tasks_list:
        thread.join()

    total_sum = sum(result)
    return total_sum
```

### Результаты: <br>

В среднем результаты получились такие:<br>
**_В async_**0,04 - 0,05 секунд.<br>
**_В multiprocessing_** - 0,6 - 0,7 секунд. <br>
**_В threading_** – 0,04 - 0,06 секунд. <br>

В этой задаче лучше всего себя показали асинк и трэдинг, чем мультипроцессинг,
так как в мультипроцессинге есть расходы на создание процессов.

### Задача 2

Тут я взял категории сайта буквоед, парсил данные и заносил их в бд. <br>
**_Ход работы:_**<br>

**_async_**

```python
async def async_main():
    await create_table()
    async with aiohttp.ClientSession() as session:
        tasks = [async_parse_and_save(session, url) for url in urls]
        await asyncio.gather(*tasks)
```

**_multiprocessing_**

```python
def mlp_main():
    processes = []
    for url in urls:
        process = multiprocessing.Process(target=mlp_parse_and_save, args=(url,))
        processes.append(process)
        process.start()

    for process in processes:
        process.join()
```

**_threading_** – 2-3 секунды.

```python
def thread_main():
    threads = []
    for url in urls:
        thread = threading.Thread(target=thread_parse_and_save, args=(url,))
        threads.append(thread)
        thread.start()

    for thread in threads:
        thread.join()
```

### Результаты: <br>

В среднем результаты получились такие:<br>
**_В async_** - 1-3 секунды. <br>
**_В multiprocessing_** - 4-5 секунд. <br>
**_В threading_** – 2-3 секунды. <br>

В этой задаче лучше всего результаты у async, так как это задача ввода-вывода, а с этим как раз таки лучше справляется async
threading в целом тоже неплохо себя показывает, но из-за особенностей интерпретатора python, потоки не выполняют код одновременно, соответственно тут время среднем больше
Ну и хуже всего себя показал multiprocessing, потому что как и в первом задании – процессы довольно минорные, а вот на их создание и управление уходит больше ресурсов, чем в threading или async.
