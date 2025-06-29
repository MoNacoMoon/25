using System;
using System.Collections.Generic;
using System.Globalization;
using System.IO;
using System.Linq;
using System.Text;

// --- Структуры и классы данных из всех заданий ---

/// <summary>
/// Перечисление для типов событий (из Задания 3).
/// </summary>
public enum EventType
{
    Info,
    Warning,
    Error
}

/// <summary>
/// Структура для представления события в системе (из Задания 3).
/// </summary>
public struct Event
{
    public DateTime EventDateTime { get; set; }
    public EventType Type { get; set; }
    public string Message { get; set; }

    public override string ToString()
    {
        string date = EventDateTime.ToString("ddMMyyyy");
        string time = EventDateTime.ToString("HH:mm:ss");
        return $"[{date} {time}] [{Type}] - {Message}";
    }
}

/// <summary>
/// Класс для события Windows (из Задания 2).
/// </summary>
public class WindowsEvent
{
    public string ApplicationName { get; set; }
    public string EventLevel { get; set; } // e.g., "Ошибка", "Предупреждение", "Информация"
    public int EventCode { get; set; }
    public DateTime EventDateTime { get; set; }

    public int GetAgeInDays()
    {
        return (int)Math.Floor((DateTime.Now - EventDateTime).TotalDays);
    }

    public void DisplayEventInfo()
    {
        Console.WriteLine($"  Приложение: {ApplicationName}");
        Console.WriteLine($"  Уровень: {EventLevel}");
        Console.WriteLine($"  Код события: {EventCode}");
        Console.WriteLine($"  Дата и время: {EventDateTime:dd.MM.yyyy HH:mm:ss}");
    }
}

/// <summary>
/// Основной класс программы, объединяющий все задания.
/// </summary>
public class Program
{
    // Хранилище для Задания 2 (в памяти)
    private static List<WindowsEvent> eventLog = new List<WindowsEvent>();
    private static int eventCode;

    public static void Main(string[] args)
    {
        // Устанавливаем кодировку для корректного отображения кириллицы в консоли
        Console.OutputEncoding = Encoding.UTF8;

        while (true)
        {
            Console.WriteLine("\n--- ГЛАВНОЕ МЕНЮ ---");
            Console.WriteLine("1. Расчет параметров телепередачи (Задание 1)");
            Console.WriteLine("2. Журнал событий Windows [в памяти] (Задание 2)");
            Console.WriteLine("3. Обработка событий из файла (Задание 3)");
            Console.WriteLine("4. Выход");
            Console.Write("Выберите опцию: ");

            string choice = Console.ReadLine();

            switch (choice)
            {
                case "1":
                    RunTask1_TvCalculator();
                    break;
                case "2":
                    RunTask2_WindowsEventLog();
                    break;
                case "3":
                    RunTask3_FileEventLogger();
                    break;
                case "4":
                    Console.WriteLine("Выход из программы.");
                    return;
                default:
                    Console.WriteLine("Неверный выбор. Пожалуйста, попробуйте снова.");
                    break;
            }
            Console.WriteLine("\nНажмите любую клавишу для возврата в главное меню...");
            Console.ReadKey();
        }
    }

    #region --- ЗАДАНИЕ 1: Расчет параметров телепередачи ---
    private static void RunTask1_TvCalculator()
    {
        Console.WriteLine("\n--- Задание 1: Расчет параметров телепередачи ---");

        // Вывод текущей даты и времени
        Console.WriteLine($"Текущая системная дата: {DateTime.Now:dd.MM.yyyy}");
        Console.WriteLine($"Текущее системное время: {DateTime.Now:HH:mm:ss}");
        Console.WriteLine();

        DateTime startTime;
        int durationSeconds;

        // Ввод времени начала телепередачи
        while (true)
        {
            Console.Write("Введите время начала телепередачи (ЧЧ:ММ:СС): ");
            if (DateTime.TryParseExact(Console.ReadLine(), "HH:mm:ss", CultureInfo.InvariantCulture, DateTimeStyles.None, out startTime))
            {
                break;
            }
            Console.WriteLine("Неверный формат времени. Пожалуйста, используйте ЧЧ:ММ:СС.");
        }

        // Ввод продолжительности телепередачи в секундах
        while (true)
        {
            Console.Write("Введите продолжительность телепередачи в секундах: ");
            if (int.TryParse(Console.ReadLine(), out durationSeconds) && durationSeconds >= 0)
            {
                break;
            }
            Console.WriteLine("Неверный формат продолжительности. Введите целое положительное число.");
        }

        DateTime endTime = startTime.AddSeconds(durationSeconds);
        TimeSpan duration = TimeSpan.FromSeconds(durationSeconds);
        double durationMinutes = duration.TotalMinutes;
        int commercialBreakIntervalMinutes = 5;
        int numberOfCommercialBreaks = (int)(duration.TotalMinutes / commercialBreakIntervalMinutes);

        Console.WriteLine("\n--- Результаты расчета ---");
        Console.WriteLine($"Время начала телепередачи: {startTime:HH:mm:ss}");
        Console.WriteLine($"Продолжительность телепередачи (секунды): {durationSeconds}");
        Console.WriteLine($"Время окончания телепередачи: {endTime:HH:mm:ss}");
        Console.WriteLine($"Продолжительность телепередачи (минуты): {durationMinutes:F2}");
        Console.WriteLine($"Количество рекламных пауз: {numberOfCommercialBreaks}");
    }
    #endregion

    #region --- ЗАДАНИЕ 2: Журнал событий Windows (в памяти) ---
    private static void RunTask2_WindowsEventLog()
    {
        while (true)
        {
            Console.WriteLine("\n--- Меню Журнала Событий Windows ---");
            Console.WriteLine("1. Ввести новое событие");
            Console.WriteLine("2. Вывести информацию по всем событиям с указанием давности");
            Console.WriteLine("3. Вывести сведения об ошибках за определенную дату");
            Console.WriteLine("4. Вернуться в главное меню");
            Console.Write("Выберите опцию: ");

            string choice = Console.ReadLine();

            switch (choice)
            {
                case "1":
                    InputNewEvent();
                    break;
                case "2":
                    DisplayAllEventsWithAge();
                    break;
                case "3":
                    DisplayErrorsByDate();
                    break;
                case "4":
                    return; // Возврат в главное меню
                default:
                    Console.WriteLine("Неверный выбор. Пожалуйста, попробуйте снова.");
                    break;
            }
        }
    }

    private static void InputNewEvent()
    {
        Console.WriteLine("\n--- Ввод нового события ---");
        var newEvent = new WindowsEvent();

        Console.Write("Введите название приложения: ");
        newEvent.ApplicationName = Console.ReadLine();

        Console.Write("Введите уровень события (Ошибка, Предупреждение, Информация): ");
        newEvent.EventLevel = Console.ReadLine();

        Console.Write("Введите код события: ");
        while (!int.TryParse(Console.ReadLine(), out int eventCode))
        {
            Console.Write("Неверный ввод. Пожалуйста, введите числовой код события: ");
        }
        newEvent.EventCode = eventCode;

        DateTime eventDate = GetDateInput("Введите дату события (ДД.ММ.ГГГГ): ");
        DateTime eventTime = GetTimeInput("Введите время события (ЧЧ:ММ:СС): ");
        newEvent.EventDateTime = eventDate.Add(eventTime.TimeOfDay);

        eventLog.Add(newEvent);
        Console.WriteLine("Событие успешно добавлено.");
    }

    private static void DisplayAllEventsWithAge()
    {
        Console.WriteLine("\n--- Информация по всем событиям ---");
        if (eventLog.Count == 0)
        {
            Console.WriteLine("Журнал событий пуст.");
            return;
        }

        foreach (var ev in eventLog)
        {
            Console.WriteLine("-----------------------------------");
            ev.DisplayEventInfo();
            Console.WriteLine($"  Давность события: {ev.GetAgeInDays()} дней.");
        }
        Console.WriteLine("-----------------------------------");
    }

    private static void DisplayErrorsByDate()
    {
        Console.WriteLine("\n--- Ошибки за определенную дату ---");
        if (eventLog.Count == 0)
        {
            Console.WriteLine("Журнал событий пуст.");
            return;
        }

        DateTime targetDate = GetDateInput("Введите дату для поиска ошибок (ДД.ММ.ГГГГ): ");
        bool foundErrors = false;

        var errorsOnDate = eventLog.Where(ev =>
            ev.EventDateTime.Date == targetDate.Date &&
            ev.EventLevel.Equals("Ошибка", StringComparison.OrdinalIgnoreCase)
        ).ToList();

        if (errorsOnDate.Any())
        {
            Console.WriteLine($"\nОшибки, произошедшие {targetDate:dd.MM.yyyy}:");
            foundErrors = true;
            foreach (var ev in errorsOnDate)
            {
                Console.WriteLine("-----------------------------------");
                ev.DisplayEventInfo();
            }
        }

        if (!foundErrors)
        {
            Console.WriteLine($"Ошибок за {targetDate:dd.MM.yyyy} не найдено.");
        }
        Console.WriteLine("-----------------------------------");
    }

    // Вспомогательные методы для Задания 2
    private static DateTime GetDateInput(string prompt)
    {
        DateTime date;
        while (true)
        {
            Console.Write(prompt);
            if (DateTime.TryParseExact(Console.ReadLine(), "dd.MM.yyyy", CultureInfo.InvariantCulture, DateTimeStyles.None, out date))
            {
                return date;
            }
            Console.WriteLine("Неверный формат даты. Используйте ДД.ММ.ГГГГ.");
        }
    }

    private static DateTime GetTimeInput(string prompt)
    {
        DateTime time;
        while (true)
        {
            Console.Write(prompt);
            if (DateTime.TryParseExact(Console.ReadLine(), "HH:mm:ss", CultureInfo.InvariantCulture, DateTimeStyles.None, out time))
            {
                return time;
            }
            Console.WriteLine("Неверный формат времени. Используйте ЧЧ:ММ:СС.");
        }
    }
    #endregion

    #region --- ЗАДАНИЕ 3: Обработка событий из файла ---
    private const string DataFileName = "events.dat";

    private static void RunTask3_FileEventLogger()
    {
        Console.WriteLine("\n--- Задание 3: Обработка событий из файла ---");

        // 1. Создаем тестовый набор данных
        var events = CreateSampleEvents();
        Console.WriteLine("\n--- 1. Создан тестовый набор данных ---");
        events.ForEach(e => Console.WriteLine(e));

        // 2. Записываем структуру в файл
        WriteEventsToFile(DataFileName, events);
        Console.WriteLine($"\n--- 2. Данные успешно записаны в файл '{DataFileName}' ---");

        // 3. Читаем данные из файла
        var eventsFromFile = ReadEventsFromFile(DataFileName);
        Console.WriteLine($"\n--- 3. Данные успешно прочитаны из файла '{DataFileName}' ---");

        // 4. Выполняем задания по анализу
        Console.WriteLine("\n--- 4. Выполнение заданий по анализу ---");
        Console.WriteLine("\n--- ЗАДАНИЕ 3.1: Предупреждения за сегодня ---");
        CountAndDisplayTodayWarnings(eventsFromFile);

        Console.WriteLine("\n--- ЗАДАНИЕ 3.2: Последняя ошибка за прошлый месяц ---");
        FindLastErrorOfPreviousMonth(eventsFromFile);

        Console.WriteLine("\n--- ЗАДАНИЕ 3.3: Упорядочить информацию по дате в разные файлы ---");
        SortEventsIntoDateFiles(eventsFromFile);
    }

    private static List<Event> CreateSampleEvents()
    {
        DateTime now = DateTime.Now;
        DateTime previousMonth = now.AddMonths(-1);

        return new List<Event>
        {
            // События за сегодня
            new Event { EventDateTime = now.Date.AddHours(9).AddMinutes(15), Type = EventType.Info, Message = "Система запущена." },
            new Event { EventDateTime = now.Date.AddHours(10).AddMinutes(5), Type = EventType.Warning, Message = "Мало места на диске C:" },
            new Event { EventDateTime = now.Date.AddHours(12).AddMinutes(30), Type = EventType.Warning, Message = "Превышен лимит CPU." },
            // События за вчера
            new Event { EventDateTime = now.AddDays(-1).Date.AddHours(20), Type = EventType.Error, Message = "Не удалось подключиться к базе данных." },
            // События за прошлый месяц
            new Event { EventDateTime = new DateTime(previousMonth.Year, previousMonth.Month, 10, 14, 0, 0), Type = EventType.Error, Message = "Ошибка авторизации пользователя 'admin'." },
            new Event { EventDateTime = new DateTime(previousMonth.Year, previousMonth.Month, 15, 18, 22, 0), Type = EventType.Info, Message = "Обновление системы завершено." },
            new Event { EventDateTime = new DateTime(previousMonth.Year, previousMonth.Month, 28, 23, 50, 0), Type = EventType.Error, Message = "Критическая ошибка: сервис 'X' остановлен." }, // Это последняя ошибка
            // События за позапрошлый месяц
             new Event { EventDateTime = now.AddMonths(-2).Date.AddHours(11), Type = EventType.Info, Message = "Архивация данных." },
        };
    }

    private static void WriteEventsToFile(string filePath, List<Event> events)
    {
        using (var writer = new BinaryWriter(File.Open(filePath, FileMode.Create)))
        {
            writer.Write(events.Count);
            foreach (var ev in events)
            {
                writer.Write(ev.EventDateTime.ToBinary());
                writer.Write((int)ev.Type);
                writer.Write(ev.Message);
            }
        }
    }

    private static List<Event> ReadEventsFromFile(string filePath)
    {
        var events = new List<Event>();
        if (!File.Exists(filePath)) return events;

        using (var reader = new BinaryReader(File.Open(filePath, FileMode.Open)))
        {
            int count = reader.ReadInt32();
            for (int i = 0; i < count; i++)
            {
                events.Add(new Event
                {
                    EventDateTime = DateTime.FromBinary(reader.ReadInt64()),
                    Type = (EventType)reader.ReadInt32(),
                    Message = reader.ReadString()
                });
            }
        }
        return events;
    }

    private static void CountAndDisplayTodayWarnings(List<Event> allEvents)
    {
        DateTime today = DateTime.Today;
        var todayWarnings = allEvents.Where(e => e.Type == EventType.Warning && e.EventDateTime.Date == today).ToList();

        Console.WriteLine($"Количество предупреждений за сегодня ({today:dd.MM.yyyy}): {todayWarnings.Count}");

        if (todayWarnings.Any())
        {
            Console.WriteLine("Сведения о них:");
            foreach (var warning in todayWarnings)
            {
                Console.WriteLine($"  - Время: {warning.EventDateTime:HH:mm:ss}, Сообщение: {warning.Message}");
            }
        }
    }

    private static void FindLastErrorOfPreviousMonth(List<Event> allEvents)
    {
        DateTime now = DateTime.Now;
        DateTime firstDayOfCurrentMonth = new DateTime(now.Year, now.Month, 1);
        DateTime firstDayOfPreviousMonth = firstDayOfCurrentMonth.AddMonths(-1);

        var lastError = allEvents
            .Where(e => e.Type == EventType.Error &&
                        e.EventDateTime >= firstDayOfPreviousMonth &&
                        e.EventDateTime < firstDayOfCurrentMonth)
            .OrderByDescending(e => e.EventDateTime)
            .FirstOrDefault();

        if (lastError.Message != null)
        {
            Console.WriteLine("Найдено последнее сообщение об ошибке за прошлый месяц:");
            Console.WriteLine(lastError.ToString());
        }
        else
        {
            Console.WriteLine("Сообщения об ошибках за прошлый месяц не найдены.");
        }
    }

    private static void SortEventsIntoDateFiles(List<Event> allEvents)
    {
        if (!allEvents.Any())
        {
            Console.WriteLine("Нет событий для сортировки.");
            return;
        }

        var groupedByDate = allEvents.GroupBy(e => e.EventDateTime.Date);

        Console.WriteLine("Создание файлов по датам...");
        foreach (var group in groupedByDate)
        {
            string fileName = $"{group.Key:ddMMyyyy}.txt";
            var fileContent = new StringBuilder();
            fileContent.AppendLine($"События за {group.Key:dd.MM.yyyy}:");

            foreach (var ev in group.OrderBy(e => e.EventDateTime))
            {
                fileContent.AppendLine($"  Время: {ev.EventDateTime:HH:mm:ss}, Тип: {ev.Type}, Сообщение: {ev.Message}");
            }

            File.WriteAllText(fileName, fileContent.ToString());
            Console.WriteLine($"  - Создан файл: {fileName}");
        }
        Console.WriteLine("Сортировка завершена.");
    }
    #endregion
}
