# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]



Отчет по лабораторной работе #5 выполнил:
- Исмагилов Денис Рустамович
- РИ210945
Отметка о выполнении заданий (заполняется студентом):


| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | # | 60 |
| Задание 2 | # | 20 |
| Задание 3 | # | 20 |


знак "*" - задание выполнено; знак "#" - задание не выполнено;


Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.




## Цель работы
 - Интеграция экономической системы в проект Unity и обучение ML-Agent.


## Задание 1
 - Изменить параметры файла .yaml-агента и определить какие параметры и как влияют на обучение модели.. 

## Ход работы:
Исследуем проект, который нам предоставили. 
 - В модели реализовано передвижение Агента RollerAgent между 
двумя объектами: Target и GoldMine. Модель представляет собой 
упрощенный симулятор добычи ресурсов и их транспортировки.
 - Направление движения Агента определяется переменными скрипт-файла Move.cs в инспекторе свойств:
 - Модель обучения Агента, как и ранее определяется скрипт-файлами, 
импортированными из ml-agents-release_19: Decision Requester и 
Behavior Parameters. 
 - В скрипт-файле Decision Requester устанавливается период 
принятия решения, равный 10. Это значит, что цикл наблюдение-решение-действие-вознаграждение повторяется каждый раз, когда 
агент запрашивает решение. Агенты будут запрашивать решение при 
вызове Agent.RequestDecision(). Через период Decision Period = 10.
Бывают ситуации, когда нужно, чтобы агент запрашивал решения 
самостоятельно через регулярные промежутки времени. В этом 
случае мы можем использовать в коде компонент Decision Requester 
в GameObject агента.
 - Также обратитим внимание на настройки параметров поведения 
агента Behavior Parameters. Здесь Vector Observation определяется 
параметрами: Space Size - длина наблюдения вектора для Агента. 
Stacked Vectors - Количество предыдущих векторных наблюдений, 
которые будут суммироваться и совместно использоваться для 
принятия решений. Другими словами, Vector Observation определяет 
эффективный размер векторного наблюдения, передаваемого в 
политику, составляет: Space Size x Stacked Vectors. 
 - Action (действия) определяют количество одновременных 
непрерывных действий (непрерывные действия — Continuous
Actions), которые может выполнять агент. Дискретные ветви — 
массив целых чисел, определяющий несколько одновременных 
дискретных действий. Значения в массиве Discrete Branches 
соответствуют количеству возможных дискретных значений для 
каждой ветви действия.
 - Скрипт-файл Move.cs описывает поведение при движении Агента. А 
также его вознаграждение и обучение:

|Функция в скрипт-файле Moce.cs|Пояснение работы функции|
|-|-|
|public override void CollectObservations(VectorSensor sensor)|В этой функции задаются параметры, на основе которых обучается Агент (сенсоры агента)|
|public override void OnActionReceived(ActionBuffers actionBuffers)|В этой функции описываются основные действия и взаимодействия Агента с внешним миром. А также его «вознаграждение» за успешное или не успешное обучение.В нашем примере в эту функцию входят условия передвижения агента, и расчет темпа инфляции.//стандартная функция Агента|
|IEnumerator StartGoldMine()|Coroutine – сопрограмма, которая выполняется параллельно выполняемому скрипту.В нашем случае эта функция отвечает за старт добычи «золота в шахте».|
|IEnumerator StartMonth()|Coroutine – сопрограмма, которая выполняется параллельно выполняемому скрипту.В нашем случае эта функция отвечает за отсчет месяца (фоновый счетчик).|

Напишем .yaml файл для конфигурации обучающего тренажера.

```yaml
behaviors:
  Economic:
    trainer_type: ppo
    hyperparameters:
      batch_size: 1024
      buffer_size: 10240
      learning_rate: 3.0e-4
      learning_rate_schedule: linear
      beta: 1.0e-2
      epsilon: 0.2
      lambd: 0.95
      num_epoch: 3      
    network_settings:
      normalize: false
      hidden_units: 128
      num_layers: 2
    reward_signals:
      extrinsic:
        gamma: 0.99
        strength: 1.0
    checkpoint_interval: 500000
    max_steps: 750000
    time_horizon: 64
    summary_freq: 5000
    self_play:
      save_steps: 20000
      team_change: 100000
      swap_steps: 10000
      play_against_latest_model_ratio: 0.5
      window: 10
```

Поместим файл Economic.yaml в папку с проектом Unity. В нем описываются, какой тренажер использовать: PPO или SAC (парметр trainer_type). Количество опыта, которое необходимо собрать, прежде чем создавать и отображать тренировочную статистику (параметр summary_freq, он определяет детализацию графиков в Tensorboard), и так далее.
