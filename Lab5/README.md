# Исследование информации о состоянии беспроводных сетей
aleksandr.zwonov@yandex.ru

## Цель работы

1.  Получить знания о методах исследования радиоэлектронной обстановки.
2.  Составить представление о механизмах работы Wi-Fi сетей на канальном
    и сетевом уровне модели OSI.
3.  Зекрепить практические навыки использования языка программирования R
    для обработки данных
4.  Закрепить знания основных функций обработки данных экосистемы
    tidyverse языка R

## Исходные данные

1.  Программное обеспечение Windows 10 Pro
2.  Rstudio Desktop
3.  Интерпретатор языка R 4.5.1

## Задание

Используя программный пакет dplyr языка программирования R провести
анализ журналов и ответить на вопросы.

### Шаги

#### Установка пакетов

``` r
options(repos = c(CRAN = "https://cloud.r-project.org"))
install.packages("readr")
```

    пакет 'readr' успешно распакован, MD5-суммы проверены

    Warning: не могу удалить прежнюю установку пакета 'readr'

    Warning in file.copy(savedcopy, lib, recursive = TRUE): проблема с копированием
    D:\Programs\R-4.5.1\library\00LOCK\readr\libs\x64\readr.dll в
    D:\Programs\R-4.5.1\library\readr\libs\x64\readr.dll: Permission denied

    Warning: восстановлен 'readr'


    Скачанные бинарные пакеты находятся в
        C:\Users\zvono\AppData\Local\Temp\Rtmp67qAM3\downloaded_packages

``` r
install.packages("dplyr")
```

    пакет 'dplyr' успешно распакован, MD5-суммы проверены

    Warning: не могу удалить прежнюю установку пакета 'dplyr'

    Warning in file.copy(savedcopy, lib, recursive = TRUE): проблема с копированием
    D:\Programs\R-4.5.1\library\00LOCK\dplyr\libs\x64\dplyr.dll в
    D:\Programs\R-4.5.1\library\dplyr\libs\x64\dplyr.dll: Permission denied

    Warning: восстановлен 'dplyr'


    Скачанные бинарные пакеты находятся в
        C:\Users\zvono\AppData\Local\Temp\Rtmp67qAM3\downloaded_packages

``` r
install.packages("stringr")
```

    пакет 'stringr' успешно распакован, MD5-суммы проверены

    Скачанные бинарные пакеты находятся в
        C:\Users\zvono\AppData\Local\Temp\Rtmp67qAM3\downloaded_packages

``` r
install.packages("lubridate")
```

    пакет 'lubridate' успешно распакован, MD5-суммы проверены

    Warning: не могу удалить прежнюю установку пакета 'lubridate'

    Warning in file.copy(savedcopy, lib, recursive = TRUE): проблема с копированием
    D:\Programs\R-4.5.1\library\00LOCK\lubridate\libs\x64\lubridate.dll в
    D:\Programs\R-4.5.1\library\lubridate\libs\x64\lubridate.dll: Permission denied

    Warning: восстановлен 'lubridate'


    Скачанные бинарные пакеты находятся в
        C:\Users\zvono\AppData\Local\Temp\Rtmp67qAM3\downloaded_packages

``` r
library(readr)
```

    Warning: пакет 'readr' был собран под R версии 4.5.2

``` r
library(stringr)
```

    Warning: пакет 'stringr' был собран под R версии 4.5.2

``` r
library(dplyr)
```

    Warning: пакет 'dplyr' был собран под R версии 4.5.2


    Присоединяю пакет: 'dplyr'

    Следующие объекты скрыты от 'package:stats':

        filter, lag

    Следующие объекты скрыты от 'package:base':

        intersect, setdiff, setequal, union

``` r
library(lubridate)
```

    Warning: пакет 'lubridate' был собран под R версии 4.5.2


    Присоединяю пакет: 'lubridate'

    Следующие объекты скрыты от 'package:base':

        date, intersect, setdiff, union

#### Подготовка данных

1.  Импортируйте данные

``` r
lines <- readLines("P2_wifi_data.csv")

split_idx <- which(lines == "")[1]

ap_data <- read_csv(
  "P2_wifi_data.csv",
  skip = 0,
  n_max = split_idx - 2,  
  col_types = cols(.default = col_character())
)
```

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

``` r
client_data <- read_csv(
  "P2_wifi_data.csv",
  skip = split_idx,
  col_types = cols(.default = col_character())
)
```

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

1.  Привести датасеты в вид “аккуратных данных”, преобразовать типы
    столбцов в соответствии с типом данных

``` r
ap_clean <- ap_data %>%
  rename(
    BSSID = 1,
    First_time_seen = 2,
    Last_time_seen = 3,
    Channel = 4,
    Speed = 5,
    Privacy = 6,
    Cipher = 7,
    Authentication = 8,
    Power = 9,
    Beacons = 11,
    ESSID = 14
  ) %>%
  mutate(
    First_time_seen = ymd_hms(First_time_seen, tz = "UTC"),
    Last_time_seen = ymd_hms(Last_time_seen, tz = "UTC"),
    Speed = as.numeric(Speed),
    Power = as.numeric(Power),
    Beacons = as.numeric(Beacons),
    Channel = as.numeric(Channel)
  )
```

    Warning: There were 6 warnings in `mutate()`.
    The first warning was:
    ℹ In argument: `First_time_seen = ymd_hms(First_time_seen, tz = "UTC")`.
    Caused by warning:
    !  1 failed to parse.
    ℹ Run `dplyr::last_dplyr_warnings()` to see the 5 remaining warnings.

``` r
client_clean <- client_data %>%
  rename(
    Station_MAC = 1,
    First_time_seen = 2,
    Last_time_seen = 3,
    CPower = 4,
    Probes = 6
  ) %>%
  mutate(
    First_time_seen = ymd_hms(First_time_seen, tz = "UTC"),
    Last_time_seen = ymd_hms(Last_time_seen, tz = "UTC"),
    Power = as.numeric(Power)
  )
```

    Warning: There were 3 warnings in `mutate()`.
    The first warning was:
    ℹ In argument: `First_time_seen = ymd_hms(First_time_seen, tz = "UTC")`.
    Caused by warning:
    !  1 failed to parse.
    ℹ Run `dplyr::last_dplyr_warnings()` to see the 2 remaining warnings.

``` r
client_clean
```

    # A tibble: 12,249 × 15
       Station_MAC       First_time_seen     Last_time_seen      CPower Speed Probes
       <chr>             <dttm>              <dttm>              <chr>  <chr> <chr> 
     1 BE:F1:71:D5:17:8B 2023-07-28 09:13:03 2023-07-28 11:50:50 1      195   WPA2  
     2 6E:C7:EC:16:DA:1A 2023-07-28 09:13:03 2023-07-28 11:55:12 1      130   WPA2  
     3 9A:75:A8:B9:04:1E 2023-07-28 09:13:03 2023-07-28 11:53:31 1      360   WPA2  
     4 4A:EC:1E:DB:BF:95 2023-07-28 09:13:03 2023-07-28 11:04:01 7      360   WPA2  
     5 D2:6D:52:61:51:5D 2023-07-28 09:13:03 2023-07-28 10:30:19 6      130   WPA2  
     6 E8:28:C1:DC:B2:52 2023-07-28 09:13:03 2023-07-28 11:55:38 6      130   OPN   
     7 BE:F1:71:D6:10:D7 2023-07-28 09:13:03 2023-07-28 11:50:44 11     195   WPA2  
     8 0A:C5:E1:DB:17:7B 2023-07-28 09:13:03 2023-07-28 11:36:31 11     130   WPA2  
     9 38:1A:52:0D:84:D7 2023-07-28 09:13:03 2023-07-28 10:25:02 11     130   WPA2  
    10 BE:F1:71:D5:0E:53 2023-07-28 09:13:03 2023-07-28 10:29:21 1      195   WPA2  
    # ℹ 12,239 more rows
    # ℹ 9 more variables: Cipher <chr>, Authentication <chr>, Power <dbl>,
    #   `# beacons` <chr>, `# IV` <chr>, `LAN IP` <chr>, `ID-length` <chr>,
    #   ESSID <chr>, Key <chr>

1.  Просмотрите общую структуру данных с помощью функции glimpse()

``` r
ap_clean %>% glimpse()
```

    Rows: 12,249
    Columns: 15
    $ BSSID           <chr> "BE:F1:71:D5:17:8B", "6E:C7:EC:16:DA:1A", "9A:75:A8:B9…
    $ First_time_seen <dttm> 2023-07-28 09:13:03, 2023-07-28 09:13:03, 2023-07-28 …
    $ Last_time_seen  <dttm> 2023-07-28 11:50:50, 2023-07-28 11:55:12, 2023-07-28 …
    $ Channel         <dbl> 1, 1, 1, 7, 6, 6, 11, 11, 11, 1, 6, 14, 11, 11, 6, 6, …
    $ Speed           <dbl> 195, 130, 360, 360, 130, 130, 195, 130, 130, 195, 180,…
    $ Privacy         <chr> "WPA2", "WPA2", "WPA2", "WPA2", "WPA2", "OPN", "WPA2",…
    $ Cipher          <chr> "CCMP", "CCMP", "CCMP", "CCMP", "CCMP", NA, "CCMP", "C…
    $ Authentication  <chr> "PSK", "PSK", "PSK", "PSK", "PSK", NA, "PSK", "PSK", "…
    $ Power           <dbl> -30, -30, -68, -37, -57, -63, -27, -38, -38, -66, -42,…
    $ `# beacons`     <chr> "846", "750", "694", "510", "647", "251", "1647", "125…
    $ Beacons         <dbl> 504, 116, 26, 21, 6, 3430, 80, 11, 0, 0, 86, 0, 0, 0, …
    $ `LAN IP`        <chr> "0.  0.  0.  0", "0.  0.  0.  0", "0.  0.  0.  0", "0.…
    $ `ID-length`     <chr> "12", "4", "2", "14", "25", "13", "12", "13", "24", "1…
    $ ESSID           <chr> "C322U13 3965", "Cnet", "KC", "POCO X5 Pro 5G", NA, "M…
    $ Key             <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…

``` r
client_clean %>% glimpse()
```

    Rows: 12,249
    Columns: 15
    $ Station_MAC     <chr> "BE:F1:71:D5:17:8B", "6E:C7:EC:16:DA:1A", "9A:75:A8:B9…
    $ First_time_seen <dttm> 2023-07-28 09:13:03, 2023-07-28 09:13:03, 2023-07-28 …
    $ Last_time_seen  <dttm> 2023-07-28 11:50:50, 2023-07-28 11:55:12, 2023-07-28 …
    $ CPower          <chr> "1", "1", "1", "7", "6", "6", "11", "11", "11", "1", "…
    $ Speed           <chr> "195", "130", "360", "360", "130", "130", "195", "130"…
    $ Probes          <chr> "WPA2", "WPA2", "WPA2", "WPA2", "WPA2", "OPN", "WPA2",…
    $ Cipher          <chr> "CCMP", "CCMP", "CCMP", "CCMP", "CCMP", NA, "CCMP", "C…
    $ Authentication  <chr> "PSK", "PSK", "PSK", "PSK", "PSK", NA, "PSK", "PSK", "…
    $ Power           <dbl> -30, -30, -68, -37, -57, -63, -27, -38, -38, -66, -42,…
    $ `# beacons`     <chr> "846", "750", "694", "510", "647", "251", "1647", "125…
    $ `# IV`          <chr> "504", "116", "26", "21", "6", "3430", "80", "11", "0"…
    $ `LAN IP`        <chr> "0.  0.  0.  0", "0.  0.  0.  0", "0.  0.  0.  0", "0.…
    $ `ID-length`     <chr> "12", "4", "2", "14", "25", "13", "12", "13", "24", "1…
    $ ESSID           <chr> "C322U13 3965", "Cnet", "KC", "POCO X5 Pro 5G", NA, "M…
    $ Key             <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…

#### Анализ

##### Точки доступа

1.  Определить небезопасные точки доступа (без шифрования – OPN)

``` r
ap_clean %>%
  filter(Privacy == "OPN")
```

    # A tibble: 42 × 15
       BSSID    First_time_seen     Last_time_seen      Channel Speed Privacy Cipher
       <chr>    <dttm>              <dttm>                <dbl> <dbl> <chr>   <chr> 
     1 E8:28:C… 2023-07-28 09:13:03 2023-07-28 11:55:38       6   130 OPN     <NA>  
     2 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:12       6   130 OPN     <NA>  
     3 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:11       6   130 OPN     <NA>  
     4 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:10       6    -1 OPN     <NA>  
     5 00:25:0… 2023-07-28 09:13:06 2023-07-28 11:56:21      44    -1 OPN     <NA>  
     6 E8:28:C… 2023-07-28 09:13:09 2023-07-28 11:56:05      11   130 OPN     <NA>  
     7 E8:28:C… 2023-07-28 09:13:13 2023-07-28 10:27:06       6   130 OPN     <NA>  
     8 E8:28:C… 2023-07-28 09:13:13 2023-07-28 10:39:43       6   130 OPN     <NA>  
     9 E8:28:C… 2023-07-28 09:13:17 2023-07-28 11:52:32       1   130 OPN     <NA>  
    10 E8:28:C… 2023-07-28 09:13:50 2023-07-28 11:43:39      11   130 OPN     <NA>  
    # ℹ 32 more rows
    # ℹ 8 more variables: Authentication <chr>, Power <dbl>, `# beacons` <chr>,
    #   Beacons <dbl>, `LAN IP` <chr>, `ID-length` <chr>, ESSID <chr>, Key <chr>

1.  Определить производителя для каждого обнаруженного устройства

``` r
oui_db <- read_tsv("manuf.txt", col_names = c("OUI", "Vendor", "Comment"))
```

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 41477 Columns: 3
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: "\t"
    chr (3): OUI, Vendor, Comment

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
ap_clean %>%
  mutate(
    OUI = substr(BSSID, 1, 8)
  ) %>%
  left_join(oui_db %>% select(OUI, Vendor), by = "OUI")
```

    # A tibble: 12,249 × 17
       BSSID    First_time_seen     Last_time_seen      Channel Speed Privacy Cipher
       <chr>    <dttm>              <dttm>                <dbl> <dbl> <chr>   <chr> 
     1 BE:F1:7… 2023-07-28 09:13:03 2023-07-28 11:50:50       1   195 WPA2    CCMP  
     2 6E:C7:E… 2023-07-28 09:13:03 2023-07-28 11:55:12       1   130 WPA2    CCMP  
     3 9A:75:A… 2023-07-28 09:13:03 2023-07-28 11:53:31       1   360 WPA2    CCMP  
     4 4A:EC:1… 2023-07-28 09:13:03 2023-07-28 11:04:01       7   360 WPA2    CCMP  
     5 D2:6D:5… 2023-07-28 09:13:03 2023-07-28 10:30:19       6   130 WPA2    CCMP  
     6 E8:28:C… 2023-07-28 09:13:03 2023-07-28 11:55:38       6   130 OPN     <NA>  
     7 BE:F1:7… 2023-07-28 09:13:03 2023-07-28 11:50:44      11   195 WPA2    CCMP  
     8 0A:C5:E… 2023-07-28 09:13:03 2023-07-28 11:36:31      11   130 WPA2    CCMP  
     9 38:1A:5… 2023-07-28 09:13:03 2023-07-28 10:25:02      11   130 WPA2    CCMP  
    10 BE:F1:7… 2023-07-28 09:13:03 2023-07-28 10:29:21       1   195 WPA2    CCMP  
    # ℹ 12,239 more rows
    # ℹ 10 more variables: Authentication <chr>, Power <dbl>, `# beacons` <chr>,
    #   Beacons <dbl>, `LAN IP` <chr>, `ID-length` <chr>, ESSID <chr>, Key <chr>,
    #   OUI <chr>, Vendor <chr>

1.  Выявить устройства, использующие последнюю версию протокола
    шифрования WPA3, и названия точек доступа, реализованных на этих
    устройствах

``` r
ap_clean %>%
  filter(Privacy == "WPA3 WPA2")
```

    # A tibble: 8 × 15
      BSSID     First_time_seen     Last_time_seen      Channel Speed Privacy Cipher
      <chr>     <dttm>              <dttm>                <dbl> <dbl> <chr>   <chr> 
    1 26:20:53… 2023-07-28 09:15:45 2023-07-28 09:33:10      44   866 WPA3 W… CCMP  
    2 A2:FE:FF… 2023-07-28 09:41:52 2023-07-28 09:41:52       6   130 WPA3 W… CCMP  
    3 96:FF:FC… 2023-07-28 09:52:54 2023-07-28 10:25:02      44   866 WPA3 W… CCMP  
    4 CE:48:E7… 2023-07-28 09:59:20 2023-07-28 10:04:15      44   866 WPA3 W… CCMP  
    5 8E:1F:94… 2023-07-28 10:08:32 2023-07-28 10:15:27      44   866 WPA3 W… CCMP  
    6 BE:FD:EF… 2023-07-28 10:15:24 2023-07-28 10:15:28       6   130 WPA3 W… CCMP  
    7 3A:DA:00… 2023-07-28 10:27:01 2023-07-28 10:27:10       6   130 WPA3 W… CCMP  
    8 76:C5:A0… 2023-07-28 11:16:36 2023-07-28 11:16:38       6   130 WPA3 W… CCMP  
    # ℹ 8 more variables: Authentication <chr>, Power <dbl>, `# beacons` <chr>,
    #   Beacons <dbl>, `LAN IP` <chr>, `ID-length` <chr>, ESSID <chr>, Key <chr>

1.  Отсортировать точки доступа по интервалу времени, в течение которого
    они находились на связи, по убыванию.

``` r
ap_clean %>%
  arrange(BSSID, First_time_seen) %>%
  group_by(BSSID) %>%
  mutate(
    gap = First_time_seen - lag(Last_time_seen),
    new_session = is.na(gap) | gap > dminutes(45),
    session_id = cumsum(new_session)
  ) %>%
  group_by(BSSID, session_id) %>%
  summarise(
    session_start = min(First_time_seen),
    session_end = max(Last_time_seen),
    duration = session_end - session_start,
    .groups = "drop"
  ) %>%
  group_by(BSSID) %>%
  summarise(total_time = sum(duration), .groups = "drop") %>%
  arrange(desc(total_time))
```

    # A tibble: 12,241 × 2
       BSSID             total_time
       <chr>             <drtn>    
     1 00:25:00:FF:94:73 9795 secs 
     2 10:51:07:CB:33:BF 9784 secs 
     3 00:95:69:E7:7C:ED 9782 secs 
     4 00:95:69:E7:7D:21 9782 secs 
     5 10:51:07:CB:33:E7 9781 secs 
     6 8C:55:4A:DE:F2:38 9779 secs 
     7 00:95:69:E7:7F:35 9776 secs 
     8 E8:28:C1:DD:04:52 9776 secs 
     9 9E:A3:A9:D6:28:3C 9758 secs 
    10 E8:28:C1:DC:B2:52 9755 secs 
    # ℹ 12,231 more rows

1.  Обнаружить топ-10 самых быстрых точек доступа.

``` r
ap_clean %>%
  arrange(desc(Speed)) %>%
  head(10)
```

    # A tibble: 10 × 15
       BSSID    First_time_seen     Last_time_seen      Channel Speed Privacy Cipher
       <chr>    <dttm>              <dttm>                <dbl> <dbl> <chr>   <chr> 
     1 00:95:6… 2023-07-28 09:13:15 2023-07-28 11:56:17     -33  8171 (not a… nvrip…
     2 00:95:6… 2023-07-28 09:13:11 2023-07-28 11:56:13     -55  4096 (not a… nvrip…
     3 00:95:6… 2023-07-28 09:13:11 2023-07-28 11:56:07     -69  2245 (not a… nvrip…
     4 98:F6:2… 2023-07-28 10:41:15 2023-07-28 11:56:14     -59  2143 E8:28:… Beeli…
     5 F6:4D:9… 2023-07-28 09:14:37 2023-07-28 11:55:29     -61  1062 00:26:… GIVC  
     6 52:FE:C… 2023-07-28 09:31:57 2023-07-28 11:54:07     -57  1037 8E:55:… Vladi…
     7 C0:E4:3… 2023-07-28 09:13:03 2023-07-28 11:53:16     -61   958 BE:F1:… C322U…
     8 74:DF:B… 2023-07-28 09:45:48 2023-07-28 10:31:11     -65   911 E8:28:… MIREA…
     9 26:20:5… 2023-07-28 09:15:45 2023-07-28 09:33:10      44   866 WPA3 W… CCMP  
    10 96:FF:F… 2023-07-28 09:52:54 2023-07-28 10:25:02      44   866 WPA3 W… CCMP  
    # ℹ 8 more variables: Authentication <chr>, Power <dbl>, `# beacons` <chr>,
    #   Beacons <dbl>, `LAN IP` <chr>, `ID-length` <chr>, ESSID <chr>, Key <chr>

1.  Отсортировать точки доступа по частоте отправки запросов (beacons) в
    единицу времени по их убыванию

``` r
ap_clean %>%
  mutate(
    duration_sec = as.numeric(difftime(Last_time_seen, First_time_seen, units = "secs")),
    beacon_rate = Beacons / duration_sec
  ) %>%
  arrange(desc(beacon_rate))
```

    # A tibble: 12,249 × 17
       BSSID    First_time_seen     Last_time_seen      Channel Speed Privacy Cipher
       <chr>    <dttm>              <dttm>                <dbl> <dbl> <chr>   <chr> 
     1 E8:28:C… 2023-07-28 10:20:20 2023-07-28 10:20:20      11    -1 OPN     <NA>  
     2 30:B4:B… 2023-07-28 11:05:02 2023-07-28 11:05:02       2    -1 WPA     <NA>  
     3 00:26:9… 2023-07-28 11:10:12 2023-07-28 11:10:12      44    -1 OPN     <NA>  
     4 E8:28:C… 2023-07-28 09:13:03 2023-07-28 11:55:38       6   130 OPN     <NA>  
     5 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:12       6   130 OPN     <NA>  
     6 8E:55:4… 2023-07-28 09:13:06 2023-07-28 11:55:09       6    65 WPA2    CCMP  
     7 BE:F1:7… 2023-07-28 09:13:03 2023-07-28 11:50:50       1   195 WPA2    CCMP  
     8 00:26:9… 2023-07-28 09:13:20 2023-07-28 11:55:10      11    54 WPA2    CCMP  
     9 E8:28:C… 2023-07-28 09:13:09 2023-07-28 11:56:05      11   130 OPN     <NA>  
    10 5E:C7:C… 2023-07-28 09:34:56 2023-07-28 10:29:21       1   180 WPA2    CCMP  
    # ℹ 12,239 more rows
    # ℹ 10 more variables: Authentication <chr>, Power <dbl>, `# beacons` <chr>,
    #   Beacons <dbl>, `LAN IP` <chr>, `ID-length` <chr>, ESSID <chr>, Key <chr>,
    #   duration_sec <dbl>, beacon_rate <dbl>

##### Данные клиентов

1.  Определить производителя для каждого обнаруженного устройства

``` r
client_clean %>%
  mutate(
    OUI = substr(Station_MAC, 1, 8),
  ) %>%
  left_join(oui_db %>% select(OUI, Vendor), by = "OUI")
```

    # A tibble: 12,249 × 17
       Station_MAC       First_time_seen     Last_time_seen      CPower Speed Probes
       <chr>             <dttm>              <dttm>              <chr>  <chr> <chr> 
     1 BE:F1:71:D5:17:8B 2023-07-28 09:13:03 2023-07-28 11:50:50 1      195   WPA2  
     2 6E:C7:EC:16:DA:1A 2023-07-28 09:13:03 2023-07-28 11:55:12 1      130   WPA2  
     3 9A:75:A8:B9:04:1E 2023-07-28 09:13:03 2023-07-28 11:53:31 1      360   WPA2  
     4 4A:EC:1E:DB:BF:95 2023-07-28 09:13:03 2023-07-28 11:04:01 7      360   WPA2  
     5 D2:6D:52:61:51:5D 2023-07-28 09:13:03 2023-07-28 10:30:19 6      130   WPA2  
     6 E8:28:C1:DC:B2:52 2023-07-28 09:13:03 2023-07-28 11:55:38 6      130   OPN   
     7 BE:F1:71:D6:10:D7 2023-07-28 09:13:03 2023-07-28 11:50:44 11     195   WPA2  
     8 0A:C5:E1:DB:17:7B 2023-07-28 09:13:03 2023-07-28 11:36:31 11     130   WPA2  
     9 38:1A:52:0D:84:D7 2023-07-28 09:13:03 2023-07-28 10:25:02 11     130   WPA2  
    10 BE:F1:71:D5:0E:53 2023-07-28 09:13:03 2023-07-28 10:29:21 1      195   WPA2  
    # ℹ 12,239 more rows
    # ℹ 11 more variables: Cipher <chr>, Authentication <chr>, Power <dbl>,
    #   `# beacons` <chr>, `# IV` <chr>, `LAN IP` <chr>, `ID-length` <chr>,
    #   ESSID <chr>, Key <chr>, OUI <chr>, Vendor <chr>

1.  Обнаружить устройства, которые НЕ рандомизируют свой MAC адрес

``` r
client_clean %>%
  mutate(
    randbit = str_sub(Station_MAC, 2, 2),
    israndomized = randbit %in% c("2", "6", "a", "e", "A", "E")
  ) %>%
  filter(!israndomized)
```

    # A tibble: 317 × 17
       Station_MAC       First_time_seen     Last_time_seen      CPower Speed Probes
       <chr>             <dttm>              <dttm>              <chr>  <chr> <chr> 
     1 E8:28:C1:DC:B2:52 2023-07-28 09:13:03 2023-07-28 11:55:38 6      130   OPN   
     2 38:1A:52:0D:84:D7 2023-07-28 09:13:03 2023-07-28 10:25:02 11     130   WPA2  
     3 1C:7E:E5:8E:B7:DE 2023-07-28 09:13:04 2023-07-28 11:51:48 14     65    WPA2  
     4 38:1A:52:0D:97:60 2023-07-28 09:13:05 2023-07-28 10:21:11 11     130   WPA2  
     5 38:1A:52:0D:90:A1 2023-07-28 09:13:06 2023-07-28 11:37:27 11     130   WPA2  
     6 E8:28:C1:DC:B2:50 2023-07-28 09:13:06 2023-07-28 11:55:12 6      130   OPN   
     7 E8:28:C1:DC:B2:51 2023-07-28 09:13:06 2023-07-28 11:55:11 6      130   OPN   
     8 E8:28:C1:DC:FF:F2 2023-07-28 09:13:06 2023-07-28 11:55:10 6      -1    OPN   
     9 00:25:00:FF:94:73 2023-07-28 09:13:06 2023-07-28 11:56:21 44     -1    OPN   
    10 00:26:99:F2:7A:E2 2023-07-28 09:13:06 2023-07-28 11:54:53 1      54    WPA2  
    # ℹ 307 more rows
    # ℹ 11 more variables: Cipher <chr>, Authentication <chr>, Power <dbl>,
    #   `# beacons` <chr>, `# IV` <chr>, `LAN IP` <chr>, `ID-length` <chr>,
    #   ESSID <chr>, Key <chr>, randbit <chr>, israndomized <lgl>

1.  Кластеризовать запросы от устройств к точкам доступа по их именам.
    Определить время появления устройства в зоне радиовидимости и время
    выхода его из нее.

``` r
client_clean %>%
  group_by(ESSID) %>%
  summarise(
    first_time_seen = min(First_time_seen, na.rm = TRUE),
    last_time_seen  = max(Last_time_seen, na.rm = TRUE)
  )
```

    # A tibble: 65 × 3
       ESSID         first_time_seen     last_time_seen     
       <chr>         <dttm>              <dttm>             
     1 Amuler        2023-07-28 10:27:46 2023-07-28 10:27:46
     2 AndroidAP177B 2023-07-28 09:13:03 2023-07-28 11:36:31
     3 C239U51 0701  2023-07-28 10:42:14 2023-07-28 11:51:18
     4 C239U52 6706  2023-07-28 10:38:54 2023-07-28 11:55:11
     5 C239U53 6056  2023-07-28 10:43:11 2023-07-28 11:53:35
     6 C322U06 9080  2023-07-28 09:13:03 2023-07-28 10:29:21
     7 C322U13 3965  2023-07-28 09:13:03 2023-07-28 11:50:50
     8 C322U21 0566  2023-07-28 09:13:03 2023-07-28 11:50:44
     9 Christie’s    2023-07-28 09:41:52 2023-07-28 09:41:52
    10 Cnet          2023-07-28 09:13:03 2023-07-28 11:55:12
    # ℹ 55 more rows

1.  Оценить стабильность уровня сигнала внури кластера во времени.
    Выявить наиболее стабильный кластер

``` r
client_clean %>%
  left_join(ap_clean %>% select(ESSID, BSSID), by = c("Probes" = "ESSID")) %>%
  group_by(Probes) %>%
  summarise(
    mean_power = mean(Power, na.rm = TRUE),
    sd_power = sd(Power, na.rm = TRUE),
    n = n(),
    .groups = "drop"
  ) %>%
  arrange(sd_power)
```

    Warning in left_join(., ap_clean %>% select(ESSID, BSSID), by = c(Probes = "ESSID")): Detected an unexpected many-to-many relationship between `x` and `y`.
    ℹ Row 45 of `x` matches multiple rows in `y`.
    ℹ Row 5 of `y` matches multiple rows in `x`.
    ℹ If a many-to-many relationship is expected, set `relationship =
      "many-to-many"` to silence this warning.

    # A tibble: 81 × 4
       Probes            mean_power sd_power      n
       <chr>                  <dbl>    <dbl>  <int>
     1 (not associated)        1         0    11895
     2 WPA3 WPA2             -69.1      11.1      8
     3 WPA2                  -64.5      14.2     72
     4 <NA>                   -6.81     18.2 522579
     5 OPN                   -63.3      26.7     42
     6 WPA                   -36.5      50.2      2
     7 00:03:7F:10:17:56     NaN        NA        1
     8 00:0D:97:6B:93:DF     NaN        NA        1
     9 00:23:EB:E3:49:31     NaN        NA        1
    10 00:25:00:FF:94:73     NaN        NA       45
    # ℹ 71 more rows

## Вывод

В ходе работы были успешно получены знания о методах исследования
радиоэлектронной обстановки и составленно представление о механизмах
работы Wi-Fi сетей на канальном и сетевом уровне модели OSI, а также
закреплены практические навыки использования языка программирования R
для обработки данных и знания основных функций обработки данных
экосистемы tidyverse языка R.
