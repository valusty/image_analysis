# Визначення морфометричних характеристик аксонів

## Опис використаних зображень
Для аналізу використовувалися зображення, отримані з використанням водоімерсійного об’єктиву 40 х 0,9 (_PLAPO40XWLSM, Olympus_) при розмірі пінхолу 200-400 мкм. Використовувані зображення мали розмірність 1024х1024 пікселя, один піксель дорівнював 0,317 мкм. Враховуючи багатоканальність зображення, для подальшого аналізу використовувався виключно канал з флюорисценцією NfH. У випадку використання зображень Z-stack використовувалась проєкція максимальної інтенсивності. 

Зображення зрізів периферичних нервів були оброблені та проаналізовані з використанням мови програмування Python. Були використані бібліотеки pandas (_McKiney, 2010_) , numpy (_Harris et. al., 2020_), matplotlib (_Hunter, 2007_) для роботи з даними та їх візуалізації. Для роботи з зображеннями (_завантаження, обробки, фільтрації та їх сегментації_) використовували skimage (_van der Walt et al., 2014_).

## Library import

Для початку імпротували необхідні бібліотеки:

``` bash
import matplotlib.pyplot as plt #для створення графіків та візуалізації зображень
import matplotlib.patches as patches #для додавання фігур на графіки
from matplotlib.colors import LinearSegmentedColormap #для створення власних кольорових  схем для графіків

import scipy #для проведення обчислень на основі даних, що кодують графічне зображення
from scipy import signal #для пошуку пікових точок на зображеннях (точок, з найбільшою інтенсивністю сигналу)
from scipy import stats #для проведення статистичних обрахунків 
from scipy import ndimage #для обробки багатовимірних масивів, якими є зображення

import numpy as np #для виконання числових обчислень та математичних функцій

import pandas as pd #для обробки та аналізу даних, роботи з таблицями (dataframe), імпорту та експорту даних в формі таблиці в форматі .csv

from skimage import io #для роботи з зображеннями, їх імпорту, збереження і т.п.
from skimage import filters #для обробки та фільтрації зображень, їх згладжування, виявлення країв, визначення порогових значень
from skimage import measure #для аналізу об’єктів у зображеннях
from skimage.measure import regionprops #для вимірювання властивостей зображених об’єктів, таких як наприклад площа
from skimage.measure import label #для маркування різних об’єктів на зображенні
from skimage import morphology #для проведення морфологічних операцій
from skimage import segmentation #для сегментації зображень
from skimage.segmentation import watershed #для сегментації об’єктів на основі меж між ними
from skimage.segmentation import mark_boundaries #додавання меж на зображення між різними об’єктами, що можуть дуже близько розміщуватися
from skimage.feature import peak_local_max #для знаходження місць з максимальними значеннями інтенсивності

import sympy as sp #для роботи з математичними виразами

#import oiffile as oi #для завантаження зображень в форматі .oib, наразі не потрібно, так як для вашої роботи використовуватимуться зображення в форматі .tiff
```

_Примітка_: якщо перед функцією стоїть #, як наприклад в ```#import oiffile as oi```, то вона виконуватися не буде - зручний спосіб інактивації функцій без їх видалення, може знадобитися у випадку наявності помилку та пошуку причини її виникнення, або для внесення коментарів в текст. Краще за все перед вашими формулами коментувати, що саме буде відбуватися при виконанні наступного компоненту коду для кращої читабельності та розуміння. 

## Image

Потім імпортували зображення в форматі .tiff:

``` bash
# Імпорт файлу

img_raw = io.imread("C:/шлях/до/місця/де/збережений/ваш/файл.tif")

# Перевірка форми об'єкту

print(img_raw.shape)
```

_Примітки_: 
- B ```img_raw = io.imread("C:/шлях/до/місця/де/збережений/ваш/файл.tif")```. У випадку, якщо посилання матиме формат C:шлях/до/місця/де/збережений/ваш/файл.tif, лишіть як ```img_raw = io.imread("C:/шлях/до/місця/де/збережений/ваш/файл.tif")```, якщо посилання на зображення матиме наступний вигляд: C:\шлях\до\місця\де\збережений\ваш\файл.tif, додайте r в функції, щоб вона мала наступний вигляд ```img_raw = io.imread(r"C:\шлях\до\місця\де\збережений\ваш\файл.tif")```;
- Обов’язково перевіряйте форму файлу з використанням ```img_raw.shape```. Результатом буде щось по типу (2, 31, 1024, 1024), де:
    - 2 - кількість каналів зображення;
    - 31 - кількість оптичних зрізів (або кількість шарів по вісі Z);
    - 1024, 1024 - кількість пікселів по вісям X та Y.

Потім додаються відомості щодо зображення:

``` bash
name = "example_image" #назва файлу
date = "170125" #дата отримання зображення, для тих, що доступні в гугл диску поки тільки 170125
op_date = "co" #дата операції, тут в нас буде co (control)
group = "co" #операційна група, тут в нас контрольна
paw = "co" #маркування щура, тут в нас буде co
spot = "ps" #ділянка дослідження (невдовзі я додам опис)
```

## For multichannel

З використанням ```img_raw.shape``` ми виявили, що наше зображення є мультиканальним. Для розділу нашого зображення на різні канали робимо:

``` bash
# Розділення багатоканального зображння на окремі канали

ch0_img_raw = img_raw[0]
ch1_img_raw = img_raw[1]
``` 
_Френдлі ремайндер:_ В Python нумерація починається з 0, а не з 1

І перевіряємо знову форму файлу з використанням ```img_raw.shape``` для виявлення чи коректно була використана функція:

```bash
# Перевірка форми об'єктів

print(ch0_img_raw.shape) 
print(ch1_img_raw.shape) 
```

Результатом буде щось по типу (31, 1024, 1024), де:
    - 31 - кількість оптичних зрізів (або кількість шарів по вісі Z);
    - 1024, 1024 - кількість пікселів по вісям X та Y.
Як бачите, 2 на початку пропала, через те, що ваше зображення стало одноканальним.

## Max image projection

Далі створюємо проєкції максимальної інтенсивності для окремих каналів, та візуалізуємо отримане:

``` bash
# Створення максимальних проєкцій зображення в окремих каналах

ch0_img_raw_max = np.max(ch0_img_raw, axis=0)
ch1_img_raw_max = np.max(ch1_img_raw, axis=0)

# Побудова графіка для візуалізації 

plt.figure(figsize=(12,6))

ax0 = plt.subplot(121)  # Зображення каналу 1
ax0.imshow(ch0_img_raw_max)
ax0.set_title('Ch0')
ax0.axis('off')

ax1 = plt.subplot(122)  # Зображення каналу 2
ax1.imshow(ch1_img_raw_max)
ax1.set_title('Ch1')
ax1.axis('off')

plt.suptitle('Max projection')

plt.tight_layout()"C:\Users\valusty\image_analysis\results\results_axon\segmented_outputs_axon\example_image\01_max_projection.png"
plt.show()
```
![max_projection](/results/results_axon/segmented_outputs_axon/example_image/01_max_projection.png)

_Рис. 1. Приклад максимальних проєкцій використовуваного в аналізі зображення. (А) Розподіл основного білку мієліну (myelin basic protein, MBP, для візуалізації мієлінових оболонок) в проксимальному кінці (PS) контрлатерального сідничого нерву. (B) Розподіл важкого ланцюга нейрофіламентів (Neurofilament heavy chain, NfH, для візуалізації аксонів) в проксимальному кінці (PS) контрлатерального сідничого нерву._ 

## Preprocessing

Далі проводимо корекцію фону для каналу зображення шляхом видалення 1-го процентиля інтенсивностей пікселів та обрізання від'ємних значень до нуля 

``` bash
# Корекція фонового шуму

bc_p = lambda x:np.array([f - np.percentile(f, 1) for f in x]).clip(min=0).astype(dtype=x.dtype)

ch0_img_corr = bc_p(ch0_img_raw)
ch1_img_corr = bc_p(ch1_img_raw)

# Побудова графіка для візуалізації - на прикладі першого (згідно з нумерацією в Python) каналу, який відповідає аксонам

fig, (ax0, ax1) = plt.subplots(ncols=2, figsize=(12,6))

ax0.hist(ch1_img_raw.ravel(), bins=256) # Зображення каналу 1
ax0.set_yscale('log')
ax0.set_title('Before correction')

ax1.hist(ch1_img_corr.ravel(), bins=256) # Зображення каналу 2
ax1.set_yscale('log')
ax1.set_title('After correction')

plt.suptitle('Correction of intensities')

plt.tight_layout()
plt.show()
```

![correction_plot](/results/results_axon/segmented_outputs_axon/example_image/02_correction_plot.png)

_Рис. 2. Гістограма інтенсивностей пікселів до (А) та після (В) корекції._

## Median filter

Далі застосовуємо медіанний фільтр (```filters.median()```) для зниження рівня шуму. Як кернел для медіанного фільтра  використовувався дископодібний структурний елемент. Стандартно, радіус диску становив 2 пікселі, проте даний показник міг змінюватися в залежності від оброблюваного зображення. 

Для початку обираємо необхідний нам канал після корекції:

``` bash
#channel_corr = ch0_img_corr # myelin channel
channel_corr = ch1_img_corr # axon channel
```
Потім робимо максимальну проєкцію (_знову, попередній раз був для контролю_):

``` bash
nfh_det_img_raw = np.max(channel_corr, axis=0)
```
Далі власне застосовуємо сам фільтр:

``` bash
morphology_disk_median = 2 # спробуйте різні значення кернелу, щоб подивитися, як змінюється відфільтроване зображення
nfh_det_img = filters.median(nfh_det_img_raw,
                              footprint=morphology.disk(morphology_disk_median)) # у подальшій роботі ви можете змінити filters.median на інші фільтри, або додати
```

Будуємо графік для візуалізації результату:

``` bash
plt.figure(figsize=(11,6))

ax0 = plt.subplot(121)  # зображення до обробки
ax0.imshow(nfh_det_img_raw)
ax0.set_title('Raw')
ax0.axis('off')

ax1 = plt.subplot(122)  # зображення після приміненого фільтру
ax1.imshow(nfh_det_img)
ax1.set_title('Median filter')
ax1.axis('off')

plt.suptitle('Control image')

plt.tight_layout()

#Збереження результату
#БУДЬТЕ ОБЕРЕЖНІ З НАЗВОЮ ЗОБРАЖЕННЯ ТА МІСЦЕМ КУДИ ВИ ХОЧЕТЕ ЙОГО ЗБЕРЕГТИ - ЯКЩО ВИ НЕ ВНЕСЕТЕ НІЯКИХ ЗМІН, ІСНУЮЧЕ ЗОБРАЖЕННЯ БУДЕ ПЕРЕЗАПИСАНО
#plt.savefig(r'C:\Users\USER\image_analysis\results\results_axon\segmented_outputs_axon\example_image\03_median_filter.png', format='png', dpi=300)

plt.show()
```

![median_filter](/results/results_axon/segmented_outputs_axon/example_image/03_median_filter.png)

_Рис. 3. Зображення до (А) та після (В) застосування медіанного фільтру._

## Threshold Otsu

Далі проводимо сегментацію зображення з використанням методу мульти Оцу (```filters.threshold_multiotsu()```) для розбиття зображення на різні регіони (бічна сторона аксону, верхівка аксону) на основі інтенсивності пікселів. Для подальшого аналізу використовували маску, що містила в собі дані виключно про верхівку аксону.

``` bash
# Виявлення порогових значень Оцу

thresholds = filters.threshold_multiotsu(nfh_det_img)
print(thresholds)

# Диференціація регіонів за пороговими значеннями

regions_img = np.digitize(nfh_det_img, bins=thresholds)


# Будуємо графік для візуалізації результату

plt.figure(figsize=(6,6))
plt.imshow(regions_img)  # Multi-Otsu regions
plt.suptitle('Multi-Otsu regions')
plt.axis('off')

plt.tight_layout()

#Збереження результату
#БУДЬТЕ ОБЕРЕЖНІ З НАЗВОЮ ЗОБРАЖЕННЯ ТА МІСЦЕМ КУДИ ВИ ХОЧЕТЕ ЙОГО ЗБЕРЕГТИ - ЯКЩО ВИ НЕ ВНЕСЕТЕ НІЯКИХ ЗМІН, ІСНУЮЧЕ ЗОБРАЖЕННЯ БУДЕ ПЕРЕЗАПИСАНО
#plt.savefig(r'C:\Users\USER\image_analysis\results\results_axon\segmented_outputs_axon\example_image\04_multi_otsu.png', format='png', dpi=300)

plt.show()
```

![multi_otsu](/results/results_axon/segmented_outputs_axonexample_image/04_multi_otsu.png)

_Рис. 4. Маски бічних регіонів аксону та його верхівок після сегментації зображення з використанням методу мульти Оцу._

``` bash
# Булеві маски окремих областей

length_mask_raw = regions_img == 1
top_mask_raw = regions_img == 2

# Будуємо графік для візуалізації результату

plt.figure(figsize=(11,6))

ax0 = plt.subplot(121)  # Бічні регіони - вони нас не цікавлять
ax0.imshow(length_mask_raw)
ax0.set_title('Neurofilaments, side')
ax0.axis('off')

ax1 = plt.subplot(122)  #Верхівки аксонів - ось що нам потрібно
ax1.imshow(top_mask_raw)
ax1.set_title('Neurofilaments, top')
ax1.axis('off')

plt.suptitle('Multi-Otsu separate regions')

plt.tight_layout()

#Збереження результату
#БУДЬТЕ ОБЕРЕЖНІ З НАЗВОЮ ЗОБРАЖЕННЯ ТА МІСЦЕМ КУДИ ВИ ХОЧЕТЕ ЙОГО ЗБЕРЕГТИ - ЯКЩО ВИ НЕ ВНЕСЕТЕ НІЯКИХ ЗМІН, ІСНУЮЧЕ ЗОБРАЖЕННЯ БУДЕ ПЕРЕЗАПИСАНО
#plt.savefig(r'C:\Users\USER\image_analysis\results\results_axon\segmented_outputs_axon\example_image\05_multi_otsu_separated_regions.png', format='png', dpi=300)

plt.show()
```
![multi_otsu_regions](/results/results_axon/segmented_outputs_axon/example_image/05_multi_otsu_separated_regions.png)

_Рис. 5. Маски бічних регіонів аксону (А) та його верхівок (В) після сегментації зображення з використанням методу мульти Оцу._

Далі робимо морфологічне відкриття (```morphology.opening()```) маски для зниження рівня шуму. Як основа для морфологічного відкриття використовувався дископодібний кернелу. Стандартно, радіус кернелу становив 2 пікселі, проте даний показник міг змінюватися в залежності від оброблюваного зображення.

``` bash
morphology_disk_opening = 2
nfh_mask = morphology.opening(top_mask_raw, footprint=morphology.disk(morphology_disk_opening))

# Будуємо графік для візуалізації результату

plt.figure(figsize=(6,6))


plt.imshow(nfh_mask)
plt.suptitle('Neurofilaments regions after opening')
plt.axis('off')

plt.tight_layout()

#Збереження результату
#БУДЬТЕ ОБЕРЕЖНІ З НАЗВОЮ ЗОБРАЖЕННЯ ТА МІСЦЕМ КУДИ ВИ ХОЧЕТЕ ЙОГО ЗБЕРЕГТИ - ЯКЩО ВИ НЕ ВНЕСЕТЕ НІЯКИХ ЗМІН, ІСНУЮЧЕ ЗОБРАЖЕННЯ БУДЕ ПЕРЕЗАПИСАНО
#plt.savefig(r'C:\Users\USER\image_analysis\results\results_axon\segmented_outputs_axon\example_image\06_opening_filter.png', format='png', dpi=300)

plt.show()
```

![opening_filter](/results/results_axon/segmented_outputs_axon/example_image/06_opening_filter.png)

_Рис. 6. Маска верхівок аксонів після її морфологічного відкриття._

І наостанок проводили Watershed сегментацію для розділення об'єктів, що тісно торкаються один одного.

``` bash
# Watershed cегментація

distance = ndimage.distance_transform_edt(nfh_mask.astype(bool))
local_maxi = peak_local_max(distance, footprint=np.ones((3, 3)), labels=nfh_mask)
peaks_mask = np.zeros_like(distance, dtype=bool)
peaks_mask[tuple(local_maxi.T)] = True
markers = morphology.label(peaks_mask)
labels_ws = watershed(-distance, markers, mask=nfh_mask)
ans = mark_boundaries(nfh_mask, labels_ws, color=(1, 1, 0), outline_color=(1,0,1), mode='inner', background_label=0)
```

![watershed](/results/results_axon/segmented_outputs_axon/example_image/07_watershed.png)

_Рис. 7. Маски після сегментації методом Оцу (А) та watershed сегментації (В)._

Всі наступні кроки спрямовані на аналіз та кількісне визначення властивостей сегментованих областей (```measure.regionprops()```), отриманих в результаті сегментації. Враховуючи, що ваша основна задача наразі полягає в підборі умов для успішної сегментації мієлінових оболонок, то я опишу даний момент трошки пізніше. 