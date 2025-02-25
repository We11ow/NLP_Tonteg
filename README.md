# Введение
Cтремительное развитие социальных медиа, онлайн-платформ, а также огромное количество текстов, создаваемых и обменивающихся пользователями каждый день, делают необходимым понимание того, какие эмоции и оттенки содержатся в текстовых данных. Определение эмоциональной тональности текста позволяет анализировать мнение и реакцию пользователей.
# Проблематика
Определение уровня точности и объективности в определении эмоционального окраса текста. Это важно для понимания общественного мнения и реакции на различные события.

# Анализ на эмоции текста и голоса при помощи Aniemore
Aniemore - это библиотека для Python, которая позволяет добавить в ваше программное обеспечение возможность определять эмоциональный фон речи человека, как в голосе, так и в тексте. Для этого в библиотеке разработано два соответсвующих модуля - Voice и Text.

Показатели моделей в разрезе эмоций

![model_sota](https://github.com/user-attachments/assets/9a9b75e3-1396-4651-ae9e-b274f67dd2ea)

Установка
```javascript copy
pip install aniemore
```
Распознования эмоций в тексте
```javascript copy
import torch
from aniemore.recognizers.text import TextRecognizer
from aniemore.models import HuggingFaceModel

model = HuggingFaceModel.Text.Bert_Tiny2
device = 'cuda' if torch.cuda.is_available() else 'cpu'
tr = TextRecognizer(model=model, device=device)

tr.recognize('это работает? :(', return_single_label=True)
```
Распознование эмоций в голосе
```javascript copy
import torch
from aniemore.recognizers.voice import VoiceRecognizer
from aniemore.models import HuggingFaceModel

model = HuggingFaceModel.Voice.WavLM
device = 'cuda' if torch.cuda.is_available() else 'cpu'
vr = VoiceRecognizer(model=model, device=device)
vr.recognize('/content/ваш-звуковой-файл.wav', return_single_label=True)
```
-----
Дополнительная информация по Animore: https://github.com/aniemore/Aniemore?ysclid=m66s4m6mk9436825344
# Анализ сентимента и эмоционального окраса текстов с помощью SQL
Определение положительных, отрицательных и нейтральных текстов

Для начала, создадим таблицу в базе данных, в которой будем хранить текстовые данные и их тональность.

```javascript copy
CREATE TABLE TextSentiments (
    id INT PRIMARY KEY,
    text VARCHAR(500),
    sentiment VARCHAR(10)
);
```

Классификация текста по ключевым словам или выражениям, свидетельствующим о положительной или отрицательной тональности.

```javascript copy
SELECT id, text
FROM TextSentiments
WHERE text LIKE '%SmartGadget%' AND (text LIKE '%замечательно%' OR text LIKE '%прекрасно%' OR text LIKE '%восхитительно%');
```
------
Подсчитать общее количество положительных, отрицательных и нейтральных текстов:

```javascript copy
SELECT sentiment, COUNT(*) AS count
FROM TextSentiments
GROUP BY sentiment;
```
Пример SQL-запроса для расчета общей эмоциональной окраски текстов:

```javascript copy
SELECT SUM(
    CASE
        WHEN sentiment = 'Положительный' THEN 1
        WHEN sentiment = 'Отрицательный' THEN -1
        ELSE 0
    END
) AS total_sentiment
FROM TextSentiments;
```

|id|Текст| Эмоция |
|-|-|-|
|1|Продукт просто замечательный!| Положительный |
|2|Этот гаджет оставляет желать лучшего...| Отрицательный |
|3|Нормальеый продукт, добавить нечего | Нейтральный |

Такой запрос вернет нам следующий результат:
|Эмоция| Количество |
|-|-|
|Положительный| 1 | 
|Отрицательный| 1 |
|Нейтральный | 1 |

Этот результат показывает общее количество отзывов в каждой категории тональности.

----
Использование словарей для анализа эмоционального окраса.

Таблица, которая будет содержать слова и их эмоциональные оценки.
```javascript copy
CREATE TABLE EmotionDictionary (
    word VARCHAR(100) PRIMARY KEY,
    emotion VARCHAR(20)
);

INSERT INTO EmotionDictionary (word, emotion)
VALUES
    ('радость', 'положительная'),
    ('грусть', 'отрицательная'),
    ('удовлетворение', 'положительная'),
    ('страх', 'отрицательная'),
    ('интерес', 'положительная'),
    ('ненависть', 'отрицательная');
```

|Слово |Эмоция|
|-|-|
|радость| положительная|
|грусть| отрицательная
|удовлетворение| положительная
|страх| отрицательная
|интерес| положительная
|ненависть |отрицательная

|Текст|id|
|-|-|
|Этот фильм был действительно забавным!| 1 |
|Утро началось с хороших новостей.| 2 |
|Не могу сказать, что я был впечатлен. | 3 |

В данном примере используется классификация текста по ключевым словам совместно с словарем эмоций.

-----------
Примеры применения

Предположим, у нас есть следующие отзывы:
|id| id заказа |Текст | Эмоция| 
|-|-|-|-|
|1|101|Замечательные наушники, отличное качество звука!|Положительный|
|2|103| Не покупайте, ужасное качество...|Отрицательный |
|3|101 | Средний продукт, не впечатлил.| Нейтральный |
|4|105|	Просто влюблена в этот продукт!| Положительный |

 Расчет общей эмоциональной окраски:
 ```javascript copy
 CREATE TABLE EmotionDictionary (
    word VARCHAR(100) PRIMARY KEY,
    emotion VARCHAR(20)
);

INSERT INTO EmotionDictionary (word, emotion)
VALUES
    ('замечательные', 'положительная'),
    ('влюблена', 'положительная'),
    ('ужасное', 'отрицательная'),
    ('средний', 'нейтральная'),
```
Теперь мы можем использовать этот словарь для расчета общей эмоциональной окраски каждого продукта:
 ```javascript copy
SELECT cr.product_id, 
       SUM(
           CASE
               WHEN ed.emotion = 'положительная' THEN 1
               WHEN ed.emotion = 'отрицательная' THEN -1
               ELSE 0
           END
       ) AS total_emotion
FROM CustomerReviews cr
JOIN EmotionDictionary ed ON cr.review_text LIKE '%' || ed.word || '%'
GROUP BY cr.product_id;
```
В этом запросе мы используем словарь эмоций для связывания слов из отзывов с соответствующими эмоциями. Затем, суммируя значения эмоций, мы получаем общий эмоциональный окрас для каждого продукта.
-----
Оригинальная статья с разбором анализа SQL: https://habr.com/ru/companies/otus/articles/757042/

# TextBlob: Упрощенная обработка текста

TextBlob — это библиотека Python для обработки текстовых данных. Она предоставляет простой API для решения распространённых задач обработки естественного языка (NLP), таких как определение частей речи, извлечение именных групп, анализ тональности, классификация и многое другое.

Установка
 ```javascript copy
$ pip install -U textblob
$ python -m textblob.download_corpora
```
Пример работы:
 ```javascript copy
from textblob import TextBlob

text = """
The titular threat of The Blob has always struck me as the ultimate movie
monster: an insatiably hungry, amoeba-like mass able to penetrate
virtually any safeguard, capable of--as a doomed doctor chillingly
describes it--"assimilating flesh on contact.
Snide comparisons to gelatin be damned, it's a concept with the most
devastating of potential consequences, not unlike the grey goo scenario
proposed by technological theorists fearful of
artificial intelligence run rampant.
"""

blob = TextBlob(text)
blob.tags  # [('The', 'DT'), ('titular', 'JJ'),
#  ('threat', 'NN'), ('of', 'IN'), ...]

blob.noun_phrases  # WordList(['titular threat', 'blob',
#            'ultimate movie monster',
#            'amoeba-like mass', ...])

for sentence in blob.sentences:
    print(sentence.sentiment.polarity)
# 0.060
# -0.341
```
------
Анализ настроений

Свойство sentiment возвращает именованный кортеж вида Sentiment(polarity, subjectivity). Показатель полярности — это число с плавающей точкой в диапазоне [-1,0, 1,0]. Субъективность — это число с плавающей точкой в диапазоне [0,0, 1,0], где 0,0 — это очень объективный показатель, а 1,0 — очень субъективный.
 ```javascript copy
testimonial = TextBlob("Textblob is amazingly simple to use. What great fun!")
testimonial.sentiment
Sentiment(polarity=0.39166666666666666, subjectivity=0.4357142857142857)
testimonial.sentiment.polarity
0.39166666666666666
```
-----
Токенизация

Вы можете разбивать текстовые блоки на слова или предложения.
 ```javascript copy
zen = TextBlob(
    "Beautiful is better than ugly. "
    "Explicit is better than implicit. "
    "Simple is better than complex."
)
zen.words
WordList(['Beautiful', 'is', 'better', 'than', 'ugly', 'Explicit', 'is', 'better', 'than', 'implicit', 'Simple', 'is', 'better', 'than', 'complex'])
zen.sentences
[Sentence("Beautiful is better than ugly."), Sentence("Explicit is better than implicit."), Sentence("Simple is better than complex.")]
```

Словоизменение и лемматизация слов

Каждое слово в TextBlob.words или Sentence.words — это Word-объект (подкласс unicode) с полезными методами, например, для изменения слов по падежам.
 ```javascript copy
sentence = TextBlob("Use 4 spaces per indentation level.")
sentence.words
WordList(['Use', '4', 'spaces', 'per', 'indentation', 'level'])
sentence.words[2].singularize()
'space'
sentence.words[-1].pluralize()
'levels'
```
Слова можно лемматизировать, вызвав метод lemmatize.
 ```javascript copy
from textblob import Word
w = Word("octopi")
w.lemmatize()
'octopus'
w = Word("went")
w.lemmatize("v")  # Передавать в WordNet часть речи (глагол)
'go'
```
------
Более подробный разбор анализа настроения: https://textblob.readthedocs.io/en/dev/quickstart.html#sentiment-analysis

Актуальная инофрмация по TextBlob: https://textblob.readthedocs.io/en/dev/
# Заключение
Мы разобрали некоторые методы и модели, их точность и объективность в определении эмоционального окраса текста. Смогли разделить эмоции и мнения людей на классы, создав по ним словари.

Над статьей работали Иванов Никита и Шемякин Матвей.
