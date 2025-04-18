# Прогнозирование уровня удовлетворённости сотрудников и их оттока 

Компания **«Работа с заботой»** стремится минимизировать финансовые риски и снизить отток сотрудников. Для этого были предоставлены данные с характеристиками сотрудников компании. Среди них — уровень удовлетворённости сотрудника работой в компании. Эту информацию получили из форм обратной связи: сотрудники заполняют тест-опросник, и по его результатам рассчитывается доля их удовлетворённости от 0 до 1, где 0 — совершенно неудовлетворён, 1 — полностью удовлетворён. 

Необходимо помочь бизнесу:  
1. **Предсказать уровень удовлетворённости работой**
2. **Прогнозировать риск увольнения**

# Вывод проделанной работы

**Задача 1 - предсказание уровня удовлетворённости сотрудника:**

1. ETL:
    - Пропуски в признаках `dept` и `level` были заменены на самые частые значения, используя пайплайн. 
    - Были найдены неизвестные категории в виде пробелов, которые были предобработаны в пайплайне для подготовки данных.

2. EDA:
    - За последний год в компании у 120 (3%) сотрудников было повышение, 559 (14%) сотрудников нарушали трудовой договор. 
    - Чаще всего сотрудники имеют уровни занимаемой должности 'junior' - 47% и 'middle' - 44%, и имеют среднюю загруженность - в 52%.
    - 75% сотрудников работают в компании менее 6 лет. 
    - 113 сотрудников имеют минимальную зароботную плату - 12000. 
    - Отдел 'sales' имеет наибольшее количество сотрудников - 1518 из 4000. 
    - Статистически тестовые данные имеют очень близкие распределения к тренировочным.
    - Оценка руководителя тесно связана с удовлетворенностью сотрудника. *Можно добавить в форму опроса сотрудников оценку их отношений с руководителем.* Так как утечка кадров может оказаться связана с плохими отношениями на работе.
    -  Высокая мультиколлинеарность:
        - `workload` и `salary` - 0.79
        - `level` и `salary` - 0.72
    -  Связь с целевым:
        - `supervisor_evaluation` - 0.76
        - `last_year_violations` - 0.56
        - `employment_years` - 0.33

3. Подготовка данных и обучение моделей:
    - Оценка качества работы сотрудника руководителем имеет наибольшую значимость среди всех признаков модели. 
    - Модель линейной регрессии показала себя хуже, она склонна к ошибкам, её значение SMAPE (28.27) на КВ превышает указанный критерий успеха заказчиком. 
    - Гиперпараметры модели дерева решений были подобраны, используя байесовскую опитимизацию. Модель имеет высокую предсказательную способность, ей не страшны коллинеарные входные признаки, скорее всего это сыграло большую роль, так как признак `salary` второй по значимости признаков в модели.
        - Параметры лучшей модели: max_depth=27, min_samples_leaf=4, min_samples_split=3
        - Лучшая средняя метрика SMAPE на валидации: 14.21
        - Метрика SMAPE на тестовой выборке: 13.58
        - Кэффициент детерминации: 0.88
    - Все признаки имеют влияние на предсказательскую способность модели, самые важные признаки:
        - `supervisor_evaluation` (самый влияющий)
        - `salary`
        - `last_year_violations`
        - `level`


**Задача 2 - предсказание увольнения сотрудника из компании:**

1. ETL:
    - Датасеты не нуждались в предобработки данных, они не имеют пропусков, дубликатов и неизвестных категорий.

2. EDA:
    - Тестовый набор данных целевого признака **`test_target_quit`** имеет схожее распределение с целевым тренировочного датасета с явным дисбалансом классов, минорным классом 'yes' и мажорным - 'no'. 
    -  Высокая мультиколлинеарность:
        - `workload` и `salary` - 0.79
        - `level` и `salary` - 0.75
    -  Связь с целевым:
        - `employment_years` - 0.66
        - `salary` - 0.56
        - `level` - 0.31
    - Портрет уволившегося сотрудника:
        - Отдел не имеет влияния на увольнения сотрудников
        - В большинстве случаев, уволившиеся сотрудники имели стаж менее 3-х лет в компании. Но бывали и случаи, когда сотрудники уходили с 10 летним опытом. Такая утечка критична для компании.
        -  Уволившиеся сотрудники имеют больший процент по количеству нарушений трудового договора и меньший по количеству повышений.
        - Средняя оценка руководителей для сотрудников у уволившихся меньше, чем у тех, кто продолжает работать в компании
        - На протяжении 5 лет сотрудники, которые уволились из компании, получали в среднем заработную плату меньше не зависимо от рабочей нагрузки. 
        - При малых значениях уровня удовлетворенности сотрудника работой в компании есть малая вероятность, что он уволится. При больших значениях данного признака, сотрудники чаще оставались в компании, чем увольнялись.
    
3. Подготовка данных и обучение моделей:
    - Был добавлен признак с уровнем удовлетворенности сотрудника работой в компании лучшей моделью дерева решений.
    - Так как новый признак имел мультиколлинеарность со всеми входными, то были выбраны нелинейные модели.
    - Избавились от дисбаланса классов, используя RandomOverSampler увеличили минорный класс.
    - Модели **KNeighborsClassifier** и **DecisionTreeClassifier** имеют похожие метрики на кросс-валидации.
    - После проведения анализа важности признаков в моделях, были отобраны признаки: 
        - `employment_years`
        - `job_satisfaction_rate`
        - `supervisor_evaluation`
        - `level`
        - `workload`
        - `salary`
    - Сложности моделей были облегчены с отобранными признаками, но не удалось значительно увеличить метрику ROC-AUC. Все модели примерно одинаковые по предсказательской спобности
    - Выбрана модель **DecisionTreeClassifier**
        - Гипермараметры:
            - max_depth: 15
            - min_samples_split: 20
            - min_samples_leaf: 2
        - Метрики на тестовых данных:
            - ROC-AUC - 0.92
            - f1 - 0.81
    
## Рекомендации

Группа-риска: сотрудники с низкой оценкой руководителя, стажем менее 3 лет и зарплатой ниже среднего. \
Оценка руководителя `supervisor_evaluation` оказалась важным фактором в обеих задачах, рекомендую внедрить регулярные опросы сотрудников об отношениях с руководителями. \
Большая часть утечки происходит на ранних стадиях работы в компании, рекомендую пересмотреть систему грейдов и заработных плат для  junior-сотрудников. Можно также добавить программы адаптации, дополнительные курсы для новичков.
