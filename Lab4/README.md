# Исследование метаданных DNS трафика
aleksandr.zwonov@yandex.ru

## Цель работы

1.  Закрепить практические навыки использования языка программирования R
    для обработки данных
2.  Закрепить знания основных функций обработки данных экосистемы
    tidyverse языка R
3.  Закрепить навыки исследования метаданных DNS трафика

## Исходные данные

1.  Программное обеспечение Windows 10 Pro
2.  Rstudio Desktop
3.  Интерпретатор языка R 4.5.1

## Задание

Используя программный пакет dplyr, освоить анализ DNS логов с помощью
языка программирования R.

### Шаги

Установим и подключим необходимые библиотеки

``` r
options(repos = c(CRAN = "https://cran.rstudio.com/"))
install.packages("dplyr")
```

    пакет 'dplyr' успешно распакован, MD5-суммы проверены

    Warning: не могу удалить прежнюю установку пакета 'dplyr'

    Warning in file.copy(savedcopy, lib, recursive = TRUE): проблема с копированием
    D:\Programs\R-4.5.1\library\00LOCK\dplyr\libs\x64\dplyr.dll в
    D:\Programs\R-4.5.1\library\dplyr\libs\x64\dplyr.dll: Permission denied

    Warning: восстановлен 'dplyr'


    Скачанные бинарные пакеты находятся в
        C:\Users\zvono\AppData\Local\Temp\RtmpuC8QSh\downloaded_packages

``` r
install.packages("tidyverse")
```

    пакет 'tidyverse' успешно распакован, MD5-суммы проверены

    Скачанные бинарные пакеты находятся в
        C:\Users\zvono\AppData\Local\Temp\RtmpuC8QSh\downloaded_packages

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
library(tidyverse)
```

    Warning: пакет 'tidyverse' был собран под R версии 4.5.2

    Warning: пакет 'ggplot2' был собран под R версии 4.5.2

    Warning: пакет 'tibble' был собран под R версии 4.5.2

    Warning: пакет 'tidyr' был собран под R версии 4.5.2

    Warning: пакет 'readr' был собран под R версии 4.5.2

    Warning: пакет 'purrr' был собран под R версии 4.5.2

    Warning: пакет 'stringr' был собран под R версии 4.5.2

    Warning: пакет 'forcats' был собран под R версии 4.5.2

    Warning: пакет 'lubridate' был собран под R версии 4.5.2

    ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ✔ forcats   1.0.1     ✔ readr     2.1.6
    ✔ ggplot2   4.0.1     ✔ stringr   1.6.0
    ✔ lubridate 1.9.4     ✔ tibble    3.3.0
    ✔ purrr     1.2.0     ✔ tidyr     1.3.1

    ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ✖ dplyr::filter() masks stats::filter()
    ✖ dplyr::lag()    masks stats::lag()
    ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

1.  Импортируйте данные DNS.

``` r
download.file("https://storage.yandexcloud.net/dataset.ctfsec/dns.zip", "dns.zip")
```

``` r
unzip("dns.zip")
```

``` r
data <- read_tsv("dns.log", comment = "#", col_names = FALSE)
```

    Rows: 427935 Columns: 23
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: "\t"
    chr (13): X2, X3, X5, X7, X9, X10, X11, X12, X13, X14, X15, X21, X22
    dbl  (5): X1, X4, X6, X8, X20
    lgl  (5): X16, X17, X18, X19, X23

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

1.  Добавьте пропущенные данные о структуре данных (назначении
    столбцов).

``` r
colnames(data) <- c(
  "timestamp", "uid", "source_ip", "source_port", "destination_ip", 
  "destination_port", "protocol", "transaction_id", "query", "qclass", 
  "qclass_name", "qtype", "qtype_name", "rcode", "rcode_name", 
  "AA", "TC", "RD", "RA", "Z", "answer", "TTLS", "rejected"
)
```

1.  Преобразуйте данные в столбцах в нужный формат.

``` r
data <- data %>% mutate(
    timestamp = as.POSIXct(timestamp, origin = "1970-01-01"),
    source_port = as.numeric(source_port),
    destination_port = as.numeric(destination_port),
    transaction_id = as.numeric(transaction_id),
    qclass = as.numeric(qclass),
    qtype = as.numeric(qtype),
    rcode = as.numeric(rcode),
  ) 
```

    Warning: There were 3 warnings in `mutate()`.
    The first warning was:
    ℹ In argument: `qclass = as.numeric(qclass)`.
    Caused by warning:
    ! в результате преобразования созданы NA
    ℹ Run `dplyr::last_dplyr_warnings()` to see the 2 remaining warnings.

1.  Просмотрите общую структуру данных с помощью функции glimpse().

``` r
glimpse(data)
```

    Rows: 427,935
    Columns: 23
    $ timestamp        <dttm> 2012-03-16 16:30:05, 2012-03-16 16:30:15, 2012-03-16…
    $ uid              <chr> "CWGtK431H9XuaTN4fi", "C36a282Jljz7BsbGH", "C36a282Jl…
    $ source_ip        <chr> "192.168.202.100", "192.168.202.76", "192.168.202.76"…
    $ source_port      <dbl> 45658, 137, 137, 137, 137, 137, 137, 137, 137, 137, 1…
    $ destination_ip   <chr> "192.168.27.203", "192.168.202.255", "192.168.202.255…
    $ destination_port <dbl> 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137…
    $ protocol         <chr> "udp", "udp", "udp", "udp", "udp", "udp", "udp", "udp…
    $ transaction_id   <dbl> 33008, 57402, 57402, 57402, 57398, 57398, 57398, 6218…
    $ query            <chr> "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\…
    $ qclass           <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,…
    $ qclass_name      <chr> "C_INTERNET", "C_INTERNET", "C_INTERNET", "C_INTERNET…
    $ qtype            <dbl> 33, 32, 32, 32, 32, 32, 32, 32, 32, 32, 33, 33, 33, 1…
    $ qtype_name       <chr> "SRV", "NB", "NB", "NB", "NB", "NB", "NB", "NB", "NB"…
    $ rcode            <dbl> 0, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ rcode_name       <chr> "NOERROR", "-", "-", "-", "-", "-", "-", "-", "-", "-…
    $ AA               <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALS…
    $ TC               <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALS…
    $ RD               <lgl> FALSE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE…
    $ RA               <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALS…
    $ Z                <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 1, 1, 1,…
    $ answer           <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-"…
    $ TTLS             <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-"…
    $ rejected         <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALS…

1.  Сколько участников информационного обмена в сети Доброй Организации?

``` r
length(unique(c(data$source_ip, data$destination_ip)))
```

    [1] 1359

1.  Какое соотношение участников обмена внутри сети и участников
    обращений к внешним ресурсам?

``` r
install.packages("ipaddress")
```

    пакет 'ipaddress' успешно распакован, MD5-суммы проверены

    Warning: не могу удалить прежнюю установку пакета 'ipaddress'

    Warning in file.copy(savedcopy, lib, recursive = TRUE): проблема с копированием
    D:\Programs\R-4.5.1\library\00LOCK\ipaddress\libs\x64\ipaddress.dll в
    D:\Programs\R-4.5.1\library\ipaddress\libs\x64\ipaddress.dll: Permission denied

    Warning: восстановлен 'ipaddress'


    Скачанные бинарные пакеты находятся в
        C:\Users\zvono\AppData\Local\Temp\RtmpuC8QSh\downloaded_packages

``` r
library(ipaddress)
```

    Warning: пакет 'ipaddress' был собран под R версии 4.5.2

``` r
ip_addresses <- ip_address(data %>% distinct(destination_ip) %>% pull(destination_ip))
internal <- sum(is_private(ip_addresses), na.rm = TRUE)
external <- sum(!is_private(ip_addresses), na.rm = TRUE)
internal / external
```

    [1] 28.28571

1.  Найдите топ-10 участников сети, проявляющих наибольшую сетевую
    активность.

``` r
data %>%
  count(source_ip, sort = TRUE) %>%
  head(10)
```

    # A tibble: 10 × 2
       source_ip           n
       <chr>           <int>
     1 10.10.117.210   75943
     2 192.168.202.93  26522
     3 192.168.202.103 18121
     4 192.168.202.76  16978
     5 192.168.202.97  16176
     6 192.168.202.141 14967
     7 10.10.117.209   14222
     8 192.168.202.110 13372
     9 192.168.203.63  12148
    10 192.168.202.106 10784

1.  Найдите топ-10 доменов, к которым обращаются пользователи сети и
    соответственное количество обращений.

``` r
data |>  count(query, sort = TRUE) |>  head(10)
```

    # A tibble: 10 × 2
       query                                                                       n
       <chr>                                                                   <int>
     1 "teredo.ipv6.microsoft.com"                                             39273
     2 "tools.google.com"                                                      14057
     3 "www.apple.com"                                                         13390
     4 "time.apple.com"                                                        13109
     5 "safebrowsing.clients.google.com"                                       11658
     6 "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x… 10401
     7 "WPAD"                                                                   9134
     8 "44.206.168.192.in-addr.arpa"                                            7248
     9 "HPE8AA67"                                                               6929
    10 "ISATAP"                                                                 6569

1.  Опеределите базовые статистические характеристики (функция summary()
    ) интервала времени между последовательным обращениями к топ-10
    доменам.

``` r
top_10_queries <- data %>%
  count(query, sort = TRUE) %>%
  head(10) %>%
  pull(query)

intervals_by_query <- data %>%
  filter(query %in% top_10_queries) %>%
  arrange(query, timestamp) %>%
  group_by(query) %>%
  mutate(
    time_diff_sec = c(NA, diff(as.numeric(timestamp))) 
  ) %>%
  ungroup()

summary_list <- intervals_by_query %>%
  group_by(query) %>%
  summarise(
    interval_summary = list(summary(time_diff_sec, na.rm = TRUE)),
    .groups = "drop"
  )


for (i in seq_len(nrow(summary_list))) {
  q <- summary_list$query[i]
  s <- summary_list$interval_summary[[i]]
  cat("Домен: ", q, "\n")
  print(s)
}
```

    Домен:  *\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00 
        Min.  1st Qu.   Median     Mean  3rd Qu.     Max.     NA's 
        0.00     0.15     0.50    11.24     1.50 52723.50        1 
    Домен:  44.206.168.192.in-addr.arpa 
        Min.  1st Qu.   Median     Mean  3rd Qu.     Max.     NA's 
        0.00     2.09     4.00    16.01    20.09 49679.81        1 
    Домен:  HPE8AA67 
        Min.  1st Qu.   Median     Mean  3rd Qu.     Max.     NA's 
        0.00     0.75     0.75    16.61    25.49 50044.43        1 
    Домен:  ISATAP 
        Min.  1st Qu.   Median     Mean  3rd Qu.     Max.     NA's 
        0.00     0.75     0.76    17.46     1.05 51997.79        1 
    Домен:  WPAD 
        Min.  1st Qu.   Median     Mean  3rd Qu.     Max.     NA's 
        0.00     0.75     0.75    12.61     1.11 50049.11        1 
    Домен:  safebrowsing.clients.google.com 
        Min.  1st Qu.   Median     Mean  3rd Qu.     Max.     NA's 
        0.00     0.00     1.00    10.00     2.01 49952.32        1 
    Домен:  teredo.ipv6.microsoft.com 
         Min.   1st Qu.    Median      Mean   3rd Qu.      Max.      NA's 
        0.000     0.000     0.000     2.941     0.510 50387.760         1 
    Домен:  time.apple.com 
         Min.   1st Qu.    Median      Mean   3rd Qu.      Max.      NA's 
        0.000     0.370     1.760     8.665     4.723 50924.280         1 
    Домен:  tools.google.com 
         Min.   1st Qu.    Median      Mean   3rd Qu.      Max.      NA's 
        0.000     0.000     0.000     8.187     1.000 50364.830         1 
    Домен:  www.apple.com 
         Min.   1st Qu.    Median      Mean   3rd Qu.      Max.      NA's 
        0.000     0.000     1.000     8.607     3.010 50963.630         1 

1.  Часто вредоносное программное обеспечение использует DNS канал в
    качестве канала управления, периодически отправляя запросы на
    подконтрольный злоумышленникам DNS сервер. По периодическим запросам
    на один и тот же домен можно выявить скрытый DNS канал. Есть ли
    такие IP адреса в исследуемом датасете?

``` r
pot_adrs <- data %>%
  group_by(source_ip, query) %>%
  summarise(request_count = n(), .groups = "drop") %>%
  filter(request_count > 10) %>%
  arrange(desc(request_count))


periodic_requests <- data %>%
  semi_join(pot_adrs, by = c("source_ip", "query")) %>%  
  arrange(source_ip, query, timestamp) %>%
  group_by(source_ip, query) %>%
  mutate(
    time_diff = as.numeric(difftime(timestamp, lag(timestamp), units = "secs"))
  ) %>%
  filter(!is.na(time_diff)) %>%
  summarise(
    request_count = n() + 1,
    mean_interval = mean(time_diff),
    sd_interval = sd(time_diff),
    cv = ifelse(mean_interval > 0, sd_interval / mean_interval, Inf),
    .groups = "drop"
  ) %>%
  filter(cv < 0.5) %>%
  arrange(cv)

periodic_requests
```

    # A tibble: 75 × 6
       source_ip       query         request_count mean_interval sd_interval      cv
       <chr>           <chr>                 <dbl>         <dbl>       <dbl>   <dbl>
     1 192.168.203.45  technet.micr…            16          5.00     0.00414 8.28e-4
     2 192.168.203.64  maps.google.…            32          5.00     0.00445 8.89e-4
     3 192.168.202.94  www.iana.org             32          5.00     0.00445 8.89e-4
     4 192.168.203.64  www.ticketsc…            16          5.00     0.00458 9.15e-4
     5 192.168.202.112 example.com              16          5.00     0.00458 9.15e-4
     6 192.168.202.79  www.ietf.org             16          5.00     0.00458 9.15e-4
     7 192.168.202.79  dd.cron.ru               16          5.00     0.00458 9.15e-4
     8 192.168.202.112 addons.mozil…            16          5.00     0.00458 9.15e-4
     9 192.168.202.94  input.mozill…            32          5.00     0.00475 9.50e-4
    10 192.168.202.94  www.google.c…            32          5.00     0.00475 9.50e-4
    # ℹ 65 more rows

1.  Определите местоположение (страну, город) и организацию-провайдера
    для топ-10 доменов.

``` r
install.packages(c("httr", "jsonlite"))
```

    пакет 'httr' успешно распакован, MD5-суммы проверены
    пакет 'jsonlite' успешно распакован, MD5-суммы проверены

    Warning: не могу удалить прежнюю установку пакета 'jsonlite'

    Warning in file.copy(savedcopy, lib, recursive = TRUE): проблема с копированием
    D:\Programs\R-4.5.1\library\00LOCK\jsonlite\libs\x64\jsonlite.dll в
    D:\Programs\R-4.5.1\library\jsonlite\libs\x64\jsonlite.dll: Permission denied

    Warning: восстановлен 'jsonlite'


    Скачанные бинарные пакеты находятся в
        C:\Users\zvono\AppData\Local\Temp\RtmpuC8QSh\downloaded_packages

``` r
library(httr)
```

    Warning: пакет 'httr' был собран под R версии 4.5.2

``` r
library(jsonlite)
```

    Warning: пакет 'jsonlite' был собран под R версии 4.5.2


    Присоединяю пакет: 'jsonlite'

    Следующий объект скрыт от 'package:purrr':

        flatten

``` r
top_domains <- data %>%
  count(query, sort = TRUE) %>%
  head(10)

is_private <- function(ips) {
  sapply(ips, function(ip) {
    if (is.na(ip)) return(TRUE)
    parts <- unlist(strsplit(ip, "\\."))
    if (length(parts) != 4) return(TRUE)
    nums <- suppressWarnings(as.numeric(parts))
    if (any(is.na(nums))) return(TRUE)
    a <- nums[1]; b <- nums[2]
    (a == 10) || (a == 172 && b >= 16 && b <= 31) || (a == 192 && b == 168)
  })
}

domain_to_ip <- data %>%
  filter(query %in% top_domains$query) %>%
  filter(!is_private(destination_ip)) %>%
  count(query, destination_ip, sort = TRUE) %>%
  group_by(query) %>%
  slice(1) %>%
  ungroup() %>%
  select(domain = query, ip = destination_ip)

if (nrow(domain_to_ip) == 0) {
  domain_to_ip <- data %>%
    filter(query %in% top_domains$query) %>%
    count(query, destination_ip, sort = TRUE) %>%
    group_by(query) %>%
    slice(1) %>%
    ungroup() %>%
    select(domain = query, ip = destination_ip)
}

get_geo <- function(ip) {
  url <- paste0("http://ip-api.com/json/", ip, "?fields=country,city,org,isp,status")
  Sys.sleep(1.5)
  
  resp <- tryCatch({
    GET(url, timeout(10))
  }, error = function(e) NULL)
  
  if (is.null(resp) || status_code(resp) != 200) {
    return(data.frame(ip = ip, country = NA, city = NA, org = NA, stringsAsFactors = FALSE))
  }
  
  txt <- content(resp, "text")
  parsed <- tryCatch(fromJSON(txt, simplifyVector = TRUE), error = function(e) NULL)
  
  if (is.null(parsed) || is.null(parsed$status) || parsed$status != "success") {
    return(data.frame(ip = ip, country = NA, city = NA, org = NA, stringsAsFactors = FALSE))
  }
  
  org_value <- ifelse(!is.null(parsed$org) && parsed$org != "", 
                     parsed$org, 
                     ifelse(!is.null(parsed$isp) && parsed$isp != "", 
                            parsed$isp, 
                            NA))
  
  data.frame(
    ip = ip,
    country = ifelse(is.null(parsed$country), NA, as.character(parsed$country)),
    city = ifelse(is.null(parsed$city), NA, as.character(parsed$city)),
    org = org_value,
    stringsAsFactors = FALSE
  )
}

results_list <- list()
for (i in 1:nrow(domain_to_ip)) {
  geo <- get_geo(domain_to_ip$ip[i])
  results_list[[i]] <- data.frame(
    domain = domain_to_ip$domain[i],
    ip = domain_to_ip$ip[i],
    country = geo$country,
    city = geo$city,
    org = geo$org,
    stringsAsFactors = FALSE
  )
}

results <- do.call(rbind, results_list)
results
```

                                                                       domain
    1 *\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00
    2                                                                  ISATAP
    3                                         safebrowsing.clients.google.com
    4                                                        tools.google.com
    5                                                           www.apple.com
                   ip       country       city
    1    110.209.6.25         China Jinrongjie
    2 169.254.255.255          <NA>       <NA>
    3    68.87.64.150 United States     Dallas
    4    68.87.64.150 United States     Dallas
    5    68.87.64.150 United States     Dallas
                                               org
    1 China TieTong Telecommunications Corporation
    2                                         <NA>
    3           Comcast Cable Communications, Inc.
    4           Comcast Cable Communications, Inc.
    5           Comcast Cable Communications, Inc.

## Вывод

В ходе работы были успешно получены знания и практические навыки
использования языка программирования R для обработки данных, закреплены
знания основных функций обработки данных экосистемы tidyverse языка R и
навыки исследования метаданных DNS трафика.
