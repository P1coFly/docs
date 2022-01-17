---
sourcePath: core/yql/reference/yql-docs-core-2/udf/list/_includes/url.md
sourcePath: yql/reference/yql-docs-core-2/udf/list/_includes/url.md
---

# Url

## Normalize {#normalize}

* ```Url::Normalize(String) -> String?```

Нормализует URL удобным для Web-роботов образом: приводит hostname в нижний регистр, выкидывает фрагмент и т.п.
Результат нормализации зависит только от самого URL. В процессе нормализации **НЕ** выполняются операции, зависящие от внешних данных: приведение по дублям, зеркалам и т.п.

Возвращаемое значение:
* нормализованный URL;
* `NULL`, если переданный строковый аргумент не удалось распарсить как URL.

**Примеры**

```sql
SELECT Url::Normalize("hTTp://wWw.yAnDeX.RU/"); -- "http://www.yandex.ru/"
SELECT Url::Normalize("http://ya.ru#foo");      -- "http://ya.ru/"
```

## NormalizeWithDefaultHttpScheme {#normalizewithdefaulthttpscheme}

* ```Url::NormalizeWithDefaultHttpScheme(String?) -> String?```

Выполняет нормализацию аналогично `Url::Normalize`, но подставляет схему `http://` в случае, если схемы нет.

Возвращаемое значение:

* нормализованный URL;
* исходный URL, если нормализация не удалась.

**Примеры**

```sql
SELECT Url::NormalizeWithDefaultHttpScheme("wWw.yAnDeX.RU");    -- "http://www.yandex.ru/"
SELECT Url::NormalizeWithDefaultHttpScheme("http://ya.ru#foo"); -- "http://ya.ru/"
```

## Encode / Decode {#encode}

Кодируют UTF-8 строку в urlencoded формат (`Url::Encode`) и обратно (`Url::Decode`).

**Список функций**

* ```Url::Encode(String?) -> String?```
* ```Url::Decode(String?) -> String?```

**Примеры**

```sql
SELECT Url::Decode("http://ya.ru/%D1%81%D1%82%D1%80%D0%B0%D0%BD%D0%B8%D1%86%D0%B0"); 
  -- "http://ya.ru/страница"
SELECT Url::Encode("http://ya.ru/страница");                                         
  -- "http://ya.ru/%D1%81%D1%82%D1%80%D0%B0%D0%BD%D0%B8%D1%86%D0%B0"
```

## Parse {#parse}

Разбирает URL на составные части.

* ```Url::Parse(Parse{Flags:AutoMap}) -> Struct< Frag: String?, Host: String?, ParseError: String?, Pass: String?, Path: String?, Port: String?, Query: String?, Scheme: String?, User: String? >```

**Примеры**

```sql
SELECT Url::Parse(
  "https://en.wikipedia.org/wiki/Isambard_Kingdom_Brunel?s=24&g=h-24#Great_Western_Railway");
/*
(
  "Frag": "Great_Western_Railway",
  "Host": "en.wikipedia.org",
  "ParseError": null,
  "Pass": null,
  "Path": "/wiki/Isambard_Kingdom_Brunel",
  "Port": null,
  "Query": "s=24&g=h-24",
  "Scheme": "https",
  "User": null
)
*/
```

## Get... {#get}

Получение компонента URL.

**Список функций**

* ```Url::GetScheme(String{Flags:AutoMap}) -> String```
* ```Url::GetHost(String?) -> String?```
* ```Url::GetHostPort(String?) -> String?```
* ```Url::GetSchemeHost(String?) -> String?```
* ```Url::GetSchemeHostPort(String?) -> String?```
* ```Url::GetPort(String?) -> String?```
* ```Url::GetTail(String?) -> String?``` -- всё после хоста: path + query + fragment
* ```Url::GetPath(String?) -> String?```
* ```Url::GetFragment(String?) -> String?```
* ```Url::GetCGIParam(String?, String) -> String?``` -- второй параметр — имя нужного CGI параметра
* ```Url::GetDomain(String?, Uint8) -> String?``` -- второй параметр — необходимый уровень домена
* ```Url::GetTLD(String{Flags:AutoMap}) -> String```
* ```Url::IsKnownTLD(String{Flags:AutoMap}) -> Bool``` -- зарегистрирован на http://www.iana.org/
* ```Url::IsWellKnownTLD(String{Flags:AutoMap}) -> Bool``` -- находится в небольшом whitelist из com, net, org, ru и пр.
* ```Url::GetDomainLevel(String{Flags:AutoMap}) -> Uint64```
* ```Url::GetSignificantDomain(String{Flags:AutoMap}, [List<String>?]) -> String```
  Возвращает домен второго уровня в большинстве случаев и домен третьего уровня для хостеймов вида: ***.XXX.YY, где XXX — одно из com, net, org, co, gov, edu. Этот список можно переопределить через опциональный второй аргумент

* ```Url::GetOwner(String{Flags:AutoMap}) -> String```
  Возвращает домен, которым с наибольшей вероятностью владеет отдельный человек или организация. В отличие от Url::GetSignificantDomain работает по специальному whitelist, и помимо доменов из серии ***.co.uk возвращает домен третьего уровня для, например, бесплатных хостингов и блогов, скажем something.livejournal.com

**Примеры**

```sql
SELECT Url::GetScheme("https://ya.ru");           -- "https://"
SELECT Url::GetDomain("http://www.yandex.ru", 2); -- "yandex.ru"
```

## Cut... {#cut}

* ```Url::CutScheme(String?) -> String?```
  Возвращает переданный URL уже без схемы (http://, https:// и т.п.).

* ```Url::CutWWW(String?) -> String?```
  Возвращает переданный домен без префикса "www.", если он имелся.

* ```Url::CutWWW2(String?) -> String?```
Возвращает переданный домен без префикса "www.", "www2.", "wwww777." и тому подобных, если он имелся.

* ```Url::CutQueryStringA­ndFragment(String{Flags:AutoMap}) -> String```
  Возращает копию переданного URL с удаленными всеми CGI параметрами и фрагментами ("?foo=bar" и/или "#baz").

**Примеры**

```sql
SELECT Url::CutScheme("http://www.yandex.ru"); -- "www.yandex.ru"
SELECT Url::CutWWW("www.yandex.ru");           -- "yandex.ru"
```

## ...Punycode... {#punycode}

Преобразования [Punycode](https://en.wikipedia.org/wiki/Punycode).

**Список функций**

* ```Url::HostNameToPunycode(String{Flag:AutoMap}) -> String?```
* ```Url::ForceHostNameToPunycode(String{Flag:AutoMap}) -> String```
* ```Url::PunycodeToHostName(String{Flag:AutoMap}) -> String?```
* ```Url::ForcePunycodeToHostName(String{Flag:AutoMap}) -> String```
* ```Url::CanBePunycodeHostName(String{Flag:AutoMap}) -> Bool```

**Примеры**

```sql
SELECT Url::PunycodeToHostName("xn--d1acpjx3f.xn--p1ai"); -- "яндекс.рф"
```