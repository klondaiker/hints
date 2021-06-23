# Процессы
Получение списка процессов, сортированный по FIELD
```
top -o %FIELD
```

Получение списка процессов по наименованию NAME
```bash
ps aux | grep NAME
```

Количество открытых дескрипторов конкретного процесса PID
```bash
ls /proc/PID/fd | wc -l
```

Список открытых сокетов конкретного процеcса PID
```bash
lsof -i -a -p PID
```

# Дисковое пространство 
Общая информация
```bash
df -h
```
Список самых жирных директории в DIR_NAME
```bash
du -a /DIR_NAME | sort -n -r | head -n 10
```

Список самых жирных файлов в DIR_NAME
```bash
find /path/to/dir/ -printf '%s %p\n'| sort -nr | head -10
```
