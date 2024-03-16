# Highload AppMetrica
## Содержание
* ### [1. Тема, целевая аудитория](#1)
* ### [2. Расчёт нагрузки](#2)
* ### [3. Глобальная балансировка нагрузки](#3)
* ### [ Список источников ](#sources)

## 1. Тема и целевая аудитория <a name="1"></a>

Яндекс AppMetrica — это бесплатный инструмент для трекинга и аналитики вашего приложения.

### 1.1 Целевая аудитория
![image](https://github.com/ambushidozho/Highload_AppMetrica/assets/102957421/16ba3e63-f397-47a5-be7a-d1f8e2e8a5e7)
| Страна            | Пользователи |
|:------------------|:-------------|
| Россия            | 37.4 %       |
| Китай             | 8.09 %       |
| США               | 6.67 %       |
| Сербия            | 6.07 %       |
| Украина           | 5.19 %       |
| Остальные         | 36.56 %      |

### 1.2 MVP
Ключевой функционал Appmetrica - это предоставление комплексной аналитики мобильных приложений.
1. Глубокая аналитика для продуктовых и growth-команд
2. Аналитика монетизации
3. Анализ аудитории
4. Работа с данными
5. Мониторинг стабильности
6. Пуш-уведомления

Основной технической сложностью сервиса является обработка данных в реальном времени, быстрое формирование отчётов, трекинг(атрибуция) установок

### 1.3 Аналоги
* [Google Analytics](https://marketingplatform.google.com/about/analytics/)
* [Adjust](https://www.adjust.com/)
* [Flurry](https://www.flurry.com/)
* [Amplitude](https://amplitude.com/)

## 2 Расчёт нагрузки <a name="2"></a>

### 2.1 Продуктовые метрики 

* DAU: 25 млн
* MAU: 33.2 млн
* Количество страниц за визит: 1.7 страницы
* Средняя продолжительность визита: 1 минута
* Приблизительно 2.1 млн вебсайтов пользуются AppMetrica (данные взяты исходя из количества сайтов для google analytics и сравнения его доли рынка с AppMetric`ой)
* AppMetrica установлена на более чем 250 млн устройств
* Более 12 млрд событий в сутки

Условимся, что у нас есть 2 типа пользователей - те, кто встраивают sdk в свои приложения и те, кто порождают события трекинга. 
Предположим, что при 2.1 млн подключенных вебсайтов и 250 млн устройств мы имеем 252.1 млн пользователей, которые нагружают сервис составлением отчетов. Остальные же пользователи лишь порождают события трекинга.
Допустим, что типичный рекламодатель заходит каждый 1-2 дня и проверяет отчёты по основным метрикам за определенный период(Revenue, User Acquisition, Аудитория, Вовлечённость, Retention).
 
Тогда получим следующие продуктовые метрики:

* 126 млн запросов на Revenue в день в среднем
* 126 млн запросов на User Acquisition в день в среднем
* 126 млн запросов на Аудитория в день в среднем
* 126 млн запросов на Вовлечённость в день в среднем
* 126 млн запросов на Retention в день в среднем
* Остальные 11.5 млрд событий в сутки уходят на трекинг пользователей

Допустим, что регистрацией новых рекламодателей и сервисов-партнеров в нашей платформе можно пренебречь. Также пренебрегаем авторизацией рекламодателей (0.14 RPS)
### 2.2 Технические метрики

#### 2.2.1 Расчет нагрузки на просмотр рекламного контента и получение аналитики

Суммарное количество запросов на составление отчетов
> 630 000 000 / (24 * 60 * 60) = 7292 RPS на чтение

Количество запросов от трекинга пользователей
> 11 500 000 000 / (24 * 60 * 60) = 133000 RPS на запись

#### 2.2.2 Расчет пиковой нагрузки на сетевой трафик

Суммарная пиковая нагрузка на составление отчетов
> 7292 * 500 Кб = 3.4 Гбит/с
> 630 000 000 * 500 Кб = 300000 Гбит/сутки

Количество пиковой нагрузки от трекинга пользователей
> 133000 RPS * 500 Кб = 63 Гбит/c
> 11 500 000 000 * 500 кб = 6144 Гбит/сутки

#### 2.2.3 Расчет объема занимаемого дискового пространства

Суммарное количество объема занимаемого дискового пространства на составление отчетов
Предположим что данные из которых составляются отчеты хранятся в ClickHouse и занимают незначительное количество места по сравнению с данными по трекингу

Количество объема дискового пространства от трекинга пользователей
> 11 500 000 000 * 500 кб = 6 Тб/сутки

Количество объема дискового пространства от побочной информации от трекинга пользователей
> 11 500 000 000 * 1 Мб = 12 Тб/сутки

| RPS                   | Нагрузка на сетевой трафик   |  Объем занимаемого дискового пространства  |
|:----------------------|:-----------------------------|:-------------------------------------------|
| 7292 RPS на чтение    | 66.4 Гбит/c                  | 18 Тб/сутки                                |
| 133000 RPS на запись  | 306144 Гбит/сутки            |                                            |

## 3 Глобальная балансировка нагрузки <a name="3"></a>

### 3.1 Расположение датацентров
В связи с тем, что большинство пользователей сервисом находится на территории России, Азии и Европе. Разумно было бы расположить датацентры
в этих регионах. 

Исходя из продуктовых метрик, можно примерно составить разбиение нагрузки по датацентрам

| Датацентр         | Нагрузка     |
|:------------------|:-------------|
| Россия            | ~52360 RPS   |
| Европа(Германия)  | ~56000 RPS   |
| Китай             | ~11200 RPS   |

### 3.2 Схема DNS балансировки

Будет использоваться Geo-based DNS для распределения нагрузки между датацентра в зависимости от местоположения пользователя. 

### 3.3 Схема Anycast балансировки

Кроме Geo-based DNS будем также применять технологию BGP anycast, позволяющую после получения IP-адреса влиять на выбор датацентра, на который будет сделан запрос.

## 4 Локальная балансировка нагрузки <a name="4"></a>

Для балансировки нагрузки внутри ДЦ и обеспечение автоматического горизонтального масштабирования будем использовать cloud-native систему оркестрации от google kubernetes.

### 4.1 Схема балансировки для входящих запросов
В каждом ДЦ будем расположен L7-балансировщик, предоствляемый облачным провайдером (ex. aws, gcs), который будет распределять нагрузку на ingress controller k8s по алгоритму round robin. На ингрессах будет происходить терминации ssl трафика, для уменьшия оверхеда на установления соединения в доверенной сети k8s. Ингресс контроллеры в соотвествии с указанными правилами распределяют нагрузки на сервисы k8s. Существует множество реализаций ингресс контроллера, выберем nginx ingress controller, тк он регулярно поддерживается и имеет большое сообщество.

### 4.2 Схема балансировки для межсервисных запросов
Балансировка нагрузки запросов между сервисами в kubernetes реализована с помощью абстракций kubernetes - сервисов, которые реализованы с помощью iptables, то есть является L7 балансировкой. Также в k8s предусмотрена балансировка на уровне dns - headless service, которые разумно применять для микросервисов. По итогу получаем балансировку на уровне dns и на прикладном уровне.

### 4.3 Схема отказоустойчивости
За отказоустойчивость будут отвечать Kubernetes и Nginx.
Для обеспечения отказоустойчивости и горизонтального масштабирования будем использовать Horizontal Pod Autoscaling - абстракцию k8s, позволяющую изменить количество pod\`ов в сете в зависимости от значений настроенных метрик.
Kubernetes удаляет зависшие приложения, которые перестают отвечать на healthcheck`и. Затем запускает новые, убеждается в том, что они корректно запустились, и после этого дает на них нагрузку.

## Список источников: <a name="sources"></a>
* https://habr.com/ru/articles/322724/
* https://habr.com/ru/articles/509540/
* https://appmetrica.yandex.ru/about
* https://www.youtube.com/watch?v=iTFsfc9hYjA&ab_channel=HighLoadChannel
* https://www.similarweb.com/ru/
