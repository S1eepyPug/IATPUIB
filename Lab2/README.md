# Основы обработки данных с помощью R
aleksandr.zwonov@yandex.ru

## Цель работы

1.  Развить практические навыки использования языка программирования R
    для обработки данных
2.  Закрепить знания базовых типов данных языка R
3.  Развить практические навыки использования функций обработки данных
    пакета `dplyr` – функции
    `select(), filter(), mutate(), arrange(), group_by()`

## Исходные данные

1.  Программное обеспечение Windows 10 Pro
2.  Rstudio Desktop
3.  Интерпретатор языка R 4.5.1

## Задание

Проанализировать встроенный в пакет dplyr набор данных starwars с
помощью языка R и ответить на вопросы.

### Шаги

#### Установим пакет dplyr

``` r
library(dplyr)
```


    Присоединяю пакет: 'dplyr'

    Следующие объекты скрыты от 'package:stats':

        filter, lag

    Следующие объекты скрыты от 'package:base':

        intersect, setdiff, setequal, union

#### Проведем анализ датафрейма starwars

1\. Сколько строк в датафрейме?

``` r
starwars %>% nrow()
```

    [1] 87

2\. Сколько столбцов в датафрейме?

``` r
starwars %>% ncol()
```

    [1] 14

3\. Как просмотреть примерный вид датафрейма?

``` r
starwars %>% glimpse()
```

    Rows: 87
    Columns: 14
    $ name       <chr> "Luke Skywalker", "C-3PO", "R2-D2", "Darth Vader", "Leia Or…
    $ height     <int> 172, 167, 96, 202, 150, 178, 165, 97, 183, 182, 188, 180, 2…
    $ mass       <dbl> 77.0, 75.0, 32.0, 136.0, 49.0, 120.0, 75.0, 32.0, 84.0, 77.…
    $ hair_color <chr> "blond", NA, NA, "none", "brown", "brown, grey", "brown", N…
    $ skin_color <chr> "fair", "gold", "white, blue", "white", "light", "light", "…
    $ eye_color  <chr> "blue", "yellow", "red", "yellow", "brown", "blue", "blue",…
    $ birth_year <dbl> 19.0, 112.0, 33.0, 41.9, 19.0, 52.0, 47.0, NA, 24.0, 57.0, …
    $ sex        <chr> "male", "none", "none", "male", "female", "male", "female",…
    $ gender     <chr> "masculine", "masculine", "masculine", "masculine", "femini…
    $ homeworld  <chr> "Tatooine", "Tatooine", "Naboo", "Tatooine", "Alderaan", "T…
    $ species    <chr> "Human", "Droid", "Droid", "Human", "Human", "Human", "Huma…
    $ films      <list> <"A New Hope", "The Empire Strikes Back", "Return of the J…
    $ vehicles   <list> <"Snowspeeder", "Imperial Speeder Bike">, <>, <>, <>, "Imp…
    $ starships  <list> <"X-wing", "Imperial shuttle">, <>, <>, "TIE Advanced x1",…

4\. Сколько уникальных рас персонажей (species) представлено в данных?

``` r
starwars %>% select(species) %>% filter(!is.na(species)) %>% unique() %>% count() 
```

    # A tibble: 1 × 1
          n
      <int>
    1    37

5\. Найти самого высокого персонажа.

``` r
starwars %>% select(name, height) %>% arrange(desc(height)) %>% head(1) 
```

    # A tibble: 1 × 2
      name        height
      <chr>        <int>
    1 Yarael Poof    264

6\. Найти всех персонажей ниже 170.

``` r
starwars%>% select(name, height) %>% filter(height < 170)
```

    # A tibble: 22 × 2
       name                  height
       <chr>                  <int>
     1 C-3PO                    167
     2 R2-D2                     96
     3 Leia Organa              150
     4 Beru Whitesun Lars       165
     5 R5-D4                     97
     6 Yoda                      66
     7 Mon Mothma               150
     8 Wicket Systri Warrick     88
     9 Nien Nunb                160
    10 Watto                    137
    # ℹ 12 more rows

7\. Подсчитать ИМТ (индекс массы тела) для всех персонажей по формуле.

``` r
starwars %>% mutate(res = round(mass/(height^2),5)) %>% select(name, res)
```

    # A tibble: 87 × 2
       name                   res
       <chr>                <dbl>
     1 Luke Skywalker     0.0026 
     2 C-3PO              0.00269
     3 R2-D2              0.00347
     4 Darth Vader        0.00333
     5 Leia Organa        0.00218
     6 Owen Lars          0.00379
     7 Beru Whitesun Lars 0.00275
     8 R5-D4              0.0034 
     9 Biggs Darklighter  0.00251
    10 Obi-Wan Kenobi     0.00232
    # ℹ 77 more rows

8\. Найти 10 самых “вытянутых” персонажей. “Вытянутость” оценить по
отношению массы (mass) к росту (height) персонажей.

``` r
starwars %>% mutate(res = round(mass/height,5)) %>% select(name, res) %>% arrange(res) %>% head(10)
```

    # A tibble: 10 × 2
       name                    res
       <chr>                 <dbl>
     1 Ratts Tyerel          0.190
     2 Wicket Systri Warrick 0.227
     3 Padmé Amidala         0.243
     4 Wat Tambor            0.249
     5 Yoda                  0.258
     6 Sly Moore             0.270
     7 Adi Gallia            0.272
     8 Barriss Offee         0.301
     9 Ayla Secura           0.309
    10 Shaak Ti              0.320

9\. Найти средний возраст персонажей каждой расы вселенной Звездных
войн.

``` r
starwars %>% filter(!is.na(species)) %>% filter(!is.na(birth_year)) %>% group_by( species) %>% summarise(mean_age = mean(100 - birth_year)) %>% select(species, mean_age)
```

    # A tibble: 15 × 2
       species        mean_age
       <chr>             <dbl>
     1 Cerean              8  
     2 Droid              46.7
     3 Ewok               92  
     4 Gungan             48  
     5 Human              46.3
     6 Hutt             -500  
     7 Kel Dor            78  
     8 Mirialan           51  
     9 Mon Calamari       59  
    10 Rodian             56  
    11 Trandoshan         47  
    12 Twi'lek            52  
    13 Wookiee          -100  
    14 Yoda's species   -796  
    15 Zabrak             46  

10\. Найти самый распространенный цвет глаз персонажей вселенной
Звездных войн.

``` r
starwars %>% group_by(eye_color) %>% summarise(count = n()) %>% select(eye_color, count) %>% arrange(desc(count)) %>% head(1)
```

    # A tibble: 1 × 2
      eye_color count
      <chr>     <int>
    1 brown        21

11\. Подсчитать среднюю длину имени в каждой расе вселенной Звездных
войн.

``` r
starwars %>% filter(!is.na(species)) %>% group_by(species) %>% summarise(mean_len = mean(nchar(name))) %>% select(species, mean_len)
```

    # A tibble: 37 × 2
       species   mean_len
       <chr>        <dbl>
     1 Aleena       12   
     2 Besalisk     15   
     3 Cerean       12   
     4 Chagrian     10   
     5 Clawdite     10   
     6 Droid         4.83
     7 Dug           7   
     8 Ewok         21   
     9 Geonosian    17   
    10 Gungan       11.7 
    # ℹ 27 more rows

## Вывод

В ходе работы был изучен пакет dplyr и улучшены практические навыки
использования языка программирования R,
