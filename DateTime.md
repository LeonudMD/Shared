\# Полное руководство по работе с DateTime в .NET



В данном документе представлено максимально подробное описание работы с типом `DateTime` в .NET, включая внутреннее устройство, семантику, методы форматирования и парсинга, влияние культуры, взаимодействие с часовыми поясами, использование в различных сценариях, а также распространённые ошибки и лучшие практики.



---



\## 1. Введение



`DateTime` — фундаментальный тип в .NET, предназначенный для представления моментов времени с точностью до 100 наносекунд (1 тик).  

Преимущества:

\- Широкий диапазон: от 0001-01-01 до 9999-12-31.

\- Возможность выполнения арифметических операций (сложение, вычитание).

\- Поддержка различных способов отображения и парсинга.



---



\## 2. Внутреннее устройство



\- \*\*Счётчик тиков\*\*: 64-битное целое хранит количество тиков от эпохи (0001-01-01 00:00:00).

\- \*\*Тик\*\*: 100 наносекунд.

\- \*\*Память\*\*: один экземпляр хранит дату и время в одном поле, плюс флаги `Kind`.

\- \*\*Производительность\*\*: операции с `DateTime` выполняются быстро и эффективно, так как оперируют примитивным типом.



---



\## 3. Свойство DateTime.Kind



Семантика момента времени задаётся свойством:

```csharp

public DateTimeKind Kind { get; }

```

Возможные значения:

\- `Unspecified` — отсутствие привязки к часовому поясу.

\- `Utc` — время UTC.

\- `Local` — локальное время системы.



Поведение:

\- Методы `ToUniversalTime()` и `ToLocalTime()` ориентируются на `Kind`.

\- Формат `"O"` (round-trip) добавляет букву `Z` для UTC и смещение `±hh:mm` для Local.



---



\## 4. Форматирование даты и времени



\### 4.1. Стандартные строковые спецификаторы



| Спецификатор | Описание                  | Вывод (UTC 2025-07-09 14:30:45) |

|-------------:|---------------------------|---------------------------------|

| `d`          | Короткая дата             | `9.07.2025`                     |

| `D`          | Полная дата               | `Wednesday, July 09, 2025`      |

| `t`          | Короткое время            | `14:30`                         |

| `T`          | Полное время              | `14:30:45`                      |

| `g`          | Общий формат (короткий)   | `9.07.2025 14:30`               |

| `G`          | Общий формат (длинный)    | `9.07.2025 14:30:45`            |

| `o`/`O`      | Round-trip (ISO 8601)     | `2025-07-09T14:30:45.0000000Z`  |

| `r`/`R`      | RFC1123 (GMT)             | `Wed, 09 Jul 2025 14:30:45 GMT` |

| `s`          | Sortable (ISO-like)       | `2025-07-09T14:30:45`           |

| `u`          | Universal sortable        | `2025-07-09 14:30:45Z`          |



\### 4.2. Пользовательские форматы



Комбинация токенов позволяет задавать произвольный вид:



| Токен       | Описание                   | Пример |

|-------------|----------------------------|--------|

| `yyyy`      | Год, четыре цифры          | `2025` |

| `MM`        | Месяц, две цифры           | `07`   |

| `dd`        | День, две цифры            | `09`   |

| `HH`        | Часы 24-часового формата   | `14`   |

| `mm`        | Минуты                     | `30`   |

| `ss`        | Секунды                    | `45`   |

| `f…fffffff` | Доли секунды               | `1234567` |

| `tt`        | AM/PM                      | `PM`   |



Разделители:

\- `/` → символ разделителя даты (культура).

\- `:` → символ разделителя времени (культура).

\- Кавычки (`'...'`, `"..."`) → вставка литералов.



\*\*Пример пользовательского формата\*\*:  

`yyyy-MM-dd 'г.' HH:mm:ss`  

Результат: `2025-07-09 г. 14:30:45`



---



\## 5. Парсинг строк в DateTime



\- `DateTime.Parse` / `TryParse` — гибкий разбор с учётом культуры.

\- `DateTime.ParseExact` / `TryParseExact` — строгий разбор по заданным форматам.

\- `DateTimeStyles`:

&nbsp; - `AllowWhiteSpaces`

&nbsp; - `AssumeUniversal`

&nbsp; - `AdjustToUniversal`

&nbsp; - `RoundtripKind`



\*\*Пример\*\*:  

```csharp

DateTime.ParseExact(

&nbsp;   "09.07.2025 14:30:45",

&nbsp;   "dd.MM.yyyy HH:mm:ss",

&nbsp;   CultureInfo.InvariantCulture,

&nbsp;   DateTimeStyles.AssumeLocal

);

```



---



\## 6. Влияние культуры (CultureInfo)



\- `CultureInfo.CurrentCulture` — используется по умолчанию в приложении.

\- `CultureInfo.InvariantCulture` — нейтральная культура без региональных отличий.

\- От культуры зависит:

&nbsp; - Последовательность дня, месяца и года.

&nbsp; - Символы разделителей.

&nbsp; - Названия дней недели и месяцев.



---



\## 7. Работа с часовыми поясами



\- Метод `ToLocalTime()` конвертирует UTC во время системы.

\- Метод `ToUniversalTime()` конвертирует Local или Unspecified в UTC.

\- Класс `TimeZoneInfo` применяется для работы с произвольными зонами:

&nbsp; ```csharp

&nbsp; var tz = TimeZoneInfo.FindSystemTimeZoneById("Russian Standard Time");

&nbsp; var local = TimeZoneInfo.ConvertTimeFromUtc(utcDateTime, tz);

&nbsp; ```



---



\## 8. DateTimeOffset



\- Хранение даты и времени вместе со смещением.

\- Рекомендуется для передачи времени между зонами и в API.

\- Свойство `Offset` отражает разницу от UTC.



---



\## 9. TimeSpan



\- Отдельный тип для продолжительности.

\- Операции сложения и вычитания между `DateTime` и `TimeSpan`.

\- Форматы вывода: `c`, `g`, `G`, пользовательские.



---



\## 10. Сериализация в JSON



\- В `System.Text.Json` по умолчанию формат ISO-8601 (`O`) применяется к `DateTime`.

\- В `Newtonsoft.Json` используется `DateFormatHandling.IsoDateFormat` по умолчанию.

\- Кастомные конвертеры (`JsonConverter`) обеспечивают поддержку любых форматов.



---



\## 11. Использование в Entity Framework Core



\- Свойство `DateTime` по умолчанию мэппится в SQL тип `datetime2`.

\- При необходимости задаётся точность с помощью `.HasColumnType("datetime2(3)")`.

\- Для хранения UTC рекомендуется применять конвертацию значений перед сохранением.



---



\## 12. Потокобезопасность и копирование



\- `DateTime` является иммутабельным типом.

\- Передача по значению обеспечивает безопасность в многопоточной среде.



---



\## 13. Производительность и аллокации



\- Операции с `DateTime` не требуют выделения памяти в куче.

\- Метод `ToString` может возвращать новые строки, что учитывается при оптимизации.



---



\## 14. Распространённые ошибки



\- Оставление `Kind = Unspecified` при десериализации.

\- Игнорирование Daylight Saving Time (DST).

\- Неправильные преобразования между часовыми поясами.



---



\## 15. Лучшие практики



\- Хранение и обмен данными через UTC/ISO-8601.

\- Использование `DateTimeOffset` при необходимости смещений.

\- Явное указание культуры в парсинге/форматировании.

\- Применение строгого разбора (`ParseExact`) для критичных сценариев.



---



\## 16. Дополнительные ресурсы



\- Документация Microsoft по `DateTime`:  

&nbsp; https://docs.microsoft.com/dotnet/api/system.datetime  

\- Статья о `DateTimeOffset`:  

&nbsp; https://docs.microsoft.com/dotnet/api/system.datetimeoffset  

---



\##  Примеры кода

```csharp

using System;

using System.Globalization;

using System.Text.Json;

using System.Text.Json.Serialization;

using System.Text.RegularExpressions;



class Program

{

&nbsp;   static void Main()

&nbsp;   {

&nbsp;       const string json = @"{ ""Date1"": ""31.12.2024 23:59:59"" }";

&nbsp;       var options = new JsonSerializerOptions();

&nbsp;       options.Converters.Add(new CustomDateTimeConverter());



&nbsp;       // 1) Десериализация

&nbsp;       var model = JsonSerializer.Deserialize<MyModel>(json, options);



&nbsp;       // 2) Проверка Kind

&nbsp;       /\*if (model.Date1.Kind != DateTimeKind.Utc)

&nbsp;           throw new Exception("Ожидался Kind = Utc");\*/



&nbsp;       // 3) Формат строки через ToString("O")

&nbsp;       string isoString = model.Date1.ToString("O", CultureInfo.InvariantCulture);

&nbsp;       Console.WriteLine($"ISO-строка: {isoString}");



&nbsp;       // 4) Проверяем, что строка соответствует ISO-8601 c суффиксом Z

&nbsp;       //    простая RegEx-проверка:

&nbsp;       var isoPattern = @"^\\d{4}-\\d{2}-\\d{2}T\\d{2}:\\d{2}:\\d{2}\\.\\d{7}Z$";

&nbsp;       if (!Regex.IsMatch(isoString, isoPattern))

&nbsp;           throw new Exception("Строка не в формате ISO-8601 (round-trip)");



&nbsp;       Console.WriteLine("Проверка пройдена: и Kind == Utc, и формат ISO-8601.");

&nbsp;   }

}



public class CustomDateTimeConverter : JsonConverter<DateTime>

{

&nbsp;   private const string Format = "dd.MM.yyyy HH:mm:ss";



&nbsp;   public override DateTime Read(ref Utf8JsonReader reader, Type typeToConvert, JsonSerializerOptions options)

&nbsp;   {

&nbsp;       string str = reader.GetString();

&nbsp;       return DateTime.ParseExact(str, Format, CultureInfo.InvariantCulture);

&nbsp;   }



&nbsp;   public override void Write(Utf8JsonWriter writer, DateTime value, JsonSerializerOptions options)

&nbsp;   {

&nbsp;       // Записываем точно в том же формате, каким была строка в JSON

&nbsp;       writer.WriteStringValue(value.ToString(Format, CultureInfo.InvariantCulture));

&nbsp;   }

}



public class MyModel

{

&nbsp;   \[JsonConverter(typeof(CustomDateTimeConverter))]

&nbsp;   public DateTime Date1 { get; set; }

}



```



