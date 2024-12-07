# Абкадыров Э. П.
# Разработка систем анализа больших данных. 
# Домашнее задание №1 

# <p align="center">Задание:</p>





Необходимо написать пайплайн с помощью фреймворка Luigi.

### Порядок действий
Шаг 1
- Скачать supplementary-файлы датасета GSE68849 из GEO → cтраница датасета. Архив называется GSE68849_RAW.tar.
- Как вы можете увидеть, здесь стандартный endpoint, в который параметром подставляется название датасета.
- В задаче на скачивание имя датасета должно быть параметром.
- Хотя ссылку на скачивание можно захардкодить, более корректный подход — спарсить страницу и извлечь ссылку оттуда. Выберите вариант в зависимости от своих возможностей. Баллы за это не снижены, так как задание направлено на Luigi, а не на парсинг.
- Скачайте архив в заранее подготовленную папку. Структура папок остается на ваше усмотрение, но она должна быть одновременно удобна для чтения алгоритмом и понятна человеку.
- Скачивать можно с помощью библиотеки wget для Python.
- Если нужно запустить какой-то Bash-код, используйте библиотеку subprocess.
Шаг 2
- После скачивания в папке появится tar-архив, содержимое которого — gzip-заархивированные файлы.
- Нужно разархивировать общий архив, узнать, сколько в нем файлов и как они называются, создать под каждый файл папку и разархивировать его туда.
- Имейте в виду, что датасет может быть устроен по-другому. Например, в нем может быть другое количество файлов в архиве, наименования этих файлов также могут отличаться. Чем - универсальнее будет пайплайн, тем лучше.
- Текстовые файлы представляют собой набор из четырех tsv-таблиц, каждая из которых обозначена хедером. Хедеры начинаются с символа [. Для удобства каждую таблицу нужно сохранить в отдельный tsv-файл.
- Название файла — на ваше усмотрение. Постарайтесь сделать его максимально понятным и лаконичным.
- Вы можете написать свой код для разделения таблиц или использовать код ниже:

```shell
import io

dfs = {}
with open('GPL10558_HumanHT-12_V4_0_R1_15002873_B.txt') as f:
    write_key = None
    fio = io.StringIO()
    for l in f.readlines():
        if l.startswith('['):
            if write_key:
                fio.seek(0)
                header = None if write_key == 'Heading' else 'infer'
                dfs[write_key] = pd.read_csv(fio, sep='\t', header=header)
            fio = io.StringIO()
            write_key = l.strip('[]\n')
            continue
        if write_key:
            fio.write(l)
    fio.seek(0)
    dfs[write_key] = pd.read_csv(fio, sep='\t')
```
- В словаре dfs будет содержаться четыре дата-фрейма под соответствующими ключами.
Шаг 3
- Здесь мы видим, что таблица Probes содержит очень много колонок, часть из которых — большие текстовые поля. Помимо полного файла с этой таблицей, сохраните также урезанный файл.
- Из него нужно убрать следующие колонки: Definition, Ontology_Component, Ontology_Process, Ontology_Function, Synonyms, Obsolete_Probe_Id, Probe_Sequence.
Шаг 4
- Теперь мы имеем разложенные по папкам tsv-файлы с таблицами, которые удобно читать. Изначальный текстовый файл можно удалить, убедившись, что все предыдущие этапы успешно выполнены.
Важно!

Хорошо продумайте пайплайн: из каких задач он будет состоять, какие параметры, инпуты и аутпуты будут у каждой задачи, каким образом задачи будут зависеть друг от друга.

---
# <p align="center">Что было сделано:</p>


### ETL-пайплайн с использованием Luigi
Проект реализует ETL-процесс на базе Luigi, выполняя следующие этапы:

1. Загрузка данных
2. Распаковка архива
3. Подготовка и обработка данных
4. Сохранение результатов
5. Очистка временных файлов

### Инструкция по запуску
Шаг 1: Установка зависимостей
- Перейдите в папку проекта через терминал.
- Установите необходимые зависимости (по желанию создайте виртуальное окружение):
```shell
pip install -r requirements.txt  
```
Шаг 2: Запуск пайплайна
- Выполните следующую команду, чтобы запустить весь процесс:

```shell
python main.py FinalTask --dataset-name GSE68849 --output-dir ./output --local-scheduler  
```
- `--FinalTask` — финальная задача, запускающая все этапы пайплайна.
- `--dataset-name` — параметр, задающий имя загружаемого датасета.
- `--output-dir` — директория, куда будут сохранены результаты обработки.
- `--local-scheduler` — указывает использовать локальный планировщик, чтобы избежать запуска веб-интерфейса Luigi.

### Альтернатива: Использование веб-интерфейса

Для запуска с веб-интерфейсом выполните команду для запуска сервера Luigi в отдельном процессе:

```shell
luigid  
```
Затем запустите пайплайн без флага --local-scheduler:

```shell
python main.py FinalTask --dataset-name GSE68849 --output-dir ./output  
```
Веб-интерфейс будет доступен по адресу: http://localhost:8082.
