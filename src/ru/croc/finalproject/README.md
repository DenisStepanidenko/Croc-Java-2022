# ВВЕДЕНИЕ

Чтобы сделать достижения, нужно проанализировать, какие метрики могут быть в нашем приложении.
В ходе анализа я выделил следующие категории:

![метрики процесса drawio (3) drawio](https://user-images.githubusercontent.com/110686828/207192889-cc3dd4b9-9046-4327-80c7-209ffe39a3c3.png)

Как считаются данные метрики определяется на уровне других микросервисов. Мой микросервис же получает доступ к БД(заполняется также другим микросервисом), где уже хранятся количественные метрики прогресса изучения каждого пользователя.

На основе данных метрик прогресса изучения были разработаны следующие метрики достижений.

![метрики достижений drawio](https://user-images.githubusercontent.com/110686828/207195896-cd0f973c-b69b-4f5a-b64a-a1c9f896a066.png)

# ТЕХНИЧЕСКОЕ ОПИСАНИЕ

## Базы данных:

Микросервис оперирует двумя базами данных H2.

Первая БД предоставляется другим микросервисом - в ней хранятся количественные метрики прогресса изучения пользователей:

![bandicam 2022-12-13 03-43-41-296](https://user-images.githubusercontent.com/110686828/207198657-596b434a-97de-4541-a645-b7ee092ea0de.jpg)


Вторая создаётся для хранения достижений:

![bandicam 2022-12-13 03-47-37-561](https://user-images.githubusercontent.com/110686828/207198970-aa465b4f-a057-4f74-8256-468a0e583bed.jpg)

## Классы и главные методы:
### Класс WorkWithAchievements 
Объекты данного класса хранят методы для обновления достижений в БД, а также для вывода прогресс-бара по всем достижениям конкретного пользователя.
#### Метод updateAchievements(int id, String metrics)
Данный метод нужен для обновления достижений. На вход принимает ID пользователя, а также конкретную метрику прогресса изучения по которой нужно обновить достижения. Этот метод будет использоваться микросервисом, который обновляет количественные метрики прогресса изучения. При этом обновлении будет вызываться данный метод, и этот метод уже проверит - нужно ли обновлять достижение по данной метрики или нет.
#### Метод getAllInformation(int id)
Данный метод используется для вывода прогресс-бара по всем достижениям пользователя с заданным ID.
### Класс DataSetAchievements
Объект данного класса хранит необходимые коллекции для выполнения задачи учёта достижений.В данных коллекциях учитываются все достижения, которые читаются из файла.
### Класс PathToTheDB
Объект данного класса хранит данные для доступа к БД с метриками прогресса изучения

# Пример использования
Представим, что микросервис, который ответственный за метрики прогресса изучения, для пользователя с ID=1 обновил данные по метрики WORDS.
То есть после этого БД с метриками прогресса выглядит так:

![bandicam 2022-12-13 12-36-01-490](https://user-images.githubusercontent.com/110686828/207307347-77fa6ccc-0d78-41de-8e12-e8e4bcce6e3b.jpg)

Обратим внимание, что в данном случае пользователь должен получить достижение за 1000 выученных слов, то есть "Начало положено".
Затем этот микросервис вызывает метод, который написан в моём микросервисе,а именно updateAchievements(int id, String metrics), куда вместо id передаёт 1, а вместо metrics - WORDS. Небольшое замечание: корректность введёного id и metrics контролируется на уровне микросервиса, который обновляет метрики прогресса изучения.

Посмотрим на то, как выполнилась программа в консоле.

![bandicam 2022-12-13 12-39-26-863](https://user-images.githubusercontent.com/110686828/207308255-6ea43fd7-3a49-4aeb-bdee-b3af2f4fd721.jpg)

Как можно заметить, программа выполнилась успешно и вывела информацию о том, что данные по достижениям обновлены.
Проверим БД, в которую записываются достижения.

![bandicam 2022-12-13 12-39-49-274](https://user-images.githubusercontent.com/110686828/207308533-37b58b26-b1b3-44c2-8563-c98961cf7de9.jpg)

Как видим, всё успешно записалось.

Теперь проверим опять БД с метриками прогресса изучения, чтобы проверить, что за это полученное достижение обновилась метрика MEGAMIND( это достижение, которое даётся за получение всех достижений).

![bandicam 2022-12-13 12-40-15-827](https://user-images.githubusercontent.com/110686828/207308842-6bf102d3-42e4-4cd6-84d9-6ba2278eeeae.jpg)


Как мы видим в метрике MEGAMIND обновилось значение и стало равным 1, так как мы получили одно достижение.

Теперь посмотрим на то, как работает метод getAllInformation(int id). Корректность введёного id гарантировано, так как при регистрации пользователя, микросервис, который за это ответственный, создаёт в БД с метриками прогресса изучения строчку с соответствующим id и заполняет изначально все метрики данного пользователя нулями. Поэтому данный id всегда найдётся в БД с метриками прогресса изучения.
Выведем на нашем примере информацию о достижениях пользователя с id = 1

![bandicam 2022-12-13 12-41-57-512](https://user-images.githubusercontent.com/110686828/207309702-b4dad96f-efc3-4adc-b581-7d1edf2c404b.jpg)

Как видим, всё успешно загрузилось.