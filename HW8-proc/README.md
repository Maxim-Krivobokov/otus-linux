### Домашняя работа №8.

### Задание - написать свою реализацию ps ax используя анализ /proc.

1. Команда ps ax выводит информацию о процессах в формате: PID - TTY - STAT - TIME - COMMAND

2. параметр PID - возьмем из списка директории /proc
Используем ls -v для вывода сортированного списка процессов

3. Параметр tty - из символической ссылки /proc/<PID>/fd/0, читаем ее с помощью readlink

4. STAT - состоит из статуса (running, sleeping, зомби, и пр. ) флажка '< или N ' (если процесс с высоким/низким приоритетом)  и  четырех дополнительных параметров - is_lock is_leader, is_foreground, is_thread. Все эти параметры читаем из файла /proc/<PID>/stat. 

5. TIME высчитываем из него же. 

6. Command - читая файл /proc/<PID>/cmdline. В случае, если он пуст, берем параметр name из /proc/<PID>/status

7. образуем строку stat из всех переменных - is_slock, is_leader, is_thread, is_foreground

8. с помощью printf выводим все переменные в читаемом формате

9. Для работы с файлами использованы grep, awk, cut, xargs. 

10. Скрипт копируеся провижинером при запуске vagrant в домашнюю директорию, запускается из нее. 