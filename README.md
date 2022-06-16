# Карта вулканов на территории США

см. файл _map2.py_


<a id ="anchor"></a>
___
Содержание:
* [1. Создание карты](#map)
* [2. Добавление точек на карту](#markers)
* [3. Берем координаты вулканов из базы](#coor)
* [4. Всплывающие окна](#popups)
* [5. HTML и Google ссылки для окон](#html)
* [6. Стилизация точек](#style)
* [7. Добавление нового слоя на карту](#newlayer)
* [8. lambda функция: зависимость цвета от населения](#colour)
* [9. Добавление панели переключения слоев](#panel)
* [Скриншоты](#screenshots)
___

Приложение на языке _Python_.
Использованные библиотеки: _Pandas_, _Folium_.

В приложении 3 слоя: 1-й слой - _basemap_ - очертания стран и континентов, 2-й слой - _polygon layer_ - население стран показано цветом, 3-й слой - _point layer_ с точками на карте, указывающими местоположение вулканов. Слои можно включать или выключать при помощи панели. 

Всплывающие окна с названиями вулканов, высотой и кликабельными ссылками в _Google_.

Вулканы отмечены разными цветами (зеленый, желтый и красный) в зависимости от высоты.

## Создание приложения

<a id ="map"></a>

### 1. Создаем карту

* Импортируем _Folium_ и создаем объект _map_ класса _Map_ с тремя параметрами.

> map = folium.Map(location=[38.58, -99.09], zoom_start=5, tiles="Stamen Terrain")

** location (широта, долгота) - выбираем где будет открыта карта.

** zoom_start - выбираем стартовое приближение.

** tiles - внешний вид 1-го слоя.


* Сохраняем карту в формате _HTML_.

> _map.save("Map_html_popup_advanced.html")_

<a id ="markers"></a>

### 2. Добавляем точки на карту

* Создадем _FeatureGroup_.

> _fgv = folium.FeatureGroup(name="Volcanoes")_

** _FeatureGroup_ позволяет добавить множество features, таких как: _Marker_, _CircleMarker_, _GeoJson layer_, итд.

** _FeatureGroup_ это контейнер для объектов. Все объекты на карте будут помещены в _FeatureGroup_, так их проще показать или скрыть все разом.

* Метод _add_child_ для добавления объекта (точек) к существующему объекту (_map_).

* Используем объект _CircleMarker_ с аргументами (положение, цвет, прозрачность, и т.д.) библиотеки _Folium_ для создания точек на карте.

* _CircleMarker_ позволяет создавать всплывающие окна при нажатии на точку на карте.

* Далее

> _map.add_child(fgv)_

<a id ="coor"></a>

### 3. Добавляем координаты всех вулканов из _Volcanoes.txt_ на карту в виде точек

* Импортируем библиотеку _Pandas_.

> _data = pandas.read_csv("Volcanoes.txt")_

* Используем цикл _for_ для перебора структуры данных (_dataframe_).

* Для этого создаем 2 списка: _lat & lon_ (работа со списком проще и быстрее, чем с _dataframe series_).

```
data = pandas.read_csv("Volcanoes.txt")
lat = list(data["LAT"])
lon = list(data["LON"])
```
* Проходим циклом _for_, каждый раз получая широту и долготу для каждого из вулканов.

> _for lt, ln in zip(lat, lon):_

** Функция _zip_ используется для итерации 2 списков за раз.

<a id ="popups"></a>

### 4. Создаем динамические всплывающие окна (_popup windows_)

* Создаем 2 дополнительных списка _elev & name_ - высота и названия вулканов.

* Перебираем все 4 списка.

> _for lt, ln, el, name in zip(lat, lon, elev, name):_


<a id ="html"></a>

### 5. Применяем HTML для всплывающих окон, а так же Google ссылки

* Разместим ссылки во всплывающем окне. Код ниже создает всплывающие окна с названием вулкана в качестве _Google_ ссылки.

```
html = """
Volcano name:<br>
<a href="https://www.google.com/search?q=%%22%s%%22" target="_blank">%s</a><br>
Height: %s m
"""
```

** Символы _%s_ являются заполнителями, в которые будут вставлены переменные класса _str._

** В примере ниже, _str._ переменные _name_ и _el_ вставляются в переменную _html_:
> _iframe = folium.IFrame(html=html % (name, name, el)_


* Так выглядит теперь цикл _for_:

```
for lt, ln, el, name in zip(lat, lon, elev, name):
    iframe = folium.IFrame(html=html % (name, name, el), width=200, height=100)
    fg.add_child(folium.Marker(location=[lt, ln], popup=folium.Popup(iframe), icon = folium.Icon(color = "green")))
```

<a id ="style"></a>

### 6. Стилизуем маркеры по цвету в зависимости от высоты вулкана

* Так как в библиотеке _Folium_ отсутствует функционал для создания динамических маркеров, мы используем встроенный функционал _Pyhton_.

* Создадим функцию которая будет возвращать разные цвета в зависимости от высоты.

```
def color_producer(elevation):
    if elevation < 1000:
        return 'green'
    elif 1000 <= elevation < 3000:
        return 'orange'
    else:
        return 'red'
```

* Теперь добавим функцию _color_producer_ в цикл _for_.

```
for lt, ln, el in zip(lat, lon, elev):
    iframe = folium.IFrame(html=html % str(el), width=200, height=100)
    fg.add_child(folium.Marker(location=[lt, ln], popup=folium.Popup(iframe), icon = folium.Icon(color=color_producer(el))))
```

* Изменим внешиний вид точек, показывающих вулканы на карте. Используем _dir(folium)_ для поиска метода создания круглых маркеров.
После того как нужный метод найден, воспользуемся функцией _help_ для поиска аргументов, 
которые передадим методу для стилизации маркеров.
```
fgv.add_child(folium.CircleMarker(location=[lt, ln], radius=6, popup=folium.Popup(iframe), fill_color=color_producer(el), color='grey', fill_opacity=0.7))
```

<a id ="newlayer"></a>

### 7. Добавляем новый слой на карту

* В данный момент у нас 2 слоя: базовый слой и слой с маркерами. Добавим слой, показвающий количество населения в разных странах.

* Создадим новую _FeatureGroup_.

> _fgp = folium.FeatureGroup(name="Population")_  и добавим к ней _child_  __fgp.add_child__

* Добавим _polygon layer_ с помощью метода _GeoJson_: __folium.GeoJson__ 

* Данные для _GeoJson_ возьмем из файла _world.json_

** Не открывайте _world.json_ через _Atom_, если не уверены, что в мощности вашего компьютера. Используйте редактор "полегче", напр.: _Notepad_.

* Задаём атрибут ___data___ для метода ___GeoJson___, равный:

```
fgp.add_child(folium.GeoJson(data=open('world.json', 'r', encoding='utf-8-sig').read()))
```
** Метод ___open___ нужен для создания файлового объекта _Python_, далее укажем путь к файлу _'world.json'_.

** Метод ___read()___ в конце нужен так как новые версии _Folium_ принимают строку, вместо файла, в качестве вводных данных.

<a id ="colour"></a>

### 8. Меняем цвета для каждой из стран в зависимости от населения

* В файле _world.json_ есть информация о населении каждой из стран, которая хранится в атрибуте _"POP2005"_.

* Добавим новый аргумент _style_function_ в _GeoJson_:
```
style_function=lambda x: {'fillColor':'green' if x['properties']['POP2005'] < 10000000
else 'orange' if 10000000 <= x['properties']['POP2005'] < 20000000 else 'red'}))
```
** ___x['properties']['POP2005']___  В данном примере, 'properties' это ключ, 'POP2005' его значение в словаре в файле _world.json_, x означает 'features' в том же файле.

<a id ="panel"></a>

### 9. Добавляем панель переключения слоев

* Используем класс _LayerControl_ модуля _folium_.

> _map.add_child(folium.LayerControl())_

[Вверх](#anchor)
___

<a id ="screenshots"></a>

## Скриншоты
#### Точки красного, зеленого и желтого цветов обозначают действующие вулканы по высоте.
<img width="1440" alt="Map1" src="https://user-images.githubusercontent.com/97599612/168251281-f72afa90-fbf2-4e55-a8e1-132dba23f218.png">

#### Включены оба слоя. Население стран обозначено цветом.
<img width="1438" alt="Map2" src="https://user-images.githubusercontent.com/97599612/168251294-a22b74fb-faba-4feb-adb9-9a35ab9e4c24.png">

#### Пример всплывающего окна с названием вулкана, высотой и кликабельной ссылкой в Google.
<img width="1438" alt="Map3" src="https://user-images.githubusercontent.com/97599612/168251301-b5944c36-4cc2-4c03-8fd9-55686d8c0cb8.png">

#### Код программы (см. файл map2.py).
<img width="1172" alt="Code1" src="https://user-images.githubusercontent.com/97599612/168251259-aff72f8e-3029-4e2a-9f98-82419772ce2e.png">


[Вверх](#anchor)