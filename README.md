# LDT_satellite_task18_2024
Leaders of digital transformation hackathon 2024. Task 18 efficient processing algorithm for satellite images of the Russian orbital group

## Hackaton "Leaders fo digital itransrofmation" 
_Task 18_ "Efficient professing algorithm for satellite images of the Russian orbitral group"
_Team_ "Venera-9": Maria Smirnova & Denis Kalganov

## Хакатон "Лидеры цифровой трансформации"
_Задача 18_: Алгоритм эффективной обработки спутниковых снимков российской орбитальной группировки 
_Команда "Венера-9"_: Мария Смирнова & Денис Калганов
### Постановка задачи
_Задача_: нахождение и реализация высокоскоростного алгоритма привязки 4-х канального спутникового снимка(кропа) к подложке, охватывающей большее пространство и более высокого разрешения с погрешностью ~1км (подложка со стороной ~88км) . Также фильтрация выбросов/битых пикселей.
_Сложности_: геометрия и площадные размеры на кропах отличаются от подложки, отличаются масштабы (разрешение снимков), может присутствовать поворот на небольшой угол, мы, также заметили небольшое смещение гистограмм одинаковых локаций. 
### Наши подходы
К сожалению, решения не было найдено, в материале собраны проверенные командой гипотезы
Мы пришли к выводу, что **единственный инвариант классы**: угол, масштаб, разрешение всё это может меняться от снимка к снимку. Неизменным остаются только наблюдаемые объекты: лес, реки, города.

Работая над задачей мы придерживались положений:
* Цвет имеет значение набор r, g, b, nir достаточно точно определяет класс
* Единственный инвариант классы: угол, масштаб, разрешение всё это может меняться от снимка к снимку. Неизменным остаются только наблюдаемые объекты: лес, реки, города.
* Параметры изменений нам не известны

#### Анализ снимков
* Кропы имеют разный, но существенно худшие разрешение
* На изображениях кропов геометрия объектов не сохраняется
* Данные кропов имеют существенные вылеты (особенно по каналу nir)
* Гистограммы одинаковых областей разнятся у одинаковых
локаций могут быть чуть разные «цвета»

Данные представлены изображениями с 4 каналами
* Nir позволяет рассчитать основные вегетационные индексы
* На основании индексов получаются качественные маски
* Наиболее пригодные для поиск маски с редкими классами EVI & RVI

#### План решения
Исходя из анализа данных, получился примерно такой план решения:
1) Предобработка снимков. 
	1) Обработка выбросов - функция `fix` в тетрадке `pipeline.ipynb`; 
	2) Выделение классов/масок - файлы `Rivers_masks.ipynb` ; 
	3) Расчет данных по подложке.
2) Приблизительный поиск положения снимка - эта часть у нас не получилась с достаточной точностью и быстродействием. В частности мы прибегали:
	1) Расчёт графа соседствующих классов 
	2) Поиск ближайших классов по триангуляции
	3) Выделение границ и их сопоставление/ маскирования городов (скопления границ)
	4) Поиск ближайших соседей по:
		1) Маскам (NDVI, EVI, RVI & ...) / маскам (города, реки)
		2) ML-выделенным классам (мы разметили и опробовали 2-3 класса)
		3) Процентному содержанию классов на снимке
		4) Поиска минимума разностей изображений.
3) Точно позиционирование снимка на части подложки (ORB-алгоритм). Пример успешной реализации тетрадке `final_stage.ipynb`

#### Техническая реализация
Изображения и вспомогательные данные загружались в объект класса `Scene`(напр., в `Rivers_masks.ipynb`), где для удобства хранились все модификации и слои каждого изображения. *Для продуктовой реализации это не эффективно, одна очень полезно при изучении и эксперементов.*
На основании 4-х слоёв рассчитывались индексы: `NDVI`, `EVI`, `GNDVI`, `NDTI`, `RVI`, `tass_cap`(tasselled-cap/Kauth-Thomas transformation), `NDWI` - в некоторых индексах из-за отсутствия нужных каналов, подставлялись наиболее подходящие. Так `NDSI` не рассчитывался, поскольку разделения ИК-каналов не было, а иначе был схож с другим индексом.
Для каждого индекса написана функция вида `calcus_IndexName`(напр., в `Rivers_masks.ipynb`)

Для удобства просмотра изображений с 4-мя каналами были написаны:
* `imshow(image: np.array, bands: str, hist_equal: bool = False)` - принимает исходное изображение (с 4-мя каналами), буквенное написание нужных для изображения в нужном порядке каналов, а также возможность нормализации гистограммы
* `normalized_filtered(image: np.array, bands: str, hist_equal: bool = False) -> np.array` - схоже с функций выше, только возвращает массив с 3-мя каналами.
* `normalize` & `normalize_channel(channel: np.array, perc_thr: int = 100, lower_pecr_thr: int = 2)` - функции для нормализации гистограммы с возможностью указать границы обрезки гистограммы.

#### Иллюстрации
**Особенности данных**
Видны разные машстабы, разрешения и изменение геометрии
![image](https://github.com/user-attachments/assets/a686e77f-aebd-449d-b8c2-88db359cce12)

Нагляденые примеры вылетов
![image](https://github.com/user-attachments/assets/4df8f914-a465-461e-9ddd-20b7ee457876)
![image](https://github.com/user-attachments/assets/e8ec095a-ef36-45bb-b807-9a11e12d425d)


Значение расчётанных индексов и маскирование
![image](https://github.com/user-attachments/assets/d2c3d026-b356-4a20-a356-6e777af121d9)

**Стадия приблизительного поиска положения кропа**

Маскирование
![image](https://github.com/user-attachments/assets/24efbb70-0e79-4088-83e2-dcb65890d64a)

Автомаскирование по городам (по плотности границ)
![image](https://github.com/user-attachments/assets/d1b83c3e-1fd9-4d80-86d1-6816bc07bd6b)

Маскирование по контрасту:
![image](https://github.com/user-attachments/assets/4ce47552-7930-4f1e-9535-069e52bd4971)

Маскирование по ручной размеке:
![image](https://github.com/user-attachments/assets/14bff403-5488-4705-bfbc-bb4d0168c8f2)

ORB:
![image](https://github.com/user-attachments/assets/cc6c775c-f48b-4b8f-865b-dfdb5aa01759)








