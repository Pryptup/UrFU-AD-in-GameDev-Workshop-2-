# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #2 выполнил:
- Лукоянов Владислав Александрович
- НМТ - 232202
  
Отметка о выполнении заданий:

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * |  |
| Задание 2 | * |  |
| Задание 3 | * |  |

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

## Цель работы
Научиться использовать google-таблицы для хранения и передачи данных в Unity с помощью Python.

## Задание 1
### Рассмотреть игровой валюты в игре "СПАСТИ РТФ: Выживание", описать её роль в игре, условия изменения / появления и диапазон допустимых значений. Построить схему экономической модели в игре.
В игре "СПАСТИ РТФ: Выживание" игровая валюта добывается двумя основными способами: 
  1) убиство противников (на старте +1 монета за зомби)
  2) просмотр рекламы перед игровой сессией (+100 монет) или, когда игрок ставит игру на паузу (+50 монет)

Вознаграждение за убиство противников можно увеличить, с помощью навыка "коллектор", который можно получить после повышения уровня, за каждое улучшение навыка количество выпадаемой валюты увеличивается на 1. За игровую валюту можно покупать новое оружие и патроны к нему, при большом количестве валюты можно купить дополнительные жизни. Количество валюты не может быть ниже 0, ограничение на увеличение валюты мной не установлено.

Далее представлена схема экономической модели игры:

![Economica](https://github.com/user-attachments/assets/b34535b9-f7fd-4612-a6a4-9f7af7f72edd)

## Задание 2
### С помощью скрипта на языке Python заполнить google-таблицу данными, описывающими игровую валюту в игре “СПАСТИ РТФ:Выживание”, визуализировать данные. Описать характер изменения валюты, описать недостатки ее реализации и предложить варианты модификации условий работы с валютой, чтобы сделать игровой опыт лучше.
Код для заполнения google-таблицы данными о накоплении внутриигровой валюты различными методами, в коде рассмторенно прибавление монет за две волны (wave1 и wave2):
```python
import gspread
import numpy as np

gc = gspread.service_account(filename='unity-data-sience-4ad4e2801be5.json')
sh = gc.open("SpastiRTF-money")
watchadbef = np.random.randint(0, 2) #Смотрел ли рекламу игрок до сессии
watchadinw1 = np.random.randint(0, 2) #Смотрел ли игрок рекламу во время прохождения 1 волны
watchadinw2 = np.random.randint(0, 2) #Смотрел ли игрок рекламу во время прохождения 2 волны
wave1 = list(range(1,10)) #В первую волну зомби 10 
wave2 = list(range(1,20)) #Во вторую их уже 20

summa = 0
if watchadbef == 1:
    summa += 100
if watchadinw1 == 1:
    summa += 50
i = 0
while i <= len(wave1):
    i += 1
    kprize = np.random.randint(0, 2) #Подобрал ли игрок монету с убитого врага
    if i == 0:
        continue
    else:
        if kprize == 1:
            summa += 1

        summa = str(summa)
        summa = summa.replace('.', ',')
        sh.sheet1.update(('A' + str(i+1)), int(summa))
        print(summa)
        summa = int(summa)
sh.sheet1.update(('A' + str(i + 10 + 1)), int(summa))

bonus = np.random.randint(0, 2) #Взял ли игрок бонус на дополнительные монеты за убийство противника
if watchadinw2 == 1:
    summa += 50
i = 0
while i <= len(wave2):
    i += 1
    kprize = np.random.randint(0, 2) #Подобрал ли игрок монету с убитого врага

    if i == 0:
        continue
    else:
        if kprize == 1 and bonus == 0:
            summa += 1
        if kprize == 1 and bonus == 1:
            summa += 2

        summa = str(summa)
        summa = summa.replace('.', ',')
        sh.sheet1.update(('B' + str(i+1)), int(summa))
        print(summa)
        summa = int(summa)
```
Переменная kprize нужна для подсчета собраных за убийства противников монет, в данном случае подберет игрок монету (значение 1) или пропустит (значение 0) выбирается случайно, т.е. случайно либо прибавляется 1 монета к сумму всех монет, либо сумма остается той же. Начиная со 2 волны вступает в силу переменная bonus, если игрок взял бонус на дополнительные монеты за убийство противника, что тоже выбирается случайно (пренебрегая тем, что уровень повысился бы еще в конце 1 волны, и повышается в конце 2 волны). Переменные watchadbef, watchadinw1 и watchadinw2 отвечают за вероятность просмотра игроком рекламы (причем рекламу во время волны на 50+ монет условно можно посмотреть всего 1 раз за волну, а рекламу на +100 монет до игровой сессии также можно посмотреть единожды). 

Приведу диаграмму одного из вариантов накопления монет:

![Скриншот 12-10-2024 194707](https://github.com/user-attachments/assets/88f273b5-6b01-4e2e-b91a-a4e522caf701)

На диаграмме видно, что основной прирост валюты принес не игровой процесс, а просмотор рекламы, что определенно является проблемой, ведь оказывается, что игроку выгоднее посмотреть много рекламы и приобрести предметы быстрее, чем надолго застрять на первых волнах, очень медленно собирая монеты, таким образом игровой процесс начинает меньше услекать и приносить меньше удовольствия. Кроме этого, даже если игрок накопит много монет используя любые способы, он безвозвратно их потеряет, если потратит на новое вооружение и погибнет.
Для решения этих проблем я могу привести следующие методы решения:

- Увеличить стартовое вознагрождение за убийство противников до 5 монет, а также полностью исключить просмотр рекламы во время игры, чтобы игрок не отвлекался на просмотр рекламы во время игрового процесса, к тому же монет уже и так копить проще
- При этом стоит ограничить навык на прирост вознагрождения за убийство 10-ю улучшениями, чтобы прирост монет в какой-то момент не стал слишком большим, а также чтобы игрок сосредоточился на улучшении других навыков, тем более до 10 ур. улучшения нужно также прилично накопить уровень, так что игрок не почувствует потолок сразу
- В конце концов, по моему мнению стоит сохранять приобретенное игроком оружие, но сбрасывать количество патронов, валюты и модификатора здоровья, таким образом у игрока будет сохраняться чувство постоянного прогресса, благодаря новому оружию ему будет легче копить больше валюты на оружие дороже, при этом он не сможет накопить слишком много, даже если приобретет все вооружение, т.к. после смерти придется заново копить на увеличение здоровья, а увеличение зомби в последующих волнах будет требовать больших трат на патроны

## Задание 3
### Настроить на сцене Unity воспроизведение звуковых файлов, описывающих динамику игровой валюты.

 
## Выводы
В ходе выполнения работы были получены навыки работы с google-таблицами при помощи python, передачи данных через них в Unity, также была возможность продумать экономику игры на примере "СПАСТИ РТФ: Выживание".
