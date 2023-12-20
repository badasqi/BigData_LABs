# Лабораторная работа № 4
# Задание
+ запустить ZooKeeper,
+ изучить директорию с установкой ZooKeeper,
+ запустить интерактивную сессию ZooKeeper CLI и освоить её команды,
+ научиться проводить мониторинг ZooKeeper,
+ разработать приложение с барьерной синхронизацией, основанной на ZooKeeper,
+ запустить и проверить работу приложения.

# Часть работы №1 
## Установка и запуск ZooKeeper
Zookeeper был установлен на WSL. Скриптом zkServer.sh был запущен сервер. Для запуска сессии ZooKeeper
использовался скрипт zkCli.sh

### rename файла zoo.cfg
![image](https://github.com/badasqi/BigData_LABs/assets/78803025/46d7a979-8be1-4dfd-8f96-be82edbc19ac)

### Запуск сервера
![image](https://github.com/badasqi/BigData_LABs/assets/78803025/eff7f7e1-3143-4e35-b9a6-abc11d6e74aa)

### Запуск сессии и вывод команды help
![image](https://github.com/badasqi/BigData_LABs/assets/78803025/25d4f000-c24c-4fd6-91a7-4ae4177e4cfc)

## Работа с командами ZooKeeper
Работа с командами для добавления и удаления узлов znode, считывания и записи данных в znode из CLI сессии.
С помощью команды ls / получен список узлов в корне ZooKeeper. 

Создадим узел /mynode с данными "first_version" и проверим его командой ls / и get

![image](https://github.com/badasqi/BigData_LABs/assets/78803025/b5b30272-b966-4dd4-9d70-6a2d1035f478)
![image](https://github.com/badasqi/BigData_LABs/assets/78803025/347eeacf-bcb7-4b9b-9ab2-635410292615)

Создадим два нумерованных sequential дочерных узла mynode. Флагс -s указывает, что создаваемый узел нумерованный. По нумерации можно определить хронологию запросов на сервере.

![image](https://github.com/badasqi/BigData_LABs/assets/78803025/f02d992b-72a2-4328-9955-c2e1ca9c4156)

### Пример. Принадлежность клиентов к группе
Откроем две новые CLI консоли:
Консоль 1:

![image](https://github.com/badasqi/BigData_LABs/assets/78803025/3cd7884c-0bc8-426d-bc8d-9fc4cfbd15fc)

Консоль 2:

![image](https://github.com/badasqi/BigData_LABs/assets/78803025/2631a04a-8f56-4d71-9811-6d7fb9167598)

Внутри CLI сессии создадим узел mygroup и в каждой консоли создадим дочерний узел в 1 консоли `grue` во второй консоли `bleen` в mygroup и проверим их 

Консоль 1 после создания:

![image](https://github.com/badasqi/BigData_LABs/assets/78803025/68ff7a86-6085-40cb-a10d-5d5f154cba78)

Консоль 2 после создания:

![image](https://github.com/badasqi/BigData_LABs/assets/78803025/dd16d85f-f7e9-4911-b7cd-a03f720a6f5a)

Сделаем запрос от клиента о другом клиенте во время его работы и после аварийного его завершения:

![image](https://github.com/badasqi/BigData_LABs/assets/78803025/646f4ddc-2876-47c5-83b7-2aeb810fb627)

При повторном запросе клиент пропал из группы (изменение списка дочерних узлов может произойти не сразу, через tickTime, который устанавливаетсяв zoo.cfg):

![image](https://github.com/badasqi/BigData_LABs/assets/78803025/12a7fee4-6852-4764-993b-92f094a223be)

В заключение удалим узел /mygroup

![image](https://github.com/badasqi/BigData_LABs/assets/78803025/ee43d1da-ceff-43cf-b43f-a441bd242c57)

## Мониторинг ZooKeeper
Вывод команд `ruok`, `stat`, `cons`, `conf`

![image](https://github.com/badasqi/BigData_LABs/assets/78803025/876a40d8-c82b-4b90-9387-747c81d58bdd)

![image](https://github.com/badasqi/BigData_LABs/assets/78803025/9988a6e6-cce3-4744-8f20-580fe82e9779)

# Часть работы №2
## Настройка и запуск распределённого приложения
![monkey](https://github.com/badasqi/BigData_LABs/assets/78803025/ad7caad9-3a86-44bc-8b24-b10f84fb282b)
![tiger](https://github.com/badasqi/BigData_LABs/assets/78803025/d7cd9dd2-20cb-413c-afa7-2827b5175312)
## Philosophers

Каждый филосов - поток, при запуске подключающийся к ZooKeeper и далее запускающий итерации eat(), think().
eat() создаёт node, think() удаляет её. Вилки сделаны как threading.Semaphore(1) каждая вилка может принадлежать только одному потоку, который успеет её занять.

Результат: 
```
Philosopher 0 is going to eat
Philosopher 4 is going to eat
Philosopher 1 is going to eat
Philosopher 2 is going to eat
Philosopher 3 is going to eat
Philosopher 2 picked up the left fork
Philosopher 2 picked up the right fork
Philosopher 5 picked up the left fork
Philosopher 5 picked up the right fork
Philosopher 4 picked up the left fork
Philosopher 5 put the right fork
Philosopher 5 put the loft fork and finished eating
Philosopher 4 picked up the right fork
Philosopher 5 is thinking
Philosopher 1 picked up the left fork
Philosopher 2 put the right fork
Philosopher 2 put the loft fork and finished eating
Philosopher 1 picked up the right fork
Philosopher 3 picked up the left fork
Philosopher 2 is thinking
Philosopher 4 put the right fork
Philosopher 1 put the right fork
Philosopher 1 put the loft fork and finished eating
Philosopher 4 put the loft fork and finished eating
Philosopher 3 picked up the right fork
Philosopher 1 is thinking
Philosopher 4 is thinking
Philosopher 4 is going to eat
Philosopher 5 picked up the left fork
Philosopher 5 picked up the right fork
Philosopher 1 is going to eat
Philosopher 2 picked up the left fork
Philosopher 5 put the right fork
Philosopher 5 put the loft fork and finished eating
Philosopher 1 picked up the left fork
Philosopher 5 is thinking
Philosopher 3 is going to eat
Philosopher 3 put the right fork
Philosopher 3 put the loft fork and finished eating
Philosopher 2 picked up the right fork
Philosopher 3 is thinking
Philosopher 4 picked up the left fork
Philosopher 4 picked up the right fork
Philosopher 2 put the right fork
Philosopher 2 put the loft fork and finished eating
Philosopher 1 picked up the right fork
Philosopher 2 is thinking
Philosopher 2 is going to eat
Philosopher 3 picked up the left fork
Philosopher 1 put the right fork
Philosopher 1 put the loft fork and finished eating
Philosopher 1 is thinking
Philosopher 4 put the right fork
Philosopher 4 put the loft fork and finished eating
Philosopher 3 picked up the right fork
Philosopher 4 is thinking
Philosopher 3 put the right fork
Philosopher 3 put the loft fork and finished eating
Philosopher 3 is thinking
```
## Двухфазный коммит протокол для high-available регистра
Все процессы - потоки. Координатор создаёт 
```
Waiting others clients: []
Client 1 request commit
Client 0 request rollback
Client 2 request rollback
Client 4 request commit
Client 3 request commit
Check clients
Client 0 do commit
Client 1 do commit
Client 2 do commit
Client 3 do commit
Client 4 do commit
Waiting others clients: []
```










