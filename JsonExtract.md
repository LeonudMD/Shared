# Руководство по извлечению полей из JSON в .NET

JSON (JavaScript Object Notation) служит универсальным форматом для обмена данными. При работе с JSON нередко требуется получить значение по заданному ключу в древовидной структуре, где узлы могут быть объектами, массивами или примитивными значениями (строки, числа, булевы значения, `null`).

Приводятся три подхода к извлечению значения по ключу, начиная с наиболее простого, когда доступ возможен только к верхнему уровню, и заканчивая универсальным методом рекурсивного обхода.

## Пример 1. Прямое извлечение строкового поля верхнего уровня

В случае плоской структуры JSON, где ключ гарантированно присутствует в корневом объекте, извлечение выполняется с помощью проверки наличия свойства и чтения его значения:

```csharp
using System.Text.Json;

class Program
{
    static void Main()
    {
        string json = @"{ ""name"": ""Alice"", ""age"": 30 }";
        using var doc = JsonDocument.Parse(json);
        JsonElement root = doc.RootElement;

        if (root.TryGetProperty("name", out JsonElement nameElement))
        {
            string name = nameElement.GetString();
            Console.WriteLine(name);
        }
        else
        {
            Console.WriteLine("Ключ name отсутствует");
        }
    }
}
```

Метод `TryGetProperty` обеспечивает безопасную проверку наличия свойства, а `GetString()` возвращает значение в виде строки.

## Пример 2. Последовательная проверка вложенных свойств

При глубже вложенной структуре выполняется последовательная проверка каждого уровня:

```csharp
using System.Text.Json;

class Program
{
    static void Main()
    {
        string json = @"
        {
          ""user"": {
            ""profile"": {
              ""email"": ""alice@example.com""
            }
          }
        }";
        using var doc = JsonDocument.Parse(json);
        JsonElement root = doc.RootElement;

        if (root.TryGetProperty("user", out JsonElement user) &&
            user.TryGetProperty("profile", out JsonElement profile) &&
            profile.TryGetProperty("email", out JsonElement emailEl))
        {
            Console.WriteLine(emailEl.GetString());
        }
        else
        {
            Console.WriteLine("Email не найден");
        }
    }
}
```

Подход пригоден, когда структура известна заранее и изменения в ней маловероятны.

## Пример 3. Универсальная функция рекурсивного поиска

Для извлечения по ключу без знания точного расположения применяется рекурсивный обход объектов и массивов до первого совпадения:

```csharp
using System.Text.Json;

static JsonElement? FindInJson(JsonElement element, string key)
{
    switch (element.ValueKind)
    {
        case JsonValueKind.Object:
            foreach (var prop in element.EnumerateObject())
            {
                if (prop.NameEquals(key))
                    return prop.Value;
                var sub = FindInJson(prop.Value, key);
                if (sub.HasValue)
                    return sub;
            }
            break;
        case JsonValueKind.Array:
            foreach (var item in element.EnumerateArray())
            {
                var sub = FindInJson(item, key);
                if (sub.HasValue)
                    return sub;
            }
            break;
    }
    return null;
}

// Использование:
using var doc = JsonDocument.Parse(jsonInput);
var result = FindInJson(doc.RootElement, key);
if (result.HasValue)
    Console.WriteLine(result.Value);
else
    Console.WriteLine("Не найдено");
```

Функция обходит как объекты, так и массивы, возвращая значение при первом обнаружении совпадения. Для сбора всех совпадений можно изменять алгоритм так, чтобы накопить результаты в списке.

### Практические замечания

- При глубокой вложенности возможен риск переполнения стека набором рекурсивных вызовов.
- Для нестрогих требований к скорости предпочтительны способы с прямым доступом.
- Альтернативный подход — использование итеративного обхода через собственную реализацию стека.
- Для типизированного извлечения может применяться десериализация в каскадно вложенные классы.

---

Рекомендованные решения зависят от структуры JSON и требований к производительности. Каждый вариант сочетает уровень сложности и гибкость, позволяя выбирать оптимальный способ извлечения полей.
