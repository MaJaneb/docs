# Сравнение методов вычисления суммы чисел

В этой работе я сравнил три подхода к вычислению суммы чисел от 1 до 1_000_000_000, реализовав их в отдельных скриптах в каталоге `lab2/task1`:

1. **Асинхронный подход (`asyncio`)**
2. **Многопроцессный подход (`multiprocessing`)**
3. **Многопоточный подход (`threading`)**

---

## 1. Асинхронный подход

```python
import asyncio
import time

async def partial_sum(start, end):
    return sum(range(start, end))

async def main():
    n = 1_000_000_000
    num_tasks = 10
    chunk_size = n // num_tasks

    start_time = time.time()

    tasks = []
    for i in range(0, n, chunk_size):
        start = i + 1
        end = min(i + chunk_size, n) + 1
        tasks.append(partial_sum(start, end))

    results = await asyncio.gather(*tasks)
    total = sum(results)

    end_time = time.time()
    print(f"Сумма: {total}, время: {end_time - start_time:.2f} сек")

asyncio.run(main())
```

Особенности:

Используются корутины `asyncio` и 10 задач, каждая считает часть диапазона.

Асинхронность не ускоряет CPU-bound вычисления, так как нет ожиданий I/O.

Примерное время выполнения зависит от CPU; ускорения относительно однопоточного варианта нет.


# 2. Многопроцессный подход
```python
import multiprocessing
import os
import time


def partial_sum(start, end, res_queue):
    total = sum(range(start, end))
    res_queue.put(total)


def main():
    n = 1_000_000_000
    num_processes = 4
    chunk_size = n // num_processes

    res_queue = multiprocessing.Queue()
    processes = []

    start_time = time.time()

    for i in range(num_processes):
        start = i * chunk_size + 1
        if i == num_processes - 1:
            end = n + 1
        else:
            end = start + chunk_size

        p = multiprocessing.Process(
            target=partial_sum,
            args=(start, end, res_queue)
        )
        processes.append(p)
        p.start()

    for p in processes:
        p.join()

    results = []
    while not res_queue.empty():
        results.append(res_queue.get())

    total = sum(results)
    end_time = time.time()

    print(f"Сумма: {total}, время: {end_time - start_time:.2f} сек")


if __name__ == '__main__':
    main()
```


Особенности:

Диапазон делится на 4 части, каждая считается в отдельном процессе.

Масштабируется на CPU-bound задачах за счёт обхода GIL.

Обычно самый быстрый из трёх подходов на многоядерных системах.



# 3. Многопоточный подход

```python
import threading
import time

def partial_sum(start, end, res, lock):
    total = sum(range(start, end))
    with lock:
        res.append(total)

def main():
    n = 1_000_000_000
    num_threads = 4
    chunk_size = n // num_threads

    results = []
    lock = threading.Lock()
    threads = []

    start_time = time.time()


    for i in range(0, n, chunk_size):
        start = i + 1
        end = min(i + chunk_size, n) + 1

        t = threading.Thread(
            target=partial_sum,
            args=(start, end, results, lock))
        threads.append(t)
        t.start()

    for t in threads:
        t.join()

    total = sum(results)
    end_time = time.time()

    print(f"Сумма: {total}, время: {end_time - start_time:.2f} сек")

if __name__ == '__main__':
    main()
```

Особенности:

Используются 4 потока в одном процессе, результаты защищаются через `Lock`.

Подходит для I/O-bound задач; на CPU-bound выигрыша обычно нет из-за GIL.

Время выполнения похоже на однопоточное, иногда чуть хуже из-за накладных расходов.

# Выводы
Для CPU-bound задач в этой работе эффективнее всего показал себя `multiprocessing` (4 процесса). `asyncio` и `threading` не дают ускорения на чисто вычислительной задаче.

Во всех реализациях получается один и тот же результат суммы для диапазона 1..1_000_000_000: 500000000500000000

