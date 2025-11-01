# Основы обработки данных с помощью R и Dplyr
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

Проанализировать встроенные в пакет nycflights13 наборы данных с помощью
языка R и ответить на вопросы.

### Шаги

#### Установим и подключим пакеты

``` r
library(nycflights13)
```

``` r
library(dplyr)
```


    Присоединяю пакет: 'dplyr'

    Следующие объекты скрыты от 'package:stats':

        filter, lag

    Следующие объекты скрыты от 'package:base':

        intersect, setdiff, setequal, union

#### Проанализируем встроенные в пакет nycflights13 наборы данных

1\. Сколько встроенных в пакет nycflights13 датафреймов?

``` r
data(package = "nycflights13")$results[, "Item"]
```

    [1] "airlines" "airports" "flights"  "planes"   "weather" 

``` r
length(data(package = "nycflights13")$results[, "Item"])
```

    [1] 5

2\. Сколько строк в каждом датафрейме?

``` r
airlines |> nrow()
```

    [1] 16

``` r
airports |> nrow()
```

    [1] 1458

``` r
flights |> nrow()
```

    [1] 336776

``` r
planes |> nrow()
```

    [1] 3322

``` r
weather |> nrow()
```

    [1] 26115

3\. Сколько столбцов в каждом датафрейме?

``` r
airlines |> ncol()
```

    [1] 2

``` r
airports |> ncol()
```

    [1] 8

``` r
flights |> ncol()
```

    [1] 19

``` r
planes |> ncol()
```

    [1] 9

``` r
weather |> ncol()
```

    [1] 15

4\. Как просмотреть примерный вид датафрейма?

``` r
airlines |> glimpse()
```

    Rows: 16
    Columns: 2
    $ carrier <chr> "9E", "AA", "AS", "B6", "DL", "EV", "F9", "FL", "HA", "MQ", "O…
    $ name    <chr> "Endeavor Air Inc.", "American Airlines Inc.", "Alaska Airline…

5\. Сколько компаний-перевозчиков (carrier) учитывают эти наборы данных
(представлено в наборах данных)?

``` r
airlines |> select(carrier) |> unique() |> count()
```

    # A tibble: 1 × 1
          n
      <int>
    1    16

6\. Сколько рейсов принял аэропорт John F Kennedy Intl в мае?

``` r
left_join(flights, airports, by = c("origin"="faa")) |> filter(name == "John F Kennedy Intl" & month == 5) |> count()
```

    # A tibble: 1 × 1
          n
      <int>
    1  9397

7\. Какой самый северный аэропорт?

``` r
arrange(airports, desc(lon)) |> select(name) |>  head(1)
```

    # A tibble: 1 × 1
      name        
      <chr>       
    1 Eareckson As

8\. Какой аэропорт самый высокогорный (находится выше всех над уровнем
моря)?

``` r
arrange(airports, desc(alt)) |> select(name) |>  head(1)
```

    # A tibble: 1 × 1
      name     
      <chr>    
    1 Telluride

9\. Какие бортовые номера у самых старых самолетов?

``` r
arrange(planes, year) |> select(tailnum) |> head()
```

    # A tibble: 6 × 1
      tailnum
      <chr>  
    1 N381AA 
    2 N201AA 
    3 N567AA 
    4 N378AA 
    5 N575AA 
    6 N14629 

10\. Какая средняя температура воздуха была в сентябре в аэропорту John
F Kennedy Intl (в градусах Цельсия).

``` r
left_join(weather, airports, by=c("origin"="faa")) |> filter(name == "John F Kennedy Intl" & month == 9) |> summarise(mean_temp = (mean(temp)- 32) / 1.8)
```

    # A tibble: 1 × 1
      mean_temp
          <dbl>
    1      19.4

11\. Самолеты какой авиакомпании совершили больше всего вылетов в июне?

``` r
left_join(flights, airlines, join_by(carrier)) |> filter(month == 6) |> group_by(name) |> summarise(amount = n()) |> arrange(desc(amount)) |> head(1) |> select(name)
```

    # A tibble: 1 × 1
      name                 
      <chr>                
    1 United Air Lines Inc.

12\. Самолеты какой авиакомпании задерживались чаще других в 2013 году?

``` r
left_join(flights, airlines, join_by(carrier)) |> group_by(name) |> filter( arr_delay > 0 & year == 2013) |> group_by(name) |> summarise(n = n()) |> arrange(desc(n)) |> select(name) |> head(1)
```

    # A tibble: 1 × 1
      name                    
      <chr>                   
    1 ExpressJet Airlines Inc.

## Вывод

В ходе работы были проанализированы встроенные в пакет nycflights13
наборы данных и улучшены практические навыки использования языка
программирования R.
