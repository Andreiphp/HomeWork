# 1) Lecture20 - task1: упростить скрипт, сделать проверку в цикле
1) Создать скрипт, который проходит по всем аргументам которым мы передаем в скрипт и выводит их ( каждый в новой строке)
было:
```bash
#!/bin/bash
count=1
for i in "$@"
do 
echo "parametr $count = $i"
count=$(($count+1))
done
```



# 2) Баш скрипт, который проверяет дисковое пространство на удаленных машинах
  ##  1. Если больше 70 процентов - шлет на почту оповещение
  ##  2. Если больше 80 процентов - шлет список самых больших 10 файлов по размеру
  ##  3. Если больше 90 процентов - удаляет файлы из var/log - топ 5

Для выполнения задачи, мне нужны два скрипта. Начнем с того, что поменьше 
```bash
#!/bin/bash
df -h | grep "/dev/vda5" | awk '{print $3/$2}' 
```
На него ссылаться будем, из основного.
Второй скрипт
```bash
#!/bin/bash 
FROM=jeka1993g@mail.ru
MAILTO=jeka1993g@mail.ru 
NAME=ALERT
SMTPSERVER=smtp.mail.ru
SMTPLOGIN=jeka1993g@mail.ru 
SMTPPASS=199514qaz
var=$(ssh rab2@192.168.122.150 'bash -s' < part1.sh) 
bar=$( echo $var | sed 's/,/./')
echo $bar
var1=$(echo "scale=1; $bar*10" | bc)
echo $var1
BODY="Осталось $var1 . Надо чистить"
if (( $(echo "$var1<7" | bc -l) ))
  then echo "Norm"
  else 
  if (( $(echo "$var1<8" | bc -l) ))
    then echo "Mail send for $FROM"
         /usr/bin/sendEmail -f $FROM -t $MAILTO -o message-charset=utf-8  -u $NAME -m $BODY -s $SMTPSERVER -o tls=yes -xu $SMTPLOGIN -xp $SMTPPASS
    else
      if (( $(echo "$var1 < 9" | bc -l) ))
        then echo "List 10 big file"
             ls -1S /var/log | head -n10
        else 
         if (( $(echo "$var1<10" | bc -l) ))
           then echo "Del 5 big file directory /var/log"
                ls -1S /var/log | head -n5 -exec rm -rf {} \;
           else echo "Ploho"
            fi
        fi
    fi
fi
```

# 3) Bash trap
```bash
trap ДЕЙСТВИЕ СИГНАЛ...
```
Trap работает очень просто: при возникновении сигнала будет выполнено указанное действие. Если действие простое (последовательность команд, умещающаяся на одной строке), его можно указать прямо в аргументе trap. Если не очень простое, то надо объявить функцию и поместить вызов этой функции в trap.

Можно обрабатывать стандартные сигналы (их полный список выводится по trap -l). Также доступны специфические для Bash: DEBUG, RETURN, ERR, EXIT.

побовался и разбирался при помощи примера из инета:
text script
```bash
#!/bin/bash

trap 'echo trap SIGINT' SIGINT
trap 'echo trap SIGTERM' SIGTERM
trap 'echo trap SIGHUP' SIGHUP
trap 'echo trap SIGQUIT' SIGQUIT
trap 'echo trap EXIT' EXIT
trap 'echo trap ERR' ERR

echo 'start'
sleep 1m
echo 'end'
```
вывод
```bash
anik@K53:~$ nano traptest.sh
anik@K53:~$ chmod +x traptest.sh
anik@K53:~$ ./traptest.sh
start
^Ctrap SIGINT
trap ERR
end
trap EXIT
anik@K53:~$ 
```
# 4) Обработка ошибок
Exit и Return

код завершения
Функции возвращают значение в виде кода завершения. Код завершения может быть задан явно, с помощью команды return, в противном случае будет возвращен код завершения последней команды в функции (0 -- в случае успеха, иначе -- ненулевой код ошибки). Код завершения в сценарии может быть получен через переменную $?.

return
Завершает исполнение функции. Команда return [2] может иметь необязательный аргумент типа integer, который возвращается в вызывающий сценарий как "код завершения" функции, это значение так же записывается в переменную $?.
код ошибки последней запущенной программы с проверкой $? 
пример
```bash
#!/bin/bash

touch /root/test

if [ $? -eq 0 ]
then
  echo "Successfully created file"
else
  echo "Could not create file"
fi
```

# 5) Библиотека
Оболочка bash позволяет создавать так называемые библиотеки — файлы, содержащие функции, а затем использовать эти библиотеки в любых скриптах, где они нужны.
Ключ к использованию библиотек — в команде source. Эта команда используется для подключения библиотек к скриптам. В результате функции, объявленные в библиотеке, становятся доступными в скрипте, в противном же случае функции из библиотек не будут доступны в области видимости других скриптов.

У команды source есть псевдоним — оператор «точка». Для того, чтобы подключить файл в скрипте, в скрипт надо добавить конструкцию такого вида:
```bash
. ./myscript
```
функцию из библиотеки можно подключить в файле .bashrc, используя команду source. 


# 6) Как вызывать функцию из командной строки!
функцию из библиотеки можно подключить в файле .bashrc, используя команду source.
ответил в предыдущем примере.

# 7) Expect

сначала его нужно установить ) 
```bash
sudo apt-get install expect
```
Expect предоставляет набор команд, позволяющих взаимодействовать с утилитами командной строки. Вот его основные команды:

spawn — запуск процесса или программы. Например, это может быть командная оболочка, FTP, Telnet, ssh, scp и так далее.

expect — ожидание данных, выводимых программой. При написании скрипта можно указать, какого именно вывода он ждёт и как на него нужно реагировать.

send — отправка ответа. Expect-скрипт с помощью этой команды может отправлять входные данные автоматизируемой программе. Она похожа на знакомую вам команду echo в обычных bash-скриптах.

interact — позволяет переключиться на «ручной» режим управления программой.

Доступный и понятый пример expect 
```bash
Пример
ftp_expect.sh
#!/usr/bin/expect -f
   ######################################################################################
   #$remote_server - сервер, к которому осуществляется доступ                           #
   #$my_user_id    - имя пользователя на сервере                                        #
   #$my_password   - пароль пользователя на сервере                                     #
   #$my_command    - команда, которая будет запущена на сервере        
   
   set remote_server xxx.xxx.xxx.225
   set my_user_id testuser
   set my_password testpass
   # Открыть ftp-сессию на удалённом сервере, и ждать запроса имени пользователя.
   spawn ftp $remote_server
   expect "username:"
   # Послать имя пользователя, и ждать приглашения ввода пароля.
   send "$my_user_id\r"
   expect "password:"
   # Послать пароль, и ждать приглашения ввода ftp.
   send "$my_password\r"
   expect "ftp>"
   # Переключиться в бинарный режим, и снова ждать приглашения ввода ftp.
   send "bin\r"
   expect "ftp>"
   # Выключить приглашение ввода.
   send "prompt\r"
   expect "ftp>"
   # Получить все файлы
   send "mget *\r"
   expect "ftp>"
   # Закончить ftp-сессию и ждать спецсимвол конца файла (eof).
   send "bye\r"
   expect eof
   ```

# 8) Typeset внутри функции  и local 
Инструкции declare и typeset являются встроенными инструкциями (они абсолютно идентичны друг другу и являются синонимами) и предназначена для наложения ограничений на переменные. 
Для объявления переменной в качестве локальной используется конструкция

  local x=a
Для определения переменных с контролем типа

   declare -r var              - объявление переменной var только для чтения (аналог константы)
   declare -i number           - объявление переменной number как целое число
   declare -a indices          - объявление переменной indices как массива
   declare -f functions        - выводит имя функции function_name, если она была объявлена ранее
   declare -x var_out          - объявление переменной var_out как доступной для экспорта
   declare -x var=$value       - объявление переменной var и присваивания значения ей одновременно
   declare -f                  - (без аргументов) выводит список ранее объявленных функций в сценарии

сохранил для себя
При инициализации оболочки Bash определяет большой набор внутренних переменных.

Отдельный класс внутренних переменных представляют собой переменные имена которых формируются специальными символами:

  $0     - имя выполняемого скрипта
  $#     - количество позиционных параметров, переданных сценарию
  $_     - последний аргумент предыдущей из выполнявшихся команд
  $?     - код завершения последней команды (0 – успешно, другое – ошибка)
  $$     - PID текущего процесса (? следующий свободный PID)
  $!     - PID последнего асинхронного процесса
  $*     - все позиционные параметры, собранные в одну строку («$1x$2x…$n»)
  $@     - все позиционные параметры, подлежащие дальнейшему разбору («$1» «$2» …«$n»)
  здесь позиционным параметром называется строка(и) переданная скрипту в качестве параметра при запуске.
  Имя позиционного параметра в скрипте представляет собой натуральное число начиная с 1, например
  $1 $2 ... - первый, второй и так далее позиционный параметр


# 9) Экранирование
Экранирование -- это способ заключения в кавычки одиночного символа. Экранирующий (escape) символ (\) сообщает интерпретатору, что следующий за ним символ должен восприниматься как обычный символ.
Специальное назначение некоторых экранированных символов :

используемых совместно с echo и sed
\n   перевод строки (новая строка)
\r   перевод каретки
\t   табуляция
\v   вертикальная табуляция
\b   забой (backspace)
\a   "звонок" (сигнал)
\0xx ASCII-символ с кодом 0xx в восьмеричном виде)

Поведение символа \ сильно зависит от того экранирован ли он, ограничен ли кавычками или находится внутри конструкции подстановки команды или во вложенном документе.

                      #  Простое экранирование и кавычки
echo \z               #  z
echo \\z              # \z
echo '\z'             # \z
echo '\\z'            # \\z
echo "\z"             # \z
echo "\\z"            # \z

                      #  Подстановка команды
echo `echo \z`        #  z
echo `echo \\z`       #  z
echo `echo \\\z`      # \z
echo `echo \\\\z`     # \z
echo `echo \\\\\\z`   # \z
echo `echo \\\\\\\z`  # \\z
echo `echo "\z"`      # \z
echo `echo "\\z"`     # \z

# 10)Работа с аргументами в функции

Функции могут использовать стандартные позиционные параметры, в которые записывается то, что передаётся им при вызове. Например, имя функции хранится в параметре $0, первый переданный ей аргумент — в $1, второй — в $2, и так далее. Количество переданных функции аргументов можно узнать, обратившись к переменной $#.
Аргументы передают функции, записывая их после её имени
$# — вывод количества переданных аргументов;
$* — вывод списка всех переданных аргументов;
$@ — то же, что и $* — но каждый аргумент считается как простое слово (строка);
$1 - $9 — нумерованные аргументы, в зависимости от позиции в списке.