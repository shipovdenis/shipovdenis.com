---
layout: post
title:  "Классы данных в Python"
keywords: "классы данных, data class, python"
description: "Основы классов данных с применением декоратора @dataclass, определение значений по умолчанию, указание типов данных, расширение с помощью методов, вопросы наследования, неизменяемости и оптимизации. Существующие альтернативы классам данных."
permalink: /blog/python-dataclass
---

Одной из интересных особенностей, появившихся в __Python__ с версии __3.7__, является __класс данных__ (__data class__). Это класс, как правило, просто представляющий некоторую структуру данных, но в действительности никакие функциональные ограничения не накладываются. Он создается с помощью нового декоратора __@dataclass__

```python
from dataclasses import dataclass

@dataclass
class DataClassCard:
    rank: str
    suit: str
```

> Все примеры в этой статье будут корректно работать только в __Python 3.7__ и выше.

__Классы данных__ уже обладают реализованной базовой функциональностью, например, вы можете создавать экземпляры, печатать, сравнивать их между собой, не прилагая для этого никаких усилий:

```python
>>> queen_of_hearts = DataClassCard('Q', 'Hearts')
>>> queen_of_hearts.rank
'Q'
>>> queen_of_hearts
DataClassCard(rank='Q', suit='Hearts')
>>> queen_of_hearts == DataClassCard('Q', 'Hearts')
True
```

Сравним с обычными классами. Минимальная реализация выглядит следующим образом:

```python
class RegularCard
    def __init__(self, rank, suit):
        self.rank = rank
        self.suit = suit
```

В таком простом примере разница в количестве написанного кода незначительная, однако сразу видны потенциальные проблемы: переменные __rank__ и __suit__ повторяются по три раза только для инициализации экземпляров объекта. Более того, если попытаться использовать этот класс, то представление объекта не особо информативно и почему-то один экземпляр __queen_of_hearts__ не равен другому:

```python
>>> queen_of_hearts = RegularCard('Q', 'Hearts')
>>> queen_of_hearts.rank
'Q'
>>> queen_of_hearts
<__main__.RegularCard object at 0x7fb6eee35d30>
>>> queen_of_hearts == RegularCard('Q', 'Hearts')
False
```

Понятно, что декоратор __@dataclass__ выполняет определенные действия "за кулисами".
__Класс данных__ уже реализует метод __\_\_repr\_\_()__ для обеспечения хорошего строкового представления и метод __\_\_eq\_\_()__, который выполняет базовые сравнения объектов. Чтобы класс __RegularCard__ обладал схожей функциональностью, нужно добавить это методы явно:

```python
class RegularCard
    def __init__(self, rank, suit):
        self.rank = rank
        self.suit = suit

    def __repr__(self):
        return (f'{self.__class__.__name__}'
                f'(rank={self.rank!r}, suit={self.suit!r})')

    def __eq__(self, other):
        if other.__class__ is not self.__class__:
            return NotImplemented
        return (self.rank, self.suit) == (other.rank, other.suit)
```

Эта статья показывает какие преимущества дают __классы данных__, помимо описанных выше. Мы рассмотрим:

* Как устанавливать значения по умолчанию для полей объекта?
* Как упорядочивать множество объектов?
* Как представлять неизменяемые данные?
* Как работает наследование в __классах данных__?

Скоро мы погрузимся в эти особенности новой функциональности __Python__. Однако, вам может показаться, что вы уже видели что-то подобное ранее.

# Существующие альтернативы классам данных

Для простых структур данных вы, вероятно, уже использовали кортежи или словари:

```python
>>> queen_of_hearts_tuple = ('Q', 'Hearts')
>>> queen_of_hearts_dict = {'rank': 'Q', 'suit': 'Hearts'}
```

Это работает, но накладывает большую ответственность на программиста:

* Нужно помнить, структуру какой сущности представляют __queen_of_hearts_tuple__ и __queen_of_hearts_dict__.

* Для __queen_of_hearts_tuple__ нужно помнить порядок следования атрибутов. 
Написав __queen_of_hearts_tuple = ('Hearts', 'Q')__ мы явно получим не то что хотели, однако сообщение об ошибке не возникнет.

* При использовании словарей нужно следить за согласованностью имен атрибутов. Например, __queen_of_hearts_dict = {'value': 'A', 'suit': 'Spades'}__ не будет работать должным образом, так как впоследствии мы ожидаем поле __rank__, а не __value__.

Более того, использование таких структур далеко от идеала с точки зрения чистоты и информативности кода:

```python
>>> queen_of_hearts_tuple[0]  # Доступ по индексу, а не имени
'Q'
>>> queen_of_hearts_dict['suit']  # Хотелось бы .suit
'Hearts'
```

Лучшей альтернативой является использование __именованных кортежей__ (__namedtuple__). Этот подход широко используется для создания небольших читаемых структур данных. Класс выше с использованием __namedtuple__ создается следующим образом:

```python
from collections import namedtuple

NamedTupleCard = namedtuple('NamedTupleCard', ['rank', 'suit'])
```

Такое определение __NamedTupleCard__ дает такой же результат как и класс __DataClassCard__:

```python
>>> queen_of_hearts = NamedTupleCard('Q', 'Hearts')
>>> queen_of_hearts.rank
'Q'
>>> queen_of_hearts
NamedTupleCard(rank='Q', suit='Hearts')
>>> queen_of_hearts == NamedTupleCard('Q', 'Hearts')
True
```

Так зачем тогда вообще возиться с классами данных? Во первых, классы данных имеют более широкие возможности, чем были продемонстрированы на данный момент. В то же время __namedtuple__ обладает и другими особенностями, которые не всегда желательны. По своей структуре __namedtuple__ является обычным кортежем. Это можно увидеть в сравнении:

```python
>>> queen_of_hearts == ('Q', 'Hearts')
True
```

На первый взгляд выглядит хорошо, однако недостаточная осведомленность о собственном типе может привести с неприятным ошибкам, которые нелегко отследить. Более того, будут легко проведено сравнение между разными именованными кортежами:

```python
>>> Person = namedtuple('Person', ['first_initial', 'last_name']
>>> ace_of_spades = NamedTupleCard('A', 'Spades')
>>> ace_of_spades == Person('A', 'Spades')
True
```

В именованных кортежах сложно определять значения по умолчанию, и по своей природе они являются неизменяемыми структурами данных, что в одних случаях просто превосходно, но в других сказывается недостаток гибкости:

```python
>>> card = NamedTupleCard('7', 'Diamonds')
>>> card.rank = '9'
AttributeError: can't set attribute
```

Классы данных полностью не заменяют все способы применения именованных кортежей. Если нужна структура данных, которая должна вести себя как кортеж, то нет никаких причин не использовать __namedtuple__.

Другой альтернативой и одним из вдохновителей для классов данных является библиотека __attrs__. Ее можно установить с помощью __pip__

```bash
pip install attrs
```

и реализовать наш класс следующим образом:

```python
import attr

@attr.s
class AttrsCard:
    rank = attr.ib()
    suit = attr.ib()
```

Библиотека __attrs__ содержит функциональность, которой не обладают классы данных, например, валидаторы и конвертеры. Более того, __attrs__ поддерживает __Python 2.7__ и __Python 3.4__ и выше. Но все-таки это не часть стандартной библиотеки и придется добавлять лишнюю зависимость в проект.

Помимо обычных и именованных кортежей, словарей и __attrs__ существует множество других средств, предоставляющих схожую функциональность, например, __typing.NamedTuple__, __namedlist__, __attrdict__, __plumber__, __fields__. И нужно смотреть в каждом конкретном случае, какой из них подойдет лучше всего для стоящей задачи.

# Основы классов данных

Давайте теперь вернемся к новым классам данных. Для примера мы создадим класс __Position__, который будет представлять географические координаты с именем, широтой и долготой:

```python
from dataclasses import dataclass

@dataclass
class Position:
    name: str
    lon: float
    lat: float
```

Декоратор __@dataclass__, размещенный выше определения, делает этот объект классом данных. Под строкой __class Position:__ мы размещаем нужные поля класса. Конструкция, состоящая из двоеточия, которое разделяет поле класса и указание его типа, является новой особенностью __Python 3.6__ и называется __аннотацией переменной__ (__variable annotation__). Мы скоро поговорим об этой конструкции и о том, почему определили типы данные как __str__ и __float__.

Этих нескольких строк достаточно, наш класс данных готов:

```python
>>> pos = Position('Oslo', 10.8, 59.9)
>>> print(pos)
Position(name='Oslo', lon=10.8, lat=59.9)
>>> pos.lat
59.9
>>> print(f'{pos.name} is at {pos.lat}°N, {pos.lon}°E')
Oslo is at 59.9°N, 10.8°E
```

Также можно создать класс данных подобно тому, как создаются именованные кортежи. Следующий код равносилен определению класса __Position__ выше:

```python
from dataclasses import make_dataclass

Position = make_dataclass('Position', ['name', 'lat', 'lon'])
```

Класс данных — это обычный класс __Python__. Единственным отличием является набор уже реализованных методов, таких как __\_\_init\_\_()__, __\_\_repr\_\_()__ и __\_\_eq\_\_()__.

## Значения по умолчанию

Очень просто добавить значения по умолчанию для полей класса данных:

```python
from dataclasses import dataclass

@dataclass
class Position:
    name: str
    lon: float = 0.0
    lat: float = 0.0
```

Это работает в точности так же, как определение значений в методе __\_\_init\_\_()__ обычных классов:

```python
>>> Position('Null Island')
Position(name='Null Island', lon=0.0, lat=0.0)
>>> Position('Greenwich', lat=51.8)
Position(name='Greenwich', lon=0.0, lat=51.8)
>>> Position('Vancouver', -123.1, 49.3)
Position(name='Vancouver', lon=-123.1, lat=49.3)
```

Немного позднее мы узнаем о __default_factory__, что позволит определять более сложные значения по умолчанию.

## Определение типа (Type Hints)

Классы данных поддерживают "из коробки" определения типов. С помощью конструкции __name: str__ мы говорим, что поле __name__ является строкой.

Добавление определений типа к полям классов данных является обязательным условием. Если вы не хотите в точности указывать конкретный тип, то нужно использовать __typing.Any__:

```python
from dataclasses import dataclass
from typing import Any

@dataclass
class WithoutExplicitTypes:
    name: Any
    value: Any = 42
```

Несмотря на обязательность указания типов данных, во время выполнения они не проверяются, и следующий код будет работать без проблем:

```python
>>> Position(3.14, 'pi day', 2018)
Position(name=3.14, lon='pi day', lat=2018)
```

__Python__ является языком с динамической типизацией, и для проверки согласованности типов нужно использовать библиотеки наподобие __Mypy__, которые будут постоянно отслеживать исходный код на корректность во время его написания.

## Добавление методов

Методы в классы данных добавляются так же, как и в обычные классы. Для примера вычислим расстояние между двумя точками на планете с помощью формулы гаверсинусов. Добавим метод __distance_to()__ в класс:

```python
from dataclasses import dataclass
from math import asin, cos, radians, sin, sqrt

@dataclass
class Position:
    name: str
    lon: float = 0.0
    lat: float = 0.0

    def distance_to(self, other):
        r = 6371  # Earth radius in kilometers
        lam_1, lam_2 = radians(self.lon), radians(other.lon)
        phi_1, phi_2 = radians(self.lat), radians(other.lat)
        h = (sin((phi_2 - phi_1) / 2)**2
             + cos(phi_1) * cos(phi_2) * sin((lam_2 - lam_1) / 2)**2)
        return 2 * r * asin(sqrt(h))
```

Это работает именно так, как вы и ожидаете:

```python
>>> oslo = Position('Oslo', 10.8, 59.9)
>>> vancouver = Position('Vancouver', -123.1, 49.3)
>>> oslo.distance_to(vancouver)
7181.7841229421165
```

# Добавляем гибкости в классы данных

На данный момент мы рассмотрели базовые возможности классов данных. Теперь посмотрим на более продвинутые возможности, такие как параметры декоратора __@dataclass__ и функцию __field()__. Вместе они предоставляют больше контроля при создании классов данных.

Вернемся к примеру с игральной картой из начала статьи и добавим класс колоды карт:

```python
from dataclasses import dataclass
from typing import List

@dataclass
class PlayingCard:
    rank: str
    suit: str

@dataclass
class Deck:
    cards: List[PlayingCard]
```

Простая колода из двух карт может быть создана при помощи следующего кода:

```python
>>> queen_of_hearts = PlayingCard('Q', 'Hearts')
>>> ace_of_spades = PlayingCard('A', 'Spades')
>>> two_cards = Deck([queen_of_hearts, ace_of_spades])
Deck(cards=[PlayingCard(rank='Q', suit='Hearts'),
            PlayingCard(rank='A', suit='Spades')])
```

## Расширенные значения по умолчанию

Теперь нам нужно определить значения по умолчанию для класса __Deck__. Было бы удобно, если бы вызов __Deck()__ создавал обычную колоду из 52 игральных карт. Для начала определим масти и ранги карт. Затем добавим функцию __make_french_deck()__, которая создаст список экземпляров класса __PlayingCard__:

```python
RANKS = '2 3 4 5 6 7 8 9 10 J Q K A'.split()
SUITS = '♣ ♢ ♡ ♠'.split()

def make_french_deck():
    return [PlayingCard(r, s) for s in SUITS for r in RANKS]
```

Для наглядности четыре масти мы определили, используя символы __Юникода__. Для упрощения сравнений карт в дальнейшем, ранги и масти карт перечислены в возрастающем порядке.

```python
>>> make_french_deck()
[PlayingCard(rank='2', suit='♣'), PlayingCard(rank='3', suit='♣'), ...
 PlayingCard(rank='K', suit='♠'), PlayingCard(rank='A', suit='♠')]
```

Теоретически теперь можно использовать эту функцию, чтобы определить значение по умолчанию для __Deck.cards__:

```python
from dataclasses import dataclass
from typing import List

@dataclass
class Deck: # Не будет работать
    cards: List[PlayingCard] = make_french_deck()
```

__Не делайте так!__ Это один из наиболее распространенных анти-паттернов в __Python__ — использование изменяемых значений по-умолчанию. Проблема в том, что все экземпляры класса __Deck__ будут использовать один и тот же список значений как значение для поля __cards__. Это значит, что если карта была удалена из одной колоды, то она также будет удалена из всех существующих колод. В действительности, классы данных ограждают от этого, и код выше вызовет исключение __ValueError__.

Вместо этого классы данных используют __default_factory__ для обработки изменяемых значений по умолчанию. Для использования __default_factory__ нужно прибегнуть к спецификатору __field()__:

```python
from dataclasses import dataclass, field
from typing import List

@dataclass
class Deck:
    cards: List[PlayingCard] = field(default_factory=make_french_deck)
```

Теперь можно создать полную колоду карт:

```python
>>> Deck()
Deck(cards=[PlayingCard(rank='2', suit='♣'), PlayingCard(rank='3', suit='♣'), ...
            PlayingCard(rank='K', suit='♠'), PlayingCard(rank='A', suit='♠')])
```

Спецификатор __field()__ используется для настройки каждого поля класса данных в отдельности (мы увидим примеры этого ниже) и поддерживает:

* __default__: Значение по умолчанию для поля
* __default_factory__: Функция, возвращающая начальное значение для поля
* __init__: Указание использовать поле в методе __\_\_init\_\_()__ (по умолчанию, __True__)
* __repr__: Указание использовать поле в функции __repr__ (по умолчанию, __True__)
* __compare__: Указание включать поле в сравнения (по умолчанию, __True__)
* __hash__: Указание включать поле, когда вычисляется __hash()__ (по умолчанию, совпадает с __compare__)
* __metadata__: Отображение информации о поле

В примере класса __Position__ мы увидели, как добавлять простые значения по умолчанию с помощью конструкции:

```python
__lat: float = 0.0
```

Однако, если нужно кастомизировать поле, например, чтобы скрыть его из вывода функции __repr__, следует использовать параметр __default__: 

```python
__lat: float = field(default=0.0, repr=False)__.
```

Можно не определять одновременно __default__ и __default_factory__.

Параметр __metadata__ не используется непосредственно классами данных, но он полезен для прикрепления информации о поле для себя и для других разработчиков. В классе __Position__ можно было бы указать, что широта и долгота должны быть указаны в градусах:

```python
from dataclasses import dataclass, field

@dataclass
class Position:
    name: str
    lon: float = field(default=0.0, metadata={'unit': 'degrees'})
    lat: float = field(default=0.0, metadata={'unit': 'degrees'})
```

Метаданные и другую информация о поле можно получить с помощью функции __fields()__:

```python
>>> from dataclasses import fields
>>> fields(Position)
(Field(name='name',type=<class 'str'>,...,metadata={}),
 Field(name='lon',type=<class 'float'>,...,metadata={'unit': 'degrees'}),
 Field(name='lat',type=<class 'float'>,...,metadata={'unit': 'degrees'}))
>>> lat_unit = fields(Position)[2].metadata['unit']
>>> lat_unit
'degrees'
```

## Представление

Вспомним, как мы создавали колоду карт:

```python
>>> Deck()
Deck(cards=[PlayingCard(rank='2', suit='♣'), PlayingCard(rank='3', suit='♣'), ...
            PlayingCard(rank='K', suit='♠'), PlayingCard(rank='A', suit='♠')])
```

Такое представление вполне читаемо, но уж слишком многословно. В примере выше удалены 48 из 52 карт, которые в реальности будут выведены на экран. На 80-колоночном дисплее просто печать одной полной колоды займет 22 строки! Давайте теперь добавим более лаконичное представление. В целом, объект в __Python__ имеет два строковых представления:

* __repr(obj)__ определяется с помощью __obj.\_\_repr\_\_()__ и возвращает представление __obj__ для разработчика. Классы данных уже реализуют это.

* __str(obj)__ определяется с помощью __obj.\_\_str\_\_()__ и возвращает представление __obj__ для пользователя. Классы данных не реализуют метод __\_\_str\_\_()__, поэтому будет вызван метод __\_\_repr\_\_()__.

Реализуем удобное для пользователя представление класса __PlayingCard__:

```python
from dataclasses import dataclass

@dataclass
class PlayingCard:
    rank: str
    suit: str

    def __str__(self):
        return f'{self.suit}{self.rank}'
```

Карты теперь выглядят намного приятнее, но представление колоды по прежнему многословно:

```python
>>> ace_of_spades = PlayingCard('A', '♠')
>>> ace_of_spades
PlayingCard(rank='A', suit='♠')
>>> print(ace_of_spades)
♠A
>>> print(Deck())
Deck(cards=[PlayingCard(rank='2', suit='♣'), PlayingCard(rank='3', suit='♣'), ...
            PlayingCard(rank='K', suit='♠'), PlayingCard(rank='A', suit='♠')])
```

Для демонстрации добавления собственного метода __\_\_repr\_\_()__ отойдем от принципа, что этот метод должен возвращать код для воссоздания объекта. Практичность важнее. Добавим более лаконичное представление класса __Deck__:

```python
from dataclasses import dataclass, field
from typing import List

@dataclass
class Deck:
    cards: List[PlayingCard] = field(default_factory=make_french_deck)

    def __repr__(self):
        cards = ', '.join(f'{c!s}' for c in self.cards)
        return f'{self.__class__.__name__}({cards})'
```

Обратите внимание на спецификатор __!s__. С помощью него мы указываем, что хотим использовать __str()__ для представления __PlayingCard__. Теперь отображение на экране класса __Deck__ приятно для глаз:

```python
>>> Deck()
Deck(♣2, ♣3, ♣4, ♣5, ♣6, ♣7, ♣8, ♣9, ♣10, ♣J, ♣Q, ♣K, ♣A,
     ♢2, ♢3, ♢4, ♢5, ♢6, ♢7, ♢8, ♢9, ♢10, ♢J, ♢Q, ♢K, ♢A,
     ♡2, ♡3, ♡4, ♡5, ♡6, ♡7, ♡8, ♡9, ♡10, ♡J, ♡Q, ♡K, ♡A,
     ♠2, ♠3, ♠4, ♠5, ♠6, ♠7, ♠8, ♠9, ♠10, ♠J, ♠Q, ♠K, ♠A)
```

## Сравнения

Во многих карточных играх карты сравниваются друг с другом для определения старшинства. Но в текущей реализации класса __PlayingCard__ нет поддержки таких типов операций:

```python
>>> queen_of_hearts = PlayingCard('Q', '♡')
>>> ace_of_spades = PlayingCard('A', '♠')
>>> ace_of_spades > queen_of_hearts
TypeError: '>' not supported between instances of 'Card' and 'Card'
```

Однако, это легко исправить:

```python
from dataclasses import dataclass

@dataclass(order=True)
class PlayingCard:
    rank: str
    suit: str

    def __str__(self):
        return f'{self.suit}{self.rank}'
```

Декоратор __@dataclass__ имеет две формы. До этого момента мы видели его простую форму, где не было никаких скобок и параметров. Однако в декоратор можно передать следующие параметры:

* __init__: Указание добавлять метод __\_\_init\_\_()__ (по умолчанию, __True__)
* __repr__: Указание добавлять метод __\_\_repr\_\_()__ (по умолчанию, __True__)
* __eq__: Указание добавлять метод __\_\_eq\_\_()__ (по умолчанию, __True__)
* __order__: Указание добавлять методы упорядочивания (по умолчанию, __False__)
* __unsafe_hash__: Указание принудительно добавлять метод __hash__() (по умолчанию, __False__)
* __frozen__: Если __True__, то попытка присвоения значений полям класса вызовет исключение. (по умолчанию, __False__)

После передачи параметра __order=True__, экземпляры класса __PlayingCard__ можно сравнить:

```python
>>> queen_of_hearts = PlayingCard('Q', '♡')
>>> ace_of_spades = PlayingCard('A', '♠')
>>> ace_of_spades > queen_of_hearts
False
```

Как две карты сравниваются между собой? Почему __Python__ считает, что королева главнее туза?

Оказывается, классы данных сравниваются как кортежи, элементами которых являются поля класса. Королева оказывается выше туза, так как 'Q' идет после 'A' в алфавите.

```python
>>> ('A', '♠') > ('Q', '♡')
False
```

Для нашего случая это не подходит. Нужно определить индекс сортировки, который использует порядок __RANKS__ и __SUITS__:

```python
>>> RANKS = '2 3 4 5 6 7 8 9 10 J Q K A'.split()
>>> SUITS = '♣ ♢ ♡ ♠'.split()
>>> card = PlayingCard('Q', '♡')
>>> RANKS.index(card.rank) * len(SUITS) + SUITS.index(card.suit)
42
```

Для того, чтобы класс __PlayingCard__ использовал этот индекс в сравнениях, нужно добавить в него поле __sort_index__. Значение этого поля должно автоматически вычисляться из значений полей __rank__ и __suit__. Для этого прекрасно подходит метод __\_\_post_init\_\_()__, который позволяет добавить дополнительную обработку после вызова метода __\_\_init\_\_()__:

```python
from dataclasses import dataclass, field

RANKS = '2 3 4 5 6 7 8 9 10 J Q K A'.split()
SUITS = '♣ ♢ ♡ ♠'.split()

@dataclass(order=True)
class PlayingCard:
    sort_index: int = field(init=False, repr=False)
    rank: str
    suit: str

    def __post_init__(self):
        self.sort_index = (RANKS.index(self.rank) * len(SUITS)
                           + SUITS.index(self.suit))

    def __str__(self):
        return f'{self.suit}{self.rank}'
```

Обратите внимание на то, что __sort_index__ добавляется как первое поле класса. Таким образом, сначала выполняется сравнение значений полей __sort_index__ у объектов, и только при их равенстве будут использованы другие поля. С помощью __field()__ необходимо исключить __sort_index__ из параметров метода __\_\_init\_\_()__. 

Наконец-то тузы победили:

```python
>>> queen_of_hearts = PlayingCard('Q', '♡')
>>> ace_of_spades = PlayingCard('A', '♠')
>>> ace_of_spades > queen_of_hearts
True
```

Теперь легко создать отсортированную колоду:

```python
>>> Deck(sorted(make_french_deck()))
Deck(♣2, ♢2, ♡2, ♠2, ♣3, ♢3, ♡3, ♠3, ♣4, ♢4, ♡4, ♠4, ♣5,
     ♢5, ♡5, ♠5, ♣6, ♢6, ♡6, ♠6, ♣7, ♢7, ♡7, ♠7, ♣8, ♢8,
     ♡8, ♠8, ♣9, ♢9, ♡9, ♠9, ♣10, ♢10, ♡10, ♠10, ♣J, ♢J, ♡J,
     ♠J, ♣Q, ♢Q, ♡Q, ♠Q, ♣K, ♢K, ♡K, ♠K, ♣A, ♢A, ♡A, ♠A)
```

Или раздать игрокам 10 случайных карт:

```python
>>> from random import sample
>>> Deck(sample(make_french_deck(), k=10))
Deck(♢2, ♡A, ♢10, ♣2, ♢3, ♠3, ♢A, ♠8, ♠9, ♠2)
```

# Неизменяемые классы данных

Одной из основных особенностей именованных кортежей, которые были показаны ранее, является их неизменяемость. То есть, нельзя присвоить значения полям объекта повторно. Зачастую это является крайне полезным свойством. Чтобы наделить класс данных свойством неизменяемости, нужно при создании установить параметр __frozen=True__:

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Position:
    name: str
    lon: float = 0.0
    lat: float = 0.0
```

Теперь после создания элемента класса нельзя изменить значения его полей:

```python
>>> pos = Position('Oslo', 10.8, 59.9)
>>> pos.name
'Oslo'
>>> pos.name = 'Stockholm'
dataclasses.FrozenInstanceError: cannot assign to field 'name'
```

Однако, следует помнить, что если класс данных содержит в качестве значения поля изменяемую структуру, то она в любом случае может изменяться. Это справедливо для всех вложенных структур данных в __Python__:

```python
from dataclasses import dataclass
from typing import List

@dataclass(frozen=True)
class ImmutableCard:
    rank: str
    suit: str

@dataclass(frozen=True)
class ImmutableDeck:
    cards: List[PlayingCard]
```

Классы __ImmutableCard__ и __ImmutableDeck__ неизменяемые, а поле __cards__ является списком, изменяемой структурой данных:

```python
>>> queen_of_hearts = ImmutableCard('Q', '♡')
>>> ace_of_spades = ImmutableCard('A', '♠')
>>> deck = ImmutableDeck([queen_of_hearts, ace_of_spades])
>>> deck
ImmutableDeck(cards=[ImmutableCard(rank='Q', suit='♡'), ImmutableCard(rank='A', suit='♠')])
>>> deck.cards[0] = ImmutableCard('7', '♢')
>>> deck
ImmutableDeck(cards=[ImmutableCard(rank='7', suit='♢'), ImmutableCard(rank='A', suit='♠')])
```

Чтобы избегать таких случаев, нужно обязательно использовать для полей класса, который предполагается неизменяемым, неизменяемые типы данных. Для класса __ImmutableDeck__ следовало бы использовать кортеж вместо списка.

# Наследование

Как и обычные классы, классы данных можно наследовать. Для примера, расширим класс __Position__ полем __country__ и будем использовать его для представления столиц стран:

```python
from dataclasses import dataclass

@dataclass
class Position:
    name: str
    lon: float
    lat: float

@dataclass
class Capital(Position):
    country: str
```

В этом простом примере все работает так, как и ожидается:

```python
>>> Capital('Oslo', 10.8, 59.9, 'Norway')
Capital(name='Oslo', lon=10.8, lat=59.9, country='Norway')
```

Поле __country__ класса __Capital__ добавилось после трех полей базового класса __Position__. Все становится несколько сложнее, если какие-либо поля класса-родителя содержат значения по умолчанию:

```python
from dataclasses import dataclass

@dataclass
class Position:
    name: str
    lon: float = 0.0
    lat: float = 0.0

@dataclass
class Capital(Position):
    country: str  # Не работает
```

Этот код сразу упадет с ошибкой __TypeError__ и сообщением

```python
"non-default argument 'country' follows default argument"
```

Проблема заключается в то, что поле __country__ не имеет значения по умолчанию, в то время как __lon__ и __lat__ их имеют. И класс данных __Capital__ пытается создать метод __\_\_init\_\_()__ с сигнатурой

```python
def __init__(name: str, lon: float = 0.0, lat: float = 0.0, country: str):
    ...
```

Это невалидный код __Python__. Если параметру задано значение по умолчанию, то все следующие за ним также должны иметь их. И это справедливо для полей классов данных.

Еще одна вещь, о которой стоит помнить — порядок полей в классе-наследнике. Он совпадает с последовательностью полей в классе-родителе. И даже если поле переопределено в подклассе, то порядок не меняется:

```python
from dataclasses import dataclass

@dataclass
class Position:
    name: str
    lon: float = 0.0
    lat: float = 0.0

@dataclass
class Capital(Position):
    country: str = 'Unknown'
    lat: float = 40.0
```

Порядок полей в классе __Capital__ будет __name__, __lon__, __lat__, __country__, однако значение по умолчанию для поля __lat__ будет равным 40.0:

```python
>>> Capital('Madrid', country='Spain')
Capital(name='Madrid', lon=0.0, lat=40.0, country='Spain')
```

# Оптимизация классов данных

Приближаясь к завершению, нужно сказать несколько слов о слотах (__slots__). Их можно использовать, чтобы сделать классы более быстрыми и потребляющими меньше памяти. Какого-то специального синтаксиса для классов данных по работе со слотами нет, но и обычный способ прекрасно работает, ведь классы данных — это обычные классы __Python__:

```python
from dataclasses import dataclass

@dataclass
class SimplePosition:
    name: str
    lon: float
    lat: float

@dataclass
class SlotPosition:
    __slots__ = ['name', 'lon', 'lat']
    name: str
    lon: float
    lat: float
```

По сути, в __\_\_slots\_\___ задается перечисление возможных полей или атрибутов класса. Они и только они могут быть в дальнейшем определены. Более того, если класс содержит слоты, то его поля не могут иметь значений по умолчанию.

Преимущества таких ограничений состоят в дальнейших оптимизационных улучшениях. Например, классы начинают потреблять меньше памяти, измерить это можно с помощью __pympler__:

```python
>>> from pympler import asizeof
>>> simple = SimplePosition('London', -0.1, 51.5)
>>> slot = SlotPosition('Madrid', -3.7, 40.4)
>>> asizeof.asizesof(simple, slot)
(440, 248)
```

Кроме того, работа с классами, содержащими слоты, как правило, проходит быстрее. В следующем примере демонстрируется скорость доступа к атрибутам при помощи библиотеки __timeit__:

```python
>>> from timeit import timeit
>>> timeit('slot.name', setup="from position import SlotPosition; slot=SlotPosition('Oslo', 10.8, 59.9)")
0.05882283499886398
>>> timeit('simple.name', setup="from position import SimplePosition; simple=SimplePosition('Oslo', 10.8, 59.9)")
0.09207444800267695
```

В этом примере разница по скорости составляет аж 35%.

# Заключение

Классы данных — одна из новых возможностей в __Python 3.7__. Они позволяют избавиться от написания шаблонного кода инициализации, представления и сравнения. В этой статье мы увидели, как определять свои классы данных, как задавать значения по умолчанию для полей, рассмотрели свойство неизменяемости и процедуру наследования. Если вы ходите более основательно погрузиться во все детали классов данных, то загляните в <a href="https://www.python.org/dev/peps/pep-0557/" rel="nofollow" target="_blank">PEP 557</a> и в обсуждения на <a href="https://github.com/ericvsmith/dataclasses/issues?utf8=✓&q=" rel="nofollow" target="_blank">GitHub</a>.
