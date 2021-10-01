# Программирование командной строки

Командная строка в UNIX-подобных системах выполняет текстовые команды, и этот текст может быть записан в виде построчной программы, пригодной к выполнению из файла.

Скрипт командной строки обычно начинается со строчки вида:

```
#!/usr/bin/env bash
```

Данная строка синтаксически является комментарием во многих языках программирования, но имеет специальное назначение в UNIX-подобных системах. Исполняемые файлы, которые начинаются с символов `#!` подразумевают запуск программы-интерпретатора, указанной после `#!`, которой передается в качестве аргумента передается имя файла скрипта.

В программы-интерпретатора должен быть указан ее полный путь. Расположение некоторых интерпретаторов, например `/bin/sh`, является стандартизированным для всех UNIX-подобных систем, для других, например bash, этот путь может быть как `/bin/bash`, так и `/usr/bin/bash`  или `/usr/bin/bash`, - гарантировать путь однозначно нельзя. Для поиска интерпретатора в переменной окружения `PATH` используется утилита `/usr/bin/env`, которая устанавливает переменные окружения, включая `PATH`, и запускает программу, переданную ей в качестве аргумента.

## Язык программирования SH

В разных Linux-системах и других UNIX-подобных системах используются различные интерпретаторы командной строки, общим предком которых является классический интерпретатор `/bin/sh`. При этом, сам интерпретатор `/bin/sh` является символической ссылкой на используемый в дистрибутиве интерпретатор по умолчанию (за исключением MacOS, где `/bin/sh` - это `bash` старой версии, в то время как в системе используется `zsh`).

Можно считать, что программа, ориентированная на `/bin/sh` может быть запущена на любой UNIX-подобной системе, но при написании таких скриптов необходимо ориентироваться на общее подмножество функциональности различных интерпретаторов, которое регламентировано стандартом POSIX. В дальнейшем мы будем использовать интерпретатор `bash`, который присутствует во всех популярных дистрибутивах Linux, и обладает широкой фунциональностью. 

Каждая команда shell-скрипта может располагаться на отдельной строке, либо заканчиваться символом точки с запятой, если необходимо записать в одну строку несколько команд.

Команды скриптов, в большинстве случаев, - это внешние программы, которые располагаются в одном из каталогов, перечисленных в переменной окружения `PATH`, но некоторые команды не могут быть реализованы как отдельные программы, поскольку изменяют текущее окружение, что не может быть сделано внешней программмой. Примерами таких команд являются:

* `cd` - изменение текущего каталога;
* `export` - делают переменную доступной дочерним процессам;
* `read` - читает текст из файла или стандартного потока ввода, и записывает результат в переменную;
* `ulimit` - устанавливает ограничения ресурсов на текущий сеанс;
* `exit` - завершает работу командного интерпретатора.

Кроме того, поскольку запуск внешних программ является ресурсозатратной операцией, в некоторых оболочках отдельные часто используемые команды реализованы как встроенные, хотя их функциональность дублируется внешними одноименными программами, например команда `echo` для интерпретаторов `bash` и `zsh`, или команда `[` для оболочки `bash` . Полный список встроенных команд для текущей оболочки можно получить командой `man builtins`.

Команды могут иметь аргументы, которые разделяютя пробелами. В случае, если аргумент должен содержать пробел, или какой-либо другой символ, имеющий специальное назначение, такой аргумент нужно заключать в кавычки. Также зарезервированные символы можно экранировать с помощью символа `\`. Список зарезервированных символов, помимо пробельных, которые нужно экранировать, или заключать в кавычки:

```
| & ; < > ( ) $ ` \ " ' * ? [ # ~ = %
```

Экранирование символа переноса строки выполняется специальным образом, что обусловлено необходимостью читабельности кода скрипта: `$'\n'`.

### Особенности синтаксиса и специальные символы

В скриптах используются три вида кавычек, которые имеют различное семантическое назначение:

* 'одинарные кавычки' - сохраняет текст без изменений;
* \`обратные одинарные кавычки\` - выполняют команду и результатом является вывод этой команды;
* "двойные кавычки" - внутри них возможно экранирование символом \, подстановка значений переменных (начинаются с символа $), и возможно выполнение команд с помощью вложенных `обратных кавычек`.

Переменные объявляются символом `=`, причем пробелы вокруг этого символа не допускаются. Использовать переменные можно в виде `$переменная` или `${переменная}`. 

```
# значение 123
var1=123
# пустое значение
var2=
# строка
var3="hello world"
# так нельзя - будет ошибка
var4 = 123

# обращение к переменным
# вывод будет: $var1 hello world
echo '$var1' "$var2" "$var3" "$var_not_exist"
```

Использование несуществующей переменной не приводит к ошибке, - будет просто пустое значение.

### Выполнение команд и получение их результата

Если необходимо сохранить вывод команды в переменную, то используется один из двух способов:

* \`обратные одинарные кавычки\`, между которыми заключена команда и ее аргументы;
* заключение к конструкцию `$()`.

Если вывод команды содержит в конце символы перевода строк, то они удаляются. 

Пример:

```
os_name=`uname -s`
arch_name=$(uname -m)
echo "OS is $os_name running on $arch_name"
# OS is Linux running on x86_64
```

Если необходимо получить код возврата команды, а не результат ее вывода, то можно использовать переменную `$?` сразу после ее выполнения. 

### Специальные переменные и аргументы

* `$?` - целочисленный код возврата последней команды;
* `$0`...`$9` - аргументы команды от 0 до 9, при этом `$0` - имя самого скрипта;
* `$#` - целочисленное значение количества аргументов;
* `$@` - список всех аргументов, начиная с первого;
* `$*` - строка, которая содержит список всех аргументов, начиная с первого.

### Объявление и вызов функций

Функции - это команды, которые доступны только из текущего скрипта, которым можно передавать аргументы, и они могут возвращать текст с помощью записи на "стандартный поток вывода".

Интерпретаторы `bash` и `zsh` поддерживают три вида синтаксиса объявлений:

```
# 1. Только имя и скобки
very_important_function() {
   # реализация функции
}

# 2. Полный синтаксис 
function very_important_function() {
   # реализация функции
}

# 3. Без скобок
function very_important_function {
   # реализация функции
}

# вызов функции с двумя аргументами, и сохранением результата
value=$(very_important_function hello world)

# вывзов функции без сохранения возвращаемого результата
very_important_function hello world
```

Если необходимо обеспечить совместимость с произвольным интерпретатором POSIX  `sh`, то можно использовать только первый вариант объявления.

Аргументы в функцию передаются точно так же, как и в команду, и доступны через специальные переменные.

Все переменные, объявляенные внутри функции, становятся доступными глобально после ее завершения. Если переменные нужно только локально, то в `bash` и `zsh` перед объявлением переменной можно использовать ключевое слово `local`.

Функции можно импортировать из другого файла, используя синтаксис `. имя_файла`.

### Перенаправление вывода

Для передачи данных от одной функции/команды к другой, не обязательно сохранять результаты в переменную, можно передавать из через механизм перенаправления, используя оператор `|`.

```
function f() {
   # вывод kek и списка аргументов
   echo "kek $*"
}

function g() {
   # замена e на E
   sed 's/e/E/g'
}

function h() {
   # замена d на первый аргумент функции
   echo $0
   sed "s/d/$1/g"
}

f first second third | g | h Meaow
# kEk first sEconMeaow thirMeaow

# команда wc -c подсчитвает количество байт
f | wc -c 
# 5

```

### Условное выполнение последовательности команд

Результатом работы команды, помимо вывода, является целочисленный код возврата, причем целое число должно быть в диапазоне от 0 до 127. Значение 0 означает успешное завершение команды, остальные значения, - признак "ошибки" или ложного значения. Код завершения предыдущей выполненной команды хранится в переменной `$?`. При объявлении функций код возврата определяется кодом возврата последней выполненной внутри функции команды, либо задается с помощью оператора `return`.

Команды можно объединять в последовательности, которые выполняются в зависимости от результата выполнения предыдущей команды:

* `cmd1 && cmd2` - `cmd2` будет выполнена, если `cmd1` выполнена успешно, а итоговый результат - это код возврата `cmd2`;
* `cmd1 || cmd2` - `cmd2` будет выполнена, если не удалось успешно выполнить `cmd1`, результат - либо значение 0, либо код возврата `cmd2`.

Выполнение цепочки команд можно заключать в круглые скобки для указания приоритетов логических операций.

```
function f() {
    echo "I'm function f"
    return 0
}

function g() {
    echo "I'm function g"
    return 5
}

function h() {
    echo "I'm function h"
    return 0
}

f && g && h    # вывод только от f и g, но не h
echo "---"
f && (g || h)  # вывод от f, g и h
```

Предусмотрены две программы, которые не делают абсолютно ничего, а только возвращают код 0 или 1: это команда `true`, и команда `false`. Они предназначены для использования внутри таких "логических выражений", например, если нужно подавить код ошибки для необязательной операции:

```
rm -f файл_который_не_существует || true  # всегда будет успешный код возврата
```

### Конструкция условия и команда [

Логические условия могут быть использованы условных конструкций, как в обычных языках программирования.

```
if true
then
   # эта часть всегда будет выполняться
fi

if false
then
   # это не будет выполняться никогда
fi
```

Аргументом команды `if` может быть любая команда. Истинным условием считается нулевой код возврата, а ложным - ненулевой. Для выполнения различных логических операций служит конструкция `[ .... ]`,  которая реализована, в общем случае, с помощью отдельной команды `[`.

* `$x -eq $y` - истина, если значения `$x` и `$y` равны; 
* `$x -nq $y` - истина, если значения `$x` и `$y` не равны; 
* `$x -gt $y` - истина, если значение `$x` > `$y`;
* `$x -lt $y` - истина, если значение `$x` < `$y`; 
* `$x -ge $y` - истина, если значение `$x` >= `$y`;
* `$x -le $y` - истина, если значение `$x` <= `$y`;
* `-n $str` - истина, если строка `$str` не пустая;
* `-z $str` - истина, если строка `$str` пустая;
* `$str1 = $str2` - истина, если строки `$str1` и `$str2` равны;
* `-e $pathname` - истина, если существует путь `$pathname`;
* `-f $filename` - истина, если существует обычный файл `$filename`;
* `-d $dirname` - истина, если существует каталог `$dirhame`;
* `-x $filename` - истина, если существует обычный файл `$filename`, и он является выполняемым.

Внутри конструкции `[ ... ]` можно использовать круглые скобки для указания приоритетов, и оператор отрицания `!`.

Важно особенностью этой конструкции является то, что между символами `[`, `]` и разными операторами внутри конструкции обязавтельно должны быть пробельные символы, поскольку это вызов команды с аргументами.

### Циклы

Простейшим циклом является цикл `while`, аргумент которого точно такой же, как у конструкции `if`.

```
while true
do
   # не только простейшая, но и самая 
   # опасная конструкция, поскольку цикл
   # может никогда не завершиться
done
```

Конструкция `for` предназначена для итерации по элементам списка.

```
# 1. Итерация по элементам простого списка
for item in i love akos
do
   echo "$item"
done

# 2. Итерация по элемента генерируемого по маске списка файлов
for filename in *.txt
do
   echo "$filename might be plain text"
done
```

Интерпретаторы `bash` и `zsh` имеют еще одну, нестандартную для POSIX `sh`, конструкцию циклов, которая синтаксически близка к Си-подобным языкам.

```
# только bash/zsh

for (( i=0; i<10; i++ ))
do
   echo "$i"
done
```

 

### Internal Field Separator

Элементы списка разделяются символом пробел в самом скрипте, если они перечислены после ключевого слова `in`, но могут также быть прочитаны из файла, либо получены из произвольной строки.

```
for item in $(echo "i love    akos")
do
  echo "$item"
done

# i
# love
# akos
```

В этом случае разделителями считаются подряд идущие последовательности пробельных символов: пробел, табуляция и символ перевода строки. Часто бывает необходимо переопределить символ разделителя, например для обработки текстовых файлов определенного формата. Для этого предназначена специальная переменная интерпретатора `IFS` (аббривеатура от Internal Field Separator).

```
IFS=💞
for item in $(echo "i💞love💞💞💞akos")
do
    echo "$item"
done

# i
# love
# 
# 
# akos
```

После переопределения символа в переменной `IFS` , разделители не группируются. В качестве разделителя можно использовать только односимвольную строку, причем интерпретаторы `bash` и `zsh` корректно обрабатывают многобайтные символы Юникода, но это не гарантируется для других интерпретаторов.

Чтение из файла можно организовать либо через команду `cat`, либо используя встроенную функцию `read` (обычно используется внутри цикла `while`), которая читает очередную лексему, ограниченную разделителем из `IFS`, и возвращает код 0, в случае успешного чтения.

### Арифметика

Командный интерпретатор `sh` позволяет вычислять произвольные арифметические выражения, но с принципиальным ограничением: допускается только знаковая целочисленная арифметика. Синтаксически операции эквивалентны таковым в других языках программирования, а сами выражения заключаются в конструкцию `$(( ... ))`. В отличии от команды `[`, круглые скобки не являются внешней командой, поэтому пробельные символы не обязательны.

```
a=5
b=3
c=$(($a+$b))
d=$(($a/$b))

echo "a = $a, b = $b, c = $c, d = $d"
# a = 5, b = 3, c = 8, d = 1
```

Для вычислений с вещественнозначными значениями можно использовать простой консольный калькулятор `bc` (Basic Calculator). Эта программа выполняет вычисление арифметических выражений, позволяет использовать функции логарифма, экспоненты и тригонометрические функции.

```
echo '(1+3)*2' | bc
# 8

# по умолчанию используется целочисленная арифметика,
# флаг -l подключает дополнительную функциональность
echo '(1+3)/2.5' | bc -l
# 1.60000000000000000000
```

### Массивы

Массивы являются нестандартным расширением  `sh`, реализованным (по-разному) в командных интерпретаторах `bash` и `zsh`.

Переменные массива объявляются как список, разделенный пробельными символами, заключенный в круглые скобки. Пустой массив обявляется как `()`.

Элементы массива можно индексировать целыми числами с 0 (для `bash`) или с 1 (для `zsh`). Индексы указываются в квадратных скобках, поэтому для исключения неоднозначности операторов, при использовании значения из массива, обязательно заключать переменную с индексом в фигурные скобки.

Адресация массива целиком (например, для вывода), а не его отдельного элемента осуществляется с указанием индекса `[@]`. Размер массива определяется как `${#массив[@]}`.

```
# пример для bash - индексация с 0

array=(1 2 3 4 5 6)
array_size=${#array[@]}

for (( i=0; i<$array_size; i++ ))
do
    # удвоенное значение
    array[$i]=$(( ${array[$i]} * 2 ))
done

echo "${array[@]}"
```

## Обработка текстов в командной строке

Задачи обработки текстов возникают очень часто, и во многих случаях для их решения совершенно не обязательно писать программы на высокоуровневых языках программирования, - можно воспользоваться стандартными утилитами среды POSIX.

### Регулярные выражения

Регулярное выражение - это текстовый шаблон, включающий в себя специальные символы-подставновки, который предназначен для поиска и замен в тексте.

Синтаксис описания регулярных выражений бывает различный, наиболее распространенный из них - это в формате языка программирования Perl, который также используется во многих других языках программирования.

Стандарт POSIX для регулярных выражений при этом является менее функциональным, и определяет два уровня языка описания: базовый (BRE) и расширенный (ERE). Расширенный синтаксис POSIX отличается от базового тем, что не требует обязательного экранирования символов скобок, а также вводит операции `?`, `+` и `|`. В дальнейшем будем использовать именно расширенный синтаксис (утилиты `sed` и `grep` требуют явного указания ключа `-E` для работы в расширенном синтаксисе).

Для тестирования регулярных выражений можно использовать веб-приложение [regex101.com](https://regex101.com), которое не поддерживает синтаксис POSIX, поэтому при написании выражений нужно не забывать о том, что не поддерживаются PCRE-специфичные конструкции, например определения классов символов через символ `\`.

Символы-подстановки, используемые в регулярных выражениях:

* `^` - начало строки;
* `.` - любой символ;
* `[   ]` - любой символ или диапазон символов, из перечисленных в квадратных скобках;
* `[^  ]` - то же самое, но с отрицанием;
* `$` - признак конца строки;
* `(   )` - группа символов или подстановок;
* `*` - повторение предыдущего символа 0 или более раз;
* `?` - повторение предыдущего символа 0 или 1 раз (только ERE);
* `+` - повторение предыдущего символа 1 или более раз  (только ERE);
* `{n}` - повторение предыдущего символа ровно `n` раз (только ERE);
* `{m, n}` - повторение предыдущего символа от `m` до `n` раз (только ERE);
* `|` - выбор одного из вариантов, между которыми встретился этот символ  (только ERE).

### Утилита grep

Утилита `grep` построчно просматривает текст из файла или стандартного потока ввода, и выполняет фильтрацию содержимого, оставляя только те строки текста, которые соответствуют шаблону.

Пример. Содержимое исходного файла `test.txt`:

```
мама мыла раму
папа кушал сидр
акос любят все
мы все умрем
физтех чемпион физтех лучше всех
```

```
# только одна строка, которая содержит слово "мама"
> grep мама test.txt
мама мыла раму

# строки, которые содержат слова "мама" и "папа"
> grep .а.а test.txt
мама мыла раму
папа кушал сидр

# все строки, которые начинаются со слова из четырех букв
> grep -E '^.{4} ' test.txt
мама мыла раму
папа кушал сидр
акос любят все
```

В последнем примере обратите внимание на следующие особенности:

* необходима опция `-E`, поскольку используется конструкция `{n}`, определенная в расширенном стандарте;
* регулярное выражение содержит символ пробела, поэтому заключено в кавычки.

Части регулярного выражения, которые заключены в круглые скобки, запоминают вхождение текста, и могут быть использованы в самом шаблоне. Эти вхождения нумеруются от `\1` до `\9`.

```
# найти все строки, в которых слово из алфавита [а-яА-Я]
# повторяется через одно слово
> grep -E '([а-яА-Я]+) .+ \1'
физтех чемпион физтех лучше всех 
```

Вместе с утилитой `grep` часто используется утилита `cut`, которая в найденной строке выбирает определенный "столбец", считая разделителем либо символ табуляции, либо какой-то произвольно заданный символ.

```
# найдем все вторые слова в строках, которые начинаются
# со слова из четырех букв
> grep -E '^.{4} ' test.txt | cut -d ' ' -f 2
мыла
кушал
любят
```

### Потоковый редактор sed

Помимо поиска, второй важный класс задач со строками, - это редактирование текста. Для автоматизации используются командные текстовые редакторы, такие как `sed` или `awk`. В отличии от обычных текстовых редакторов с пользовательским интерфейсом, потоковые редакторы оперируют набором команд редактирования. 

Команды `sed` разделяются символом `;` и выполняют одно из действий: вставка в начало, вставка в конец, удаление и замена текста. Общий вид команд: `[ПОЗИЦИЯ]ДЕЙСТВИЕ`, где `ПОЗИЦИЯ` - это необязательная часть команды, определяющая позицию курсора редактирвоания, `ДЕЙСТВИЕ` - однобуквенная команда с возможными аргументами.

Основные команды редактирования:

* `d` удаление;
* `a` добавление текста после курсора;
* `i` добавление текста перед курсором;
* `s` замена текста по шаблону.

`ПОЗИЦИЯ` описывается одним в одном из форматов:

* `ЧИСЛО` - номер строки, которые нумеруются с 1;
* `ЧИСЛО~ШАГ` - номер строки с повторением действия через определенное количество шагов;
* `$` - последняя строка;
* `/РЕГУЛЯРКА/` - все строки, сопоставленные с шаблоном.

Набор команд является обязательным позиционным аргументом для команды `sed`. Как и для утилиты `grep`, если предполагается использование расширенного синтаксиса регулярных выражений, необходим флаг `-E`. 

Если утилите `sed` не указывать имя входного файла, то подразумевается взаимодействие со стандартными потоками ввода и вывода. Если указываются файлы (их может быть несколько), то прозводится чтение из указанных файлов, причем по умолчанию используется сквозная нумерация строк по всем файлам. Для того, чтобы каждый файл обрабатывался по-отдельности, необходима опция `-s`.

Опция `-i`, также как и для `clang-format`, подразумевает сохранение изменений в исходный файл, а не вывод результата на стандартный поток вывода. Используйте эту опцию с осторожностью.

#### Примеры

```
# удалить первую и последнюю строки из файла
> sed '1d; $d' test.txt
папа кушал сидр
акос любят все
мы все умрем
```

```
# удалить все нечетные строки из файла
> sed '1~2d' test.txt
папа кушал сидр
мы все умрем
```

```
# вставить строку #!/bin/cat в начало файла
> sed '1i#!/bin/cat' test.txt
#!/bin/cat
мама мыла раму
папа кушал сидр
акос любят все
мы все умрем
физтех чемпион физтех лучше всех
```

```
# вставить пустую строку после первой строки
> sed '1a\ ' test.txt
мама мыла раму

папа кушал сидр
акос любят все
мы все умрем
физтех чемпион физтех лучше всех
```

```
# заменить слова "мама" и "папа" на "родитель"
> sed -E 's/(мама|папа) /родитель /' test.txt
родитель мыла раму
родитель кушал сидр
акос любят все
мы все умрем
физтех чемпион физтех лучше всех
```

```
# удалить все комментарии из Python-файлов текущего каталога
# (без контроля синтаксиса, в том числе из строковых констант)
> sed -i '/ *#/d' *.py
```

```
# поменять местами два первых слова в каждой строке
> sed -E 's/([а-я]+) ([а-я]+) (.*)/\2 \1 \3/' test.txt
мыла мама раму
кушал папа сидр
любят акос все
все мы умрем
чемпион физтех физтех лучше всех

# то же самое, то только для строк, начинающихся с буквы "м"
> sed -E '/^[м].*/s/([а-я]+) ([а-я]+) (.*)/\2 \1 \3/' test.txt
мыла мама раму
папа кушал сидр
акос любят все
все мы умрем
физтех чемпион физтех лучше всех
```








