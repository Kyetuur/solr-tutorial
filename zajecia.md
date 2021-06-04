# Przygotowanie srodowiska

## Instalacja Solr

Instalacja solra w kontenerze dockera:

(Działającego na porcie 8983 w trybie Cloud)

```
docker run -d --name solr_nosql -p 8983:8983 -t solr -cloud
```

Przechodzimy pod adres
```
http://localhost:8983/solr/#/
```
,aby sprawdzić czy serwer dobrze działa.




## Przygotowanie kolekcji
### Utworzenie kolekcji 'collection'

W panelu admina Solr klikamy **Collections** a następnie **Add Collection**.

Ustawiamy (Ważne jest aby nazwa była **collection**, - wywołania REST się do niej odwołują):
```
name: collection
config set: _default
numShards: 1
replicationFact: 1
```

i tworzymy nową kolekcję.

### Utworzenie schematu

Przygotujemy teraz schemat dla kolekcji **collection** z którego będziemy korzystać.
Schemat przedstawia pewien model książki.

Będą to odpowiednio pola:

```
author    : text_general
country   : text_general
imageLink : text_general
language  : text_general
link      : text_general
pages     : pint
title     : text_general
year      : pint
```

Pola można ręcznie wyklikać z poziomu panelu admina ( W dropdownie kolekcji wybieramy naszą kolekcje, następnie *Schema* oraz *Add Field*), można też wkleić w konsole poleceń następujące polecenia w celu zautomatyzowania klikania:

#### Dla UNIX'a
```

curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"author", "type":"text_general", "multiValued":false, "stored":true}}' http://localhost:8983/solr/collection/schema
curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"country", "type":"text_general", "multiValued":false, "stored":true}}' http://localhost:8983/solr/collection/schema
curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"imageLink", "type":"text_general", "multiValued":false, "stored":true}}' http://localhost:8983/solr/collection/schema
curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"language", "type":"text_general", "multiValued":false, "stored":true}}' http://localhost:8983/solr/collection/schema
curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"link", "type":"text_general", "multiValued":false, "stored":true}}' http://localhost:8983/solr/collection/schema
curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"pages", "type":"pint", "multiValued":false, "stored":true}}' http://localhost:8983/solr/collection/schema
curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"title", "type":"text_general", "multiValued":false, "stored":true}}' http://localhost:8983/solr/collection/schema
curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"year", "type":"pint", "multiValued":false, "stored":true}}' http://localhost:8983/solr/collection/schema

```

#### Dla Windowsa (Powershell)

```

Invoke-RestMethod http://localhost:8983/solr/collection/schema -Method POST -ContentType "application/json" -Body '{"add-field": {"name":"author", "type":"text_general", "multiValued":false, "stored":true}}'
Invoke-RestMethod http://localhost:8983/solr/collection/schema -Method POST -ContentType "application/json" -Body '{"add-field": {"name":"country", "type":"text_general", "multiValued":false, "stored":true}}'
Invoke-RestMethod http://localhost:8983/solr/collection/schema -Method POST -ContentType "application/json" -Body '{"add-field": {"name":"imageLink", "type":"text_general", "multiValued":false, "stored":true}}'
Invoke-RestMethod http://localhost:8983/solr/collection/schema -Method POST -ContentType "application/json" -Body '{"add-field": {"name":"language", "type":"text_general", "multiValued":false, "stored":true}}'
Invoke-RestMethod http://localhost:8983/solr/collection/schema -Method POST -ContentType "application/json" -Body '{"add-field": {"name":"link", "type":"text_general", "multiValued":false, "stored":true}}'
Invoke-RestMethod http://localhost:8983/solr/collection/schema -Method POST -ContentType "application/json" -Body '{"add-field": {"name":"pages", "type":"pint", "multiValued":false, "stored":true}}'
Invoke-RestMethod http://localhost:8983/solr/collection/schema -Method POST -ContentType "application/json" -Body '{"add-field": {"name":"title", "type":"text_general", "multiValued":false, "stored":true}}'
Invoke-RestMethod http://localhost:8983/solr/collection/schema -Method POST -ContentType "application/json" -Body '{"add-field": {"name":"year", "type":"pint", "multiValued":false, "stored":true}}'

```

Gdy posiadamy już wszystkie podstawowe pola, zajmijmy się dodaniem pola *Copy Field*  

W panelu edycji schematu dodajemy pole *author_str* typu *string*.  
Następnie klikamy *Add Copy Field* i ustawiamy:  
*author* jako **source** a *author_str* jako **destination**.


## Zaindeksowanie dokumentów

W tym momencie nakarmimy solr'a danymi, w panelu admina wybierzmy naszą kolekcje i przejdźmy do zakładki **Documents**.  

Przeklejmy w okienko *Documents(s)* zawartosc *data.txt* (Jest to plik zawierający dane na temat książek w formacie JSON oddzielone przecinkami).
>> https://github.com/benoitvallon/100-best-books/blob/master/books.json


Klikamy *Submit Document*  

Przechodzimy do *Query* i klikamy *Execute Query* , jeśli widzimy jakieś dokumenty znaczy że wszystko poszło dobrze.



# Przedstawienie funkcjonalnosci

## Szukanie
Przechodzimy do zakładki *Query*, można też użyć jakiegoś programu do zapytań REST.


### Wyszukanie po polu
przykładowo wyklikujemy:

---


q  
```
title:Fall
```
Takie zapytanie wyszuka nam te dokumenty, w których znajduje się słowo *Fall* w polu *title*  
Możemy użyć również wildcardów *?* dla pojedynczego znaku oraz *\**

---

### Wyszukiwanie AND

q
```
title:(Apart AND Things)
```
Ważne! AND musi być z wielkich liter.

---

### Wyszukiwanie Proximity

q
```
title: "Things Apart"~1
```

Takie wyszukiwanie znajduje dokumenty gdzie *Things* and *Apart* występują w podanej kolejności, a między nimi jest maksymalnie 1 słowo.  
W takim wyszukaniu ważne są cudzysłowy - przy tym również tracimy możliwość używania wildcard.

---

### Wyszukiwanie zakresowe

q
```
year: [1600 TO 1608}
```

Takie zapytanie zwróci książki napisane między 1600 a 1608 roku. (Nawias kwadratowy oznacza zawieranie wartości granicznej, klamrowy przeciwnie)

---

### Wyszukiwanie Fuzzy

q
```
title:Asarr~2
```

Wyszukiwanie pozwalające znaleźć słowa podobne (w sensie odległości Damerau-Levenshteina).

### Wyszukiwanie Negative

q
```
-title:apart
```

Wyszukiwanie pozwalające znaleźć dokumenty nienawierające jakiegoś słowa.

### Pozostałe pola

*fq* - filter query, zapytanie odpalane na już wyliczonych wynikach, może ograniczyć ilość odpowiedzi, ale nie wpływa na wartość score'a.  
*sort* - miejsce na pole według którego posortowane będą odpowiedzi.  
*fl* - field list, pozwala ograniczyć ilość pól zwracanych dokumentów. Pozwala także na wskazanie pseudopól w odpowiedzi (np. author_str).  
*df* - default search field, pozwala wskazać pole po jakim należy szukać wartości, jeśli nie zostało ono wpisane wprost w *q*.  
*Raw Query Parameter* - pozwala dopisać własne parametry, których nie ma w UI (np facet.range).  
*wt* - pozwala wskazać format zwracanych wartości.  
*indent off* - pozwala wylączyć wcięcia w zwracanym jsonie.  
*dismax* - włącza obsługe DixMax Query Parsera.  
*edismax* - włącza obsługe Extended DisMax Parsera.  
*hl* - highlighting, włącza podkręślanie wyszukany fraz.  

## Facet

Facet - kategoryzowanie wyników.
Tutaj przydatny jest zewnętrzny program do wywołań REST, np. Postman.  
UI Solr'a jest tutaj mocno ograniczone  

Dla ułatwienia prezentacji facet'ów, ustawmy parametr *rows* na 0.

### Facet po polu

Włączamy ustawienie *facet* a następnie ustawiamy  
facet.field
```
language
```

Zapytanie to wskaże nam ile książek zostało napisane w danych językach.  
Jeśli interesują nas tylko te języki, w których język zaczyna się na np. 's'  
możemy określić field.prefix.  

Jeśli interesują nas tylko języki w których zostało napisane minimalnie np. 10 książek,
możemy określić parametr field.mincount na odpowiednią wartość.  
Tej opcji nie ma w UI, jeśli chcemy ją wywołać musimy wpisać w   

Raw Query Parameters
```
facet.mincount=10
```

Warte zaznaczenia jest to że Solr domyślnie zwraca maksymalnie 100 różnych kategorii, jeśli chcemy więcej musimy określić facet.limit.  


Zauważmy że facet działa po *termach* konkretnych termach, dlatego zapytanie:  

facet.field
```
author
```

Zwróci facet'y rozbite na konkretne termy danego tekstu. (Pole author jest typu text)

Jeśli interesują nas facet'y po konkretnych autorach możemy użyć zdefiniowanego wcześniej pola author_str,  
którego wartość jest tożsama z wartością author, ale typ tego pola jest stringiem (który nie podlega tokenizacji)

facet.field
```
author_str
```

### Facet Zakresowy

Umożliwia kategoryzowanie w podanym zakresie, w celu jego użycia należy zdefiniować  następujące parametry

```
facet.range= pole po którym tworzymy kategorie

facet.range.start = dolny zakres kategoryzowanych danych
facet.range.end = górny zakres kategoryzowanych danych
facet.range.gap = zakres pojedynczej kategorii
```

na przykładzie:
Jeśli chcemy zkategoryzować książki po roku wydania,  
gdzie interesują nas książki stworzone między 0 a 2000 rokiem,  
oraz chcemy aby solr powiedział nam ile książek zostało stworzonych między rokiem 0 a 500, ile miedzy 500 a 1000 etc.  

Zapytanie jest następujące:  

```
facet.range=year

facet.range.start=0
facet.range.end=2000
facet.range.gap=500

W formie pojedynczego QueryStringa:

facet.range=year&facet.range.start=0&facet.range.end=2000&facet.range.gap=500
```


### Facet dzielony
Służy do tworzenia wielo-dzielonych kategorii,

przykładowo jeśli chcemy się dowiedzieć ile książek w danych latach zostało napisanych przez konkretne osoby.

Zapytanie: 
```
facet.pivot=author_str,year
```

### Facet Interwałowy
Służy do okreslenia ile znajduje się dokumentów w podanym przedziale.
Przykładowo:

```
facet.interval=year
facet.interval.set=[0,1000]

facet.interval=year&facet.interval.set=[0,1000]
```

Zwróci ilość książek napisanych w latach 0,1000 (Obustronnie zawierające, można ograniczyć zawieranie korzystając ze zwykłego nawiasu zamiast kwadratowego)



# Zadanka


## Zadanie 1

Napisz zapytanie, które zwróci nazwy wszystkich książek. (Tylko nazwy)

<details>
  <summary>Rozwiązanie:</summary>
  q: 
  <br>
  *:*
  <br>

  fl: 
  <br>
  title
  <br>
</details>

## Zadanie 2

Napisz zapytanie, które zwróci nazwy wszystkich książek napisanych w latach 1500-1650

<details>
  <summary>Rozwiązanie:</summary>
  q: 
  <br>
  year:[1500 TO 1650]
  <br>

  fl: 
  <br>
  title
  <br>
</details>


## Zadanie 3

Napisz zapytanie, które zwróci nazwy wszystkich książek w kolejności ich wydania

<details>
  <summary>Rozwiązanie:</summary>
  q: 
  <br>
  *:*
  <br>

  fl: 
  <br>
  title
  <br>

  sort:
  <br>
  year asc
  <br>

</details>


## Zadanie 4

Napisz zapytanie, które zwróci te książki, których tytuł zawiera słowo *Sound* oraz *Mountain*

<details>
  <summary>Rozwiązanie:</summary>
  q: 
  <br>
  title:(sound AND mountain)
  <br>
</details>


## Zadanie 5

Napisz zapytanie, które zwróci te książki,
których tytuł zawiera słowo *the* oraz *and* rozdzielone maksymalnie 2 słowami

<details>
  <summary>Rozwiązanie:</summary>
  q: 
  <br>
  title:"the and"~2
  <br>
</details>