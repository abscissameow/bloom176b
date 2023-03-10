# Cравнение CoT и ансамблированного CoT на GSM8K с BLOOM-176B
Этот проект является продолжением исследования для модели BLOOM-7B, которое можно найти в репозитории [BLOOM-7B](https://github.com/abscissameow/bloom). В данном исследовании я сосредоточилась на сравнении двух методов генерации текста - Chain-Of-Thoughts и Self-consistency -  на датасете GSM8K с использованием модели BLOOM-176B. 


Я провела несколько экспериментов, чтобы изучить влияние составления prompt на качество генерации текста. В процессе генерации решений я использовала несколько подходов к формированию prompts. Каждый prompt состоял из вопроса и нескольких контекстных фраз, на основе которых модель генерировала ответ. Prompts формировались по-разному: фиксированные решённые задачи для всех prompts, случайный выбор решённых задач для каждого prompt и составление prompts с использованием сократовских вопросов в решении.

Я обнаружила, что случайный выбор примеров является наиболее эффективным способом генерации текста, в то время как использование сократовских вопросов является наименее эффективным.

| Способ генерации | Greedy CoT | Self-consistency: mode| Self-consistency: median| Self-consistency: mean| Сгенерированы правильные ответы |
| -------- | ------- |  ------- |  ------- |  ------- |  ------- |
| Фиксированные примеры  | 5 | 4 | 1 | 2 | в 24 задачах |
| Случайные примеры | 4 | 7 | 3 | 0 | в 25 задачах |
| Сократовские вопросы | 6 | 5 | 2 | 0| в 16 задачах |




## Выборка и метрика
Для проведения экспериментов по сравнению Chain-Of-Thoughts и Self-consistency использована выборка из 100 задач датасета GSM8K, на основе этих  задач я генерировала решения с помощью Huggingface Inference API. Для Self-consistency метода генерировались 10 сэмплов для каждой задачи.


Для выбора ответа из множества сгенерированных вариантов использовались различные метрики отбора: мода, медиана и среднее значение. Самым эффективным методом оказалась мода. Если популярных вариантов оказывалось несколько, выбирался тот, сгенерированные решения которого были по евклидовой метрике ближе к среднему по всем решениям (ранее выяснили, что "среднее" решение оказывалось достаточно близко к эталонному)


Однако, следует отметить, что самый частый ответ может быть неправильным, поэтому важно продолжать исследование и эксперименты с использованием других метрик отбора и параметров генерации.

P.S.: ранее был предложен [дисперсный подход для больших моделей](https://github.com/abscissameow/bloom#%D0%B4%D0%BB%D1%8F-%D0%B1%D0%BE%D0%BB%D1%8C%D1%88%D0%BE%D0%B9-%D0%BC%D0%BE%D0%B4%D0%B5%D0%BB%D0%B8), но из-за маленькой выборки не возникали кластеры одинаковых результатов.

## Вывод:

В заключение можно отметить, что результаты модели BLOOM-176B являются значительным улучшением по сравнению с моделью BLOOM-7B. Однако, из-за малого количества сэмплов, пока что невозможно статистически утверждать, что greedy CoT хуже, чем Self-consistency. Для статистически значимой оценки необходимо проводить генерацию ответов на большем количестве задач и с большим числом сэмплов для каждой задачи. В таблице приведены результаты  моделей со схожим (и не очень) числом параметров.

| Модель | Greedy CoT | Self-consistency|
| -------- | ------- |  ------- |
| BLOOM 7B| 2 | 1 |
| UL2 20B  | 4.1 | 7.3 |
| LaMDA 137B | 17.1 | 27.7 |
| GPT-3 175B| 14.6 | 23.4 |
| BLOOM 176B| 4 | 7 |


### （◞‸◟）  Не получилос: 


Дисперсия сгенерированных ответов оказалась велика, что можно было сделать? Например, увеличивать размер выборки (чтобы формировать кластеры и сравнивать их) и подавать больше примеров для контекста. Однако, существует ограничение на размер входных данных в Hugging Face Inference API, что делает невозможным увеличение размера prompt. Можно попробовать изменять параметры генерации (например, температуру). Но на это не хватило времени 🐾

## Файлы

Сгенерированные решения можно найти в папке `data`:

- `bloom176b/data/data_greedy/` - результаты greedy генераций для всех трёх способов

- `bloom176b/data/data_ensemble/` - в каждом файле по 10 сэмплов для каждой задачи (фиксированный prompt)

- `bloom176b/data/data_random/` - то же для случайных prompts

- `bloom176b/data/data_socratic/` - для promts с сократовскими вопросами

Генерацию текста и запись в файлы можно найти в  `generate.ipynb`

Разбираемся, что там нагенерировалось, в блокноте `answers_search.ipynb`


