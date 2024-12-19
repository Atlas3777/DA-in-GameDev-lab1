# Работа #2. Нужно больше золота
Отчет по лабораторной работе #2 выполнил:
- Афонасьев Артём Ильич
- РИ-230948

Отметка о выполнении заданий:

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 0 |
| Задание 2 | * | 0 |
| Задание 3 | * | 0 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

Структура отчета

- Данные о работе: название работы, фио, группа, выполненные задания.
- Цель работы.
- Задание 1.
- Код реализации выполнения задания. Визуализация результатов выполнения.
- Задание 2.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 3.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Выводы.
- ✨Magic ✨

## Цель работы
Научиться передавать в Unity данные из Google Sheets с помощью Python.

## Задание 1
  Выберите одну из игровых переменных в игре СПАСТИ РТФ: Выживание (HP, SP, игровая валюта, здоровье и т.д.), опишите её роль в игре, условия изменения / появления и диапазон допустимых значений. Постройте схему экономической модели в игре и укажите место выбранного ресурса в ней.
  
  
### Здоровье.

Здоровье — это показатель жизнеспособности персонажа. Если здоровье падает до 0, игра заканчивается, а волна обнуляется.
Условия изменения / появления:
#### Уменьшение:
- Удары зомби. Каждый удар отнимает 10 HP.

#### Восстановление:
- Покупка и использование аптечек.
- Вампиризм(навык).
- Регенерация(навык).
- Повышение макс. здоровья востанавливает здоровье до максимума.

#### Диапазон значений:
- Начальное здоровье: 30 HP (базовый уровень).
- Минимальное: 0(смерть)
- Максимальное: зависит от уровня прокачки макс. здоровья.

### Схема экономической модели
#### Краны:
- Убийство зомби(опыт, валюта, здоровье(если есть навык вампиризма)

#### Инвентарь
- **Здоровье**
- Количество патронов
- Имеющиеся оружия
- Очки опыта
- Имеющиеся навыки
#### Преобразователи ресурсов
- Магазин(покупка патронов, нового оружия, востановление здоровья)
- После получения уровня покупка/прокачка навыков
#### Трубы
- Трата патронов
- Потеря здоровья


## Задание 2
С помощью скрипта на языке Python заполните google-таблицу данными, описывающими выбранную игровую переменную в игре “СПАСТИ РТФ:Выживание”. Средствами google-sheets визуализируйте данные в google-таблице (постройте график / диаграмму и пр.) для наглядного представления выбранной игровой величины. Опишите характер изменения этой величины, опишите недостатки в реализации этой величины (например, в игре может произойти условие наступления эксплойта) и предложите до 3-х вариантов модификации условий работы с переменной, чтобы сделать игровой опыт лучше.

```python
import gspread
import numpy as np
import random

# Подключение к Google Sheets
gc = gspread.service_account(filename='unitydatasience-445013-5a4a164d9ecf.json')
sh = gc.open("Workshop2")  # Название таблицы
worksheet = sh.worksheet('Лист2')

worksheet.clear()
worksheet.update('A1', 'Время')
worksheet.update('B1', 'Здоровье (HP)')
worksheet.update('C1', 'Макс. хп')
worksheet.update('D1', 'Событие')
worksheet.update('E1', 'Изменение HP')

max_hp = 30
hp = max_hp
regen_value = 5
vamp_value = 8

previous_hp = max_hp
time_steps = 20  # Количество временных шагов
hp_changes = []  # Для записи изменений здоровья

data = []
event_weights = {
    "зомби": 40,       # 50% шанса
    "аптечка": 10,     # 30% шанса
    "вампиризм": 5,   # 15% шанса
    "ничего": 5,        # 5% шанса
    "увелечение макс. хп": 20,
    "регенерация": 20
}
events = list(event_weights.keys())
weights = list(event_weights.values())

for t in range(1, time_steps + 1):
    event = random.choices(events, weights=weights, k=1)[0]
    if event == "зомби":
        damage = 10  # Урон от зомби
        hp = max(0, hp - damage)  # HP не может быть меньше 0
    elif event in "аптечка":
        hp = max_hp
    elif event == "увелечение макс. хп":
        max_hp += 10
        hp = max_hp
    elif event == "регенерация":
        hp = min(max_hp, hp + regen_value)
    elif event == "вампиризм":
        hp = min(max_hp, hp + vamp_value)

    delta_hp = hp - previous_hp
    previous_hp = hp

    data.append([t, hp, max_hp, str(event), delta_hp])
worksheet.update('A2',data)
print(f"Данные успешно записаны в таблицу: {sh.url}")

```
![image](https://github.com/user-attachments/assets/a8d32d64-1c83-4a6b-9318-c5fb345a5b8b)

### Характер изменения величины:
##### Динамика здоровья:

- Здоровье снижается скачками при атаках зомби.
- Восстановление происходит периодически через аптечки, вампиризм и регенерацию, но в зависимости от их доступности.
- Если зомби атакуют часто, здоровье может быстро достигать 0, что создаёт высокий риск.
##### Недостатки реализации:
- Нечёткий баланс: Урон зомби может быть слишком слабым или слишком сильным, что делает игру либо скучной, либо слишком сложной.

### Модификации для улучшения:
 - Вместо 30 едениц здоровья сделать 3, так так урон у зрмби фиксированный - 10. Выживаемост остенется также 3 удара без прокачки, но игроку станет более понятно, сколько урона он выдержит. Под это изменить способы перки здоровье, добавивь шанс на срабатывания вампиризма(или фикс 1 хп за 15 убийств), +1 хп, реген 1 хп раз в 30 сек 
- Прогрессия сложности: Увеличивать урон зомби по мере увеличения волн, чтобы здоровье игрока стало более ценной переменной.

## Задание 3
Настройте на сцене Unity воспроизведение звуковых файлов, описывающих динамику изменения выбранной переменной. Например, если выбрано здоровье главного персонажа вы можете выводить сообщения, связанные с его состоянием.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;
using SimpleJSON;
using System;

public class work : MonoBehaviour
{
    public AudioClip goodSpeak;
    public AudioClip normalSpeak;
    public AudioClip badSpeak;
    private AudioSource selectAudio;
    private Dictionary<int, float> dataSet = new Dictionary<int, float>();
    private bool statusStart = false;
    private int i = 1;

    // Start is called before the first frame update
    void Start()
    {
        StartCoroutine(GoogleSheets());
    }

    // Update is called once per frame
    void Update()
    {
        if (dataSet.ContainsKey(i) && dataSet[i] > 0 && statusStart == false && i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioGood());
            Debug.Log($"{i} - Здоровье увеличилось на {Math.Abs(dataSet[i])}");
        }

        if (dataSet.ContainsKey(i) && dataSet[i] == 0 && statusStart == false && i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioNormal());
            Debug.Log($"{i} - Здоровье не изменилось");
        }

        if (dataSet.ContainsKey(i) && dataSet[i] < 0 && statusStart == false && i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioBad());
            Debug.Log($"{i} - Здоровье уменьшилось на {Math.Abs(dataSet[i])}");
        }
    }

    IEnumerator GoogleSheets()
    {
        UnityWebRequest curentResp = UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/14RSOAxw1mm4a4j-Xl_9-fDjyjH-trf0hZd1LNrxqT2c/values/Лист2?key=AIzaSyDz74ovY2L7TNbHbQEuAUmn9vNWFIjg4XA");
        yield return curentResp.SendWebRequest();

        if (curentResp.result != UnityWebRequest.Result.Success)
        {
            Debug.LogError($"Error fetching data: {curentResp.error}");
            yield break;
        }

        string rawResp = curentResp.downloadHandler.text;

        if (string.IsNullOrEmpty(rawResp))
        {
            Debug.LogError("Response is empty.");
            yield break;
        }

        var rawJson = JSON.Parse(rawResp);

        if (rawJson == null || !rawJson.HasKey("values"))
        {
            Debug.LogError("Invalid JSON structure or missing 'values' key.");
            yield break;
        }

        var values = rawJson["values"];

        for (int i = 1; i < values.Count; i++) // Пропуск заголовков
        {
            try
            {
                var selectRow = values[i].AsArray; // Проверяем как массив
                if (selectRow.Count >= 5) // Убедимся, что строка содержит минимум 5 элементов
                {
                    int key = int.Parse(selectRow[0]);
                    float value = float.Parse(selectRow[4]);
                    dataSet.Add(key, value);
                }
                else
                {
                    Debug.LogWarning($"Skipping row {i}, insufficient data.");
                }
            }
            catch (Exception ex)
            {
                Debug.LogError($"Error processing row {i}: {ex.Message}");
            }
        }
    }

    IEnumerator PlaySelectAudioGood()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = goodSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(1);
        statusStart = false;
        i++;
    }
    IEnumerator PlaySelectAudioNormal()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = normalSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(4);
        statusStart = false;
        i++;
    }
    IEnumerator PlaySelectAudioBad()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = badSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(1.5f);
        statusStart = false;
        i++;
    }
}
```
![image](https://github.com/user-attachments/assets/d4bedc18-2aee-434e-9201-a613e5f41b94)


## Выводы
На практике была реализована передача данных из Google Sheets в Unity с использованием Python. Для этого:  
1. Настроено взаимодействие Python с Google Sheets API через библиотеку `gspread`.  
2. Данные, описывающие игровую переменную (например, здоровье), были записаны и обновлены в таблице Google Sheets.  
3. С помощью Unity и Python было продемонстрировано, как данные из Google Sheets могут быть считаны и интегрированы в игровой процесс, например, для динамического изменения характеристик игры.



| Plugin | README |
| ------ | ------ |
| GitHub | [plugins/github/README.md][PlGh] |

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
