Title: Pár slov o datovém typu Enum
Date: 2018-12-04 20:48:00
Modified: 2018-12-04 20:48:00
Author: Karolina Surma


# Třída Direction - příklad implementace
_(dřív zde na blogu pojmenovaná Rotation)_

Na dnešním srazu jsme řešily víc témat, z nichž jedno bylo použití třídy **Direction** datového typu Enum. Protože je to látka nová pro nás všechny na Roboprojektu, zde si podrobněji rozepíšeme, k čemu a jak třídu tohoto typu můžeme použít.

Doposud byly v projektu třídy tři: *Robot*, *Tile* a *State*. Ta poslední obsahuje jak informace o mapě, tak o robotech a jejích koordinátách. Všechny byly napsány podle pravidel nám známých ze [začátečnického kurzu PyLadies](https://naucse.python.cz/course/pyladies/beginners/class/), to znamená, že můžeme (a měly bychom) vytvořit nové objekty dané třídy tak, že ho inicializujeme, např.:

`my_robot = Robot(<direction>, <picture>, <coordinates>)`

Mezi `<>`  výše se nachází příklady argumentů, které musíme dát objektu třídy *Robot*, aby se korektně objevil na mapě (tzn. vykreslil správný obrázek na správném místě mapy se zobáčkem nasměrovaným na sever/ jih/ západ/ východ). 

Třída **Direction** je ale jiná. Zde víme předem, kolik objektů této třídy bude ve hře: 4. Víme, že samotné objekty se nebudou měnit, ale je možné změnit směr robota nebo políčka na nový. Nebudeme inicializovat nové objekty této třídy, ale uložíme si naše směry jako konstanty. Z objektu takové třídy by se měly dát vyčíst různé informace dle tabulky obsažené v [předchozím postu](https://roboprojekt.pyladies.cz/devaty-sraz-upravujeme-navrhujeme). 

O třídě, která se zdá být vhodná pro řešení tohoto zadání, jsme se již před nějakou dobou bavily. Jedná se o datový typ Enum, importovaný na začátku pythonního souboru:

```python
from enum import Enum 
        
    class Direction(Enum):
        N = 0
        E = 90
        S = 180
        W = 270
```
Pokud si tu třídu naimportujeme do konzole nebo vyzkoušíme .py soubor, můžeme k jejím objektům přistoupit velice jednoduše:

```python
print(Direction.N)
>>> 0
print(Direction(180))
>>> Direction.S
print(Direction.E + 90)
>>> Direction.S
```
K objektu takovéhle třídy se můžeme dostat jak skrz jeho _name_ (N, E, S, W), tak _value_ (0, 90, 180, 270). 
Díky tomu můžeme jednoduchým sčítáním otočit robota a aktualizovat jeho směr (tzn. přiřadit mu nový objekt této třídy).

K čemu je datový typ *Enum* dobrý? Jeho objekty jsou už uvnitř inicializované (nemusí se definovat metoda `__init__()`) a dostupné pro využití v dalším kódu. 
O směru ale, jak jsme si psaly v tabulce, můžeme uvažovat na spoustu způsobů: 
- světová strana
- stupně otočení
- kudy se posune náš objekt, pokud na něm uplatním metodu pohyb v Xové a Yové ose (delta koordinát) 
- výčet (rozsah 0-3)

Pro různé účely potřebujeme různé zápisy té stejné věci. Měly jsme si rozmyslet, které z výše uvedených informací informaci dáme třídě jako atribut, a které získáme metodou. Není jeden správný způsob, jak si tyto informace uložit. Mně se nelíbilo, že nové koordináty nebo výčet musím získat metodou, když hned při náhledu na objekt "vím", kam se nově posune. Udělala jsem tedy jinou fintu:

```python
from enum import Enum 

class Direction(Enum):
    N = 0, (0, +1), 0
    E = 90, (+1, 0), 1
    S = 180, (0, -1), 2
    W = 270, (-1, 0), 3
```

Stávající implementace všechny potřebné informace ukládá jako atributy, ke kterým můžou sahat další části programu:
- stupně potřebujeme pro vykreslení objektů (robotů a políček) v pygletu
- díky deltě koordinát můžeme vypočítat nové místo robota na mapě
- mapy s políčky, které se exportují z programů Tiled, mají přidané "custom properties" - námi nadefinované vlastnosti, které pracují s integery.

Tím pádem víme, co směr **je**, ale co má tedy **umět**? 
Cíl byl jasný: pokud se robot, který směruje na západ, otočí o 90 stupňů, bude směrovat na sever. Zdá se: nic jednoduššího, než napsat na to metodu, kopírující chování, které jsme si vyzkoušely před chvílí v konzoli! 
```python
    def get_new_direction(self, where_to):
        if where_to == "right":
            return Direction((self.value + 90) % 360)
        if ...
```
Můj robotí příklad přeložen do Pythonu by měl znít takto:

```python
>>> Direction.W.get_new_direction("right")
>>> Direction.N
```

Jenže kód výše nefunguje. *Value* totiž teď není jen `270`, ale celá hodnotá - jakási trojice: `270, (-1, 0), 3`.

V [dokumentaci Pythonu](https://docs.python.org/3/library/enum.html#planet) se můžeme dočíst, že lze dát objektu typu Enum víc atributů, ale aby nebyly zapsané jako trojice, musíme přepsat metodu `__init__()` a atributy v ní rozbalit. 

```python
    def __init__(self, degrees, coordinates_delta, tile_property):
        self.degrees = degrees
        self.coordinates_delta = coordinates_delta
        self.tile_property = tile_property
```

Díky tomu, když vypíšeme:

```python
print(Direction.E.degrees)
print(Direction.E.coordinates_delta)
print(Direction.E.tile_property)
``` 
Dostaneme:
```python
90
(+1, 0)
1
```

Ale co naše metoda? Žádný atribut *value* nemáme, tak se přepíše na `self.degrees`, a můžeme slavnostně spustit testovací otočení. V tu chvíli by to všechno mělo vybuchnout velkou chybou:

```
Traceback (most recent call last):
  File "numer.py", line 38, in <module>
    aa = Direction.W.get_new_coordinates("right")
  File "numer.py", line 22, in rotate
    return Direction((self.degrees + 270) % 360)
  File "/usr/lib/python3.6/enum.py", line 293, in __call__
    return cls.__new__(cls, value)
  File "/usr/lib/python3.6/enum.py", line 535, in __new__
    return cls._missing_(value)
  File "/usr/lib/python3.6/enum.py", line 548, in _missing_
    raise ValueError("%r is not a valid %s" % (value, cls.__name__))
ValueError: 0 is not a valid Direction 
```
Python totiž ví, že objekt třídy **Direction** nemůže mít jen jeden atribut - daly jsme mu až 3. Na druhou stranu, naši jednoduchou metodu na otočení nelze nyní napsat lépe, protože Python teď neví, jak zjistit, který objekt jeho třídy Direction má 0 stupňů, a následně mu přiřadit zbývající atributy tohoto konkrétního objektu. Uff.

Řešení přinesl internet, a konkrétně [tento post](http://xion.io/post/code/python-enums-are-ok.html), kde přistál nápad na přepsání metody `__new__()`, aby vynutila chování, které potřebujeme pro vyřešení problému.

```python
 def __new__(cls, degrees, coor_delta, tile_property):
        obj = object.__new__(cls)
        obj._value_ = degrees
        obj.coor_delta = coor_delta
        obj.map_property = tile_property
        return obj
```
        
Díky této metodě atribut *value* jsou nyní stupně. Python ví, že pokud najde objekt třídy Direction, který má *value* 90, pak tento objekt má zároveň *deltu* (+1, 0) a *property*: 1. A to je přesně ono!
Pokud nyní zavoláme naši metodu a otočíme objekt směřující na západ o 90 stupňů doprava, dostaneme objekt Direction.N, který bude mít správně přiřazené další atributy. 



# Metoda `__add__()`

V jednom z commitů se našla metoda `__add__()` (nedostala se do finální verze, ale vzbudila náš zájem, tak ji nám Petr vysvětlil na srazu).
Již známe speciální metodu `__init__()`, která umožňuje inicializovat nové objekty dané třídy. Podtržítka u `__add__()` naznačují, že se jedná o další speciální metodu.

Můžeme si představit třídu Direction, která označí čtyři směry:

```python
class Direction(Enum):
    FORWARD = 0
    RIGHT = 1
    BACK = 2
    LEFT = 3 
```

Už víme, jak se můžeme dostat k jednotlivým informacím obsaženým v objektech takové třídy.

```python
print(Direction.BACK)   		# Direction.BACK
print(Direction.BACK.value)     # 2
print(Direction(2))     		# Direction.BACK
```

Občas se nám může hodit i jiná operace, např. sčítání. Pokud máme objekt směřující dolu a posuneme ho doleva, chceme se dostat do směru vpravo. 
Můžeme se pokusit o standardní sčítání. Víme, že Python umí pracovat s čísly, pokud mu tak dáme jako argumenty integery, poradí si s nimi bez chyby:

```python
new_direction = (Direction.BACK.value + 3) % 4  # 1
print(Direction(new_direction)) 				# Direction.RIGHT
```

Ale jak Python ví, že má sečíst dvě hodnoty po obou stranách znaku plus? Všechno, co Python umí, v něm někdo naprogramoval. Základní operace tvoří balík standardních funkcí - to jsou ty, které se pak učíme na začátečnickém kurzu. Nemusíme Python znovu učit, jak má zacházet např. se základní matematikou, protože objekty třídy integer nebo float mají nadefinované metody, které to umí udělat. 
V lekci o [dědičnosti](https://naucse.python.cz/course/pyladies/beginners/inheritance/) jsme se učily, jak vytvářet podtřídy a přepisovat v nich metody nadtříd. Zde se jedná o stejný princip - pro naši třídu Direction můžeme pozměnit chování sčítání, aby si s ní dokázalo poradit.

```python 
def __add__(self, other):
    return Direction((self.value + other.value) % 4)
```

Říkáme touto metodou: 

> Pythone, pokud dostaneš dva objekty třídy Direction, vezmi atribut
> *value* obou těchto objektů, sečti je, a vrať objekt třídy Direction, jehož *value* je zbytkem dělení provedeného součtu čtyřmi. 

Proč zbytek z dělení? Máme čtyři validní směry a chceme, aby se součet "přesypal" přes nulu, místo aby se dostal do hodnot nad 4.

Když nyní zkusíme sečíst dva směry, dosáhneme stejného výsledku jako výše, při výrazné zvýšení čitelnosti zápisu. 

```python
print(Direction.BACK + Direction.LEFT)   # Direction.RIGHT
```
