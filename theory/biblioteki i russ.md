# Базовые библиотеки C++ и корректный вывод русских слов в консоль

---

# ЧАСТЬ 0. Коротко про `std::`

Раз тема namespace убрана, всё равно нужно понимать одно: **стандартная библиотека C++ физически лежит в пространстве имён `std`**, и убрать это нельзя. Есть два варианта записи:

**Вариант A. Писать `std::` перед каждым именем (используем в этом гайде):**
```cpp
std::cout << "Привет\n";
std::cin >> x;
std::string name;
```

**Вариант B. Точечно подключить нужные имена (тоже допустимо):**
```cpp
using std::cout;
using std::cin;
using std::string;

cout << "Привет\n";     // уже без std::
```

> **`using namespace std;` НЕ используем** — ни в глобальной области, ни внутри функций из-за конфликтов.

---

# ЧАСТЬ 1. РУССКИЙ ТЕКСТ В КОНСОЛИ — самое важное в начале

## 1.1. Что такое кодировка (на пальцах)

Компьютер хранит **числа**, а не буквы. Чтобы число стало буквой, нужна **таблица соответствия** — кодировка.

Проблема: таблиц много:
- **UTF-8** — современный стандарт, 1–4 байта на символ. Русская буква = 2 байта.
- **Windows-1251 (CP1251)** — старая русская кодировка Windows. 1 байт на букву.
- **CP866** — кодировка консоли Windows по умолчанию. 1 байт, но другая таблица.

Файл сохранён в UTF-8, а консоль ждёт CP866 → увидишь «кракозябры».


### Шаг 2. Добавь в `main` две строки

```cpp
#include <windows.h>

int main() {
    SetConsoleOutputCP(65001);   // вывод в UTF-8
    SetConsoleCP(65001);         // ввод в UTF-8
    // ...
}
```

`65001` — номер кодировки **UTF-8** в Windows.

### Шаг 3. Проверь консоль

Windows 10/11 — умеет UTF-8 из коробки. Windows 7 — могут быть проблемы (см. 1.5).

## 1.3. Полный рабочий пример

```cpp
#include <iostream>
#include <string>
#include <windows.h>       // для SetConsoleOutputCP и SetConsoleCP

int main() {
    SetConsoleOutputCP(65001);
    SetConsoleCP(65001);

    std::cout << "Привет, мир!\n";

    std::string name;
    std::cout << "Как вас зовут? ";
    std::getline(std::cin, name);

    if (name.empty()) {
        std::cout << "Вы ничего не ввели\n";
    } else {
        std::cout << "Привет, " << name << "!\n";
    }

    return 0;
}
```

Если всё правильно:
```
Привет, мир!
Как вас зовут? Иван
Привет, Иван!
```

## 1.4. Ввод числа + строки — классическая ловушка

После `std::cin >>` в буфере остаётся `\n`. Если сразу вызвать `getline`, он прочитает пустую строку. Лечение — `std::cin.ignore()`.

```cpp
#include <iostream>
#include <string>
#include <windows.h>

int main() {
    SetConsoleOutputCP(1251);
    SetConsoleCP(1251);

    int age;
    std::cout << "Сколько вам лет? ";
    std::cin >> age;

    if (std::cin.fail()) {
        std::cout << "Это не число!\n";
        return 1;
    }

    // ОЧЕНЬ ВАЖНО: выбрасываем остаток строки, включая \n
    std::cin.ignore(10000, '\n');

    std::string name;
    std::cout << "Как вас зовут? ";
    std::getline(std::cin, name);

    if (name.empty()) {
        std::cout << "Имя не введено\n";
    } else if (age < 18) {
        std::cout << name << ", вы несовершеннолетний\n";
    } else {
        std::cout << name << ", вы совершеннолетний\n";
    }

    return 0;
}
```

## 1.5. Если `SetConsoleOutputCP(65001)` не помог (старые Windows)

### План Б-1. Переключить консоль на 1251 (и файл сохранить в 1251)

```cpp
#include <iostream>
#include <windows.h>

int main() {
    SetConsoleOutputCP(1251);
    SetConsoleCP(1251);
    std::cout << "Привет\n";
    return 0;
}
```

Минус: файл должен быть в **Windows-1251**, это неудобно надолго. Быстрый обходной манёвр.

### План Б-2. `_setmode` (продвинутый)

```cpp
#include <iostream>
#include <fcntl.h>
#include <io.h>
#include <windows.h>

int main() {
    SetConsoleOutputCP(65001);
    SetConsoleCP(65001);
    _setmode(_fileno(stdout), _O_U8TEXT);
    // ... и тогда работает с std::wcout и L"..." литералами
    return 0;
}
```

Начни с 1.3 — в 90% случаев этого достаточно.


## 1.6. Про длину строки и индексацию

При UTF-8 **одна русская буква = 2 байта**. Поэтому:

```cpp
std::string s = "Иван";
std::cout << s.length() << "\n";   // 8 (байты, а не буквы!)
std::cout << s[0];                  // первый БАЙТ, а не буква — мусор на экране
```

**Правило:**
- `.length()` — в байтах.
- `[]` для русских букв **не используй**.
- Для «первой буквы» — `.substr(0, 2)` (первые 2 байта = первая русская буква, при 1251 1 байт).
- Сравнения `==`, `<`, `.find()`, `.substr()` — с UTF-8 работают корректно.

## 1.7. Итоговая памятка по русскому тексту

1. Файл `.cpp` — сохранить в UTF-8.
2. В начале `main`: `SetConsoleOutputCP(65001); SetConsoleCP(65001);`
3. После `cin >>` перед `getline`: `std::cin.ignore(10000, '\n');`
4. Не индексируй русские буквы через `[]`.
5. `length()` возвращает **байты**, не буквы.

---

# ЧАСТЬ 2. `<iostream>` — ввод/вывод

```cpp
#include <iostream>
#include <string>
#include <windows.h>

int main() {
    SetConsoleOutputCP(1251);
    SetConsoleCP(1251);

    // --- ВЫВОД ---
    std::cout << "Привет, мир!\n";
    std::cout << "Число: " << 42 << ", дробное: " << 3.14 << "\n";

    // Экранированные символы
    std::cout << "Первая строка\nВторая строка\n";
    std::cout << "Табуляция:\tрусский \tтекст\n";
    std::cout << "Кавычки: \"ёжик\"\n";
    std::cout << "Обратный слэш: \\\n";

    // --- ВВОД ЧИСЛА ---
    int age;
    std::cout << "Введите возраст: ";
    std::cin >> age;

    if (std::cin.fail()) {
        std::cin.clear();
        std::cin.ignore(10000, '\n');
        std::cerr << "Ошибка: это не число\n";
        return 1;
    }

    // --- ВВОД СТРОКИ С ПРОБЕЛАМИ ---
    std::cin.ignore(10000, '\n');   // ОЧЕНЬ ВАЖНО после cin >>

    std::string name;
    std::cout << "Введите имя и фамилию: ";
    std::getline(std::cin, name);

    if (name.empty()) {
        std::cout << "Пустая строка\n";
    } else {
        std::cout << "Здравствуйте, " << name << "! Возраст: " << age << "\n";
    }

    return 0;
}
```

---

# ЧАСТЬ 3. `<string>` — строки

```cpp
#include <iostream>
#include <string>
#include <windows.h>

int main() {
    SetConsoleOutputCP(1251);
    SetConsoleCP(1251);

    // Создание
    std::string greeting = "Привет";
    std::string name = "Иван";

    // Конкатенация
    std::string full = greeting + ", " + name + "!";
    std::cout << full << "\n";                    // Привет, Иван!

    // Длина (в байтах для UTF-8)
    std::cout << "Длина в байтах: " << full.length() << "\n";

    // Пустая ли?
    std::string empty;
    if (empty.empty()) {
        std::cout << "Строка пуста\n";
    }

    // Сравнение
    if (greeting == "Привет") {
        std::cout << "greeting равно Привет\n";
    }

    // Поиск подстроки
    std::string text = "Я люблю программировать на C++";
    int pos = text.find("пограммировать"); // -1 не нашел 0 и > значит нашел
    if (pos == -1) {
        std::cout << "Не найдено\n";
    }
    else {
        std::cout << "Найдено на позиции " << pos << "\n";
    }

    // Подстрока
    std::cout << full.substr(0, 12) <<"\n";      // Привет (12 байт = 6 букв)

    // Конкатенация через +=
    std::string s = "Привет";
    s += ", мир";
    s += "!";
    std::cout << s << "\n";                        // Привет, мир!

    // Число → строка
    std::string numStr = std::to_string(2025);
    std::cout << "Год: " + numStr << "\n";

    return 0;
}
```

## 3.1. Русские буквы через `substr`

```cpp
#include <iostream>
#include <string>
#include <windows.h>

int main() {
    SetConsoleOutputCP(65001);
    SetConsoleCP(65001);

    std::string s = "Привет";

    // НЕЛЬЗЯ: s[0] — это первый байт, не буква, для 1251 сработает
    // МОЖНО: substr с учётом 2 байта на русскую букву
    std::cout << s.substr(0, 2) << "\n";    // П
    std::cout << s.substr(2, 2) << "\n";    // р
    std::cout << s.substr(0, 6) << "\n";    // При (6 байт = 3 буквы)

    return 0;
}
```

---

# ЧАСТЬ 4. `<cmath>` — математика

```cpp
#include <iostream>
#include <cmath>
#include <windows.h>

int main() {
    SetConsoleOutputCP(1251);
    SetConsoleCP(1251);

    double x;
    std::cout << "Введите число: ";
    std::cin >> x;

    if (std::cin.fail()) {
        std::cout << "Это не число!\n";
        return 1;
    }

    std::cout << "Модуль:   " << std::abs(x)  << "\n";
    std::cout << "Квадрат:  " << std::pow(x, 2) << "\n";
    std::cout << "Вниз:     " << std::floor(x) << "\n";
    std::cout << "Вверх:    " << std::ceil(x)  << "\n";
    std::cout << "Ближайшее:" << std::round(x) << "\n";

    if (x >= 0) {
        std::cout << "Корень: " << std::sqrt(x) << "\n";
    } else {
        std::cout << "Корень из отрицательного числа не существует\n";
    }

    return 0;
}
```

---

# ЧАСТЬ 5. `<iomanip>` — форматирование

```cpp
#include <iostream>
#include <iomanip>
#include <windows.h>

int main() {
    SetConsoleOutputCP(1251);
    SetConsoleCP(1251);

    // Два знака после запятой
    std::cout << std::fixed << std::setprecision(2);
    std::cout << "Цена: " << 1234.5 << " руб.\n";   // Цена: 1234.50 руб.

    // Ширина поля
    std::cout << std::setw(10) << "товар" << "|\n";
    std::cout << std::setw(10) << std::left << "товар" << "|\n";

    return 0;
}
```

> **Замечание:** `setw` считает **байты**. Русское слово «товар» при UTF-8 займёт 10 байт. Для русского текста выравнивание через `setw` может работать неидеально — это нормально. Для 1251 будет работать нормально

---

# ЧАСТЬ 6. Полные примеры-программы

## Пример 1. Калькулятор с русским интерфейсом

```cpp
#include <iostream>
#include <windows.h>

int main() {
    SetConsoleOutputCP(1251);
    SetConsoleCP(1251);

    double a, b;
    char op;

    std::cout << "Введите первое число: ";
    std::cin >> a;
    if (std::cin.fail()) {
        std::cout << "Ошибка: это не число\n";
        return 1;
    }

    std::cout << "Введите операцию (+ - * /): ";
    std::cin >> op;

    std::cout << "Введите второе число: ";
    std::cin >> b;
    if (std::cin.fail()) {
        std::cout << "Ошибка: это не число\n";
        return 1;
    }

    switch (op) {
        case '+':
            std::cout << "Сумма: " << a + b << "\n";
            break;
        case '-':
            std::cout << "Разность: " << a - b << "\n";
            break;
        case '*':
            std::cout << "Произведение: " << a * b << "\n";
            break;
        case '/':
            if (b == 0) {
                std::cout << "Деление на ноль невозможно\n";
            } else {
                std::cout << "Частное: " << a / b << "\n";
            }
            break;
        default:
            std::cout << "Неизвестная операция\n";
    }

    return 0;
}
```

## Пример 2. Знакомство по-русски

```cpp
#include <iostream>
#include <string>
#include <windows.h>

int main() {
    SetConsoleOutputCP(1251);
    SetConsoleCP(1251);

    std::string name;
    int age;

    std::cout << "Как вас зовут? ";
    std::getline(std::cin, name);

    if (name.empty()) {
        std::cout << "Имя не введено, до свидания\n";
        return 1;
    }

    std::cout << "Сколько вам лет? ";
    std::cin >> age;
    if (std::cin.fail()) {
        std::cout << "Возраст введён некорректно\n";
        return 1;
    }

    std::cout << "\nЗдравствуйте, " << name << "!\n";

    if (age < 0) {
        std::cout << "Возраст не может быть отрицательным\n";
    } else if (age < 14) {
        std::cout << "Вы ребёнок\n";
    } else if (age < 18) {
        std::cout << "Вы подросток\n";
    } else if (age < 65) {
        std::cout << "Вы взрослый\n";
    } else {
        std::cout << "Вы пенсионер\n";
    }

    return 0;
}
```

## Пример 3. Сравнение двух русских строк

```cpp
#include <iostream>
#include <string>
#include <windows.h>

int main() {
    SetConsoleOutputCP(1251);
    SetConsoleCP(1251);

    std::string a, b;

    std::cout << "Первая строка: ";
    std::getline(std::cin, a);
    std::cout << "Вторая строка: ";
    std::getline(std::cin, b);

    if (a.length() > b.length()) {
        std::cout << "Первая строка длиннее\n";
    } else if (a.length() < b.length()) {
        std::cout << "Вторая строка длиннее\n";
    } else {
        std::cout << "Строки одинаковой длины\n";
    }

    if (a < b) {
        std::cout << "По алфавиту первая идёт раньше\n";
    } else if (a > b) {
        std::cout << "По алфавиту вторая идёт раньше\n";
    } else {
        std::cout << "Строки полностью равны\n";
    }

    return 0;
}
```


---

# ЧАСТЬ 8. Итоговая памятка

## 8.1. По `std::`

| Что | Как |
|-----|-----|
| Стандартные имена | `std::cout`, `std::cin`, `std::string` |
| Точечное подключение | `using std::cout;` (не путать с `using namespace std`) |
| `using namespace std;` | **НЕ используем** | 

## 8.2. По русскому тексту

| Правило | Зачем |
|---------|-------|
| Файл `.cpp` — UTF-8 | Компилятор прочитает буквы правильно |
| `SetConsoleOutputCP(65001); SetConsoleCP(65001);` в начале `main` | Консоль ждёт UTF-8 |
| `SetConsoleOutputCP(1251); SetConsoleCP(1251);` в начале `main` | Консоль ждёт 1251 |
| `#include <windows.h>` | Для этих функций |
| `std::cin.ignore(10000, '\n');` после `cin >>` перед `getline` | Иначе `getline` прочитает пустую строку |
| Не индексировать русские буквы через `[]` | Буква UTF-8 = 2 байта |
| индексировать русские буквы через `[]` | Буква 1251 = 1 байта |
| `length()` — это **байты**, не буквы | «Иван» = 4 байт |
| `length()` — это **байты**, не буквы | «Иван» = 8 байт utf|
| Использовать `.substr()`, `.find()`, сравнения | Работают с UTF-8 корректно |


---

# ЧАСТЬ 9. Частые ошибки и лечение

| Симптом | Причина | Лечение |
|---------|---------|---------|
| `╧ЁштхЄ` или `?????` | Файл не в UTF-8 или нет `SetConsoleOutputCP(65001)` | Сохранить в UTF-8 + добавить строки в `main` |
| `getline` прочитал пустую строку | После `cin >>` в буфере `\n` | `std::cin.ignore(10000, '\n');` перед `getline` |
| Русский ввод «съедается» | Консоль в другой кодировке | `SetConsoleCP(65001);` |
| `s[0]` выводит мусор для русского | Буква = 2 байта UTF-8 | Не индексируй, используй `substr` |
| `length()` вернул не то число | Считает байты, не буквы | Для «Иван» ожидай 8 |
| Программа падает при вводе не-числа | Не проверен `std::cin.fail()` | Всегда проверяй ввод числа через `if` |
