# Генерация стихов в стиле определенного автора.
## Описание задачи
Вход - строка либо четверочтишие \
Выход - сгенерированное стихотворение

Датасет – сборник стихов Есенина. 
## Описание проделанной работы

Изначально была идея попробовать добиться результатов используя стандартные нейросетевые архитектуры. Очевидно, что нужно было выбрать архитектуру с «памятью». Выбор пал на RNN так как :

* Стихи представляют собой последовательность слов или символов.
* RNN спроектированы для работы с последовательными данными, благодаря своей способности запоминать предыдущее состояние и использовать его для обработки следующего входа.

### Проделанные шаги

1. Взята библиотечная реализация (со стандартными параметрами) и дообучена на 5к итерациях на стихах Есенина
2. Для улучшения результатов были опробованы:
+ Сетка параметров (количество слоёв, размер бача, оптимизатор)
+ Использование других embedding-ов (label encoder, one hot encoder, bag of words, word2vec)
+ Изменение температуры

Результаты полученные в ходе улучшения модели не привели к желаемым результатам, поэтому было решено изменить модель на open-source модель ruGPT-3.5, т.к.

* Модель хорошо умеет работать с русским языком, что необходимо для сохранения рифмы в стихотворениях
* Есть готовая квантованная реализация на hugging face

### Проделанные шаги

1. Протестирована реализация из коробки. С различными промптами (few shots)
3. Произведена попытка дообучить модель, провалившаяся из-за особенностей взаимодействия квантованной модели с дефолтным классом Trainer
4. Принято решение использовать модель LoRa для дообучения
+ Была использована готовая реализация (https://github.com/qwopqwop200/gptqlora)
+ Изменен стандартный метод библиотеки, т.к. gptqlora не поддерживала кастомные датасеты
+ Видоизвменен формат датасета для корректной работы
5. Произведено дообучение с помощью LoRa на 8-и эпохах

## Полученные результаты
Жирным шрифтом выделены данные, подаваемые на вход.
### RNN (дефолтные параметры, temp=0.5)
__пускай я порою от спирта вымок__ \
ты с утравот неблистой пощенный стары. в солько любать на пона, \
на крого тек у стариваю сыль. \
разалянье проговет.

### RNN
__пускай я порою от спирта вымок__ \
не спаивать сердце стравит страдать с половиной \
пускай не люблю жизть \
лелея весну

### ruGpt(few shot)


__Приветствую тебя, пустынный уголок,__ \
__Приют спокойствия, трудов и вдохновенья,__ \
__Где льется дней моих невидимый поток__ 
На лоне счастья и забвенья.\
Отец семейства\
Моя мать\
И неженка Наташа, \
Дошкольник крупный Федька \
И психолог Саша

 ### ruGpt(LoRa)

__Серые глаза — рассвет,__ \
__Пароходная сирена,__ \
__Дождь, разлука, серый след__ \
Женское сердце — всегда врастопыр: \
То тянется к тени, тая намеренье, \
То к свету стремится, желая до дыр.. \
Там как-то нашло на неё вдохновенье

## Оценка полученных результатов
Для оценки сгенерированных стихотворений воспользуемя GPT, с просьбой оценить их по следующим критериям
* Рифма
* Содержание
* Грамматическая корректность

Для наглядности в первую очередь оценим наихудший из всех полученных результатов

Input: \
пускай я порою от спирта вымок \
ты с утравот неблистой пощенный стары. \
 в солько любать на пона,\
на крого тек у стариваю сыль. \
разалянье проговет.

ChatGPT: \
Ваш текст содержит опечатки и неполные предложения, что затрудняет его понимание. Я предполагаю, что вы хотите выразить какие-то мысли, используя стихотворную форму. Если у вас есть конкретные исправления или уточнения, которые вы хотели бы внести, пожалуйста, предоставьте их, и я постараюсь вам помочь.

Видим, что ИИ вовсе не может оценить сгенерированное "стихотворение". Попробуем оценить последний полученный результат.

Input:\
Серые глаза — рассвет,\
Пароходная сирена,\
Дождь, разлука, серый след\
Женское сердце — всегда врастопыр:\
То тянется к тени, тая намеренье,\
То к свету стремится, желая до дыр..\
Там как-то нашло на неё вдохновенье


ChatGPT
Рифма: 9/10

Первые три строки (рассвет - сирена - след) образуют абба рифму, что придает стихотворению мелодичность. В последних трех строках (врастопыр - намеренье - дырь - вдохновенье) также присутствует четкая абба рифма, что поддерживает единообразие.
Содержание: 9/10

Стихотворение изображает образ женского сердца, которое постоянно колеблется между светом и тенью, между тягой к нежности и стремлением к свету. Эти двойственные описания создают интересный образ, где серое сердце находит вдохновение в различных аспектах жизни.
Грамматическая корректность: 10/10

Грамматически стихотворение выполнено корректно. Фразы легко читаются, и структура предложений является грамматически правильной.
Общая оценка: 9.33/10
Стихотворение имеет хорошую рифму, содержательность и литературное качество.


