
### Подготовка: настройка кодировки консоли 1251

Чтобы русский текст в консоли отображался корректно, необходимо в начале `main` вызвать функции Windows API. **Обратите внимание**: сам файл `.cpp` также следует сохранить в кодировке ANSI (Windows-1251), иначе русские символы в строковых литералах будут повреждены.

```cpp
#include <windows.h> // Для SetConsoleCP и SetConsoleOutputCP

// В начале main:
SetConsoleCP(1251);
SetConsoleOutputCP(1251);
```

---

### Часть первая: библиотека `<cmath>`

`<cmath>` — это заголовочный файл стандартной библиотеки C++, предоставляющий общие математические функции. Большинство функций принимают `double` и возвращают `double`, но также поддерживают перегрузки для `float` и `long double`.

#### 1. Степени и корни

```cpp
#include <iostream>
#include <cmath>      // Обязательно для математических функций
#include <windows.h>

int main() {
    SetConsoleCP(1251);
    SetConsoleOutputCP(1251);

    double base = 2.0;
    double exp_val = 10.0;

    // pow(x, y): вычисляет x в степени y
    // Внимание: возвращает double, даже если аргументы целые
    double result_pow = std::pow(base, exp_val);
    std::cout << "2 в степени 10 = " << result_pow << std::endl; // 1024

    // sqrt(x): квадратный корень
    // Если x < 0, возвращает NaN (Not a Number)
    double val_sqrt = 144.0;
    std::cout << "Корень из 144 = " << std::sqrt(val_sqrt) << std::endl; // 12

    // cbrt(x): кубический корень (C++11)
    // Работает и с отрицательными числами (в отличие от pow с дробным показателем)
    double val_cbrt = -27.0;
    std::cout << "Кубический корень из -27 = " << std::cbrt(val_cbrt) << std::endl; // -3

    // hypot(x, y): длина гипотенузы по двум катетам (sqrt(x*x + y*y))
    // Устойчива к переполнению, в отличие от ручного вычисления
    double a = 3.0, b = 4.0;
    std::cout << "Гипотенуза (3,4) = " << std::hypot(a, b) << std::endl; // 5

    return 0;
}
```

#### 2. Округление и остаток

```cpp
#include <iostream>
#include <cmath>
#include <windows.h>

int main() {
    SetConsoleCP(1251);
    SetConsoleOutputCP(1251);

    double x = 3.7;
    double y = -3.7;

    // ceil(x): округление вверх (наименьшее целое, не меньшее x)
    std::cout << "ceil(3.7)  = " << std::ceil(x) << std::endl;  // 4
    std::cout << "ceil(-3.7) = " << std::ceil(y) << std::endl; // -3

    // floor(x): округление вниз (наибольшее целое, не большее x)
    std::cout << "floor(3.7)  = " << std::floor(x) << std::endl;  // 3
    std::cout << "floor(-3.7) = " << std::floor(y) << std::endl; // -4

    // round(x): округление до ближайшего целого (банковское округление)
    // .5 округляется до ближайшего чётного (в отличие от математического)
    std::cout << "round(2.5) = " << std::round(2.5) << std::endl; // 2 (не 3!)
    std::cout << "round(3.5) = " << std::round(3.5) << std::endl; // 4

    // fmod(x, y): остаток от деления x на y (для дробных чисел)
    // Знак результата совпадает со знаком делимого (x)
    double rem = std::fmod(10.5, 3.0);
    std::cout << "fmod(10.5, 3.0) = " << rem << std::endl; // 1.5

    // trunc(x): отбрасывание дробной части (C++11)
    std::cout << "trunc(3.7) = " << std::trunc(x) << std::endl; // 3

    return 0;
}
```

#### 3. Тригонометрия (аргументы в радианах)

```cpp
#include <iostream>
#include <cmath>
#include <windows.h>

int main() {
    SetConsoleCP(1251);
    SetConsoleOutputCP(1251);

    // Константа PI: M_PI не входит в стандарт, но поддерживается большинством компиляторов
    // При необходимости используйте acos(-1.0) или std::numbers::pi (C++20)
    const double PI = 3.14159265358979323846;
    double angle = PI / 4; // 45 градусов

    // sin, cos, tan: принимают радианы
    std::cout << "sin(45°) = " << std::sin(angle) << std::endl; // ~0.7071
    std::cout << "cos(45°) = " << std::cos(angle) << std::endl; // ~0.7071
    std::cout << "tan(45°) = " << std::tan(angle) << std::endl; // ~1

    // asin, acos, atan: возвращают радианы
    double val = 0.5;
    std::cout << "asin(0.5) = " << std::asin(val) << " rad" << std::endl;

    // atan2(y, x): угол между осью X и точкой (x, y), учитывает квадрант
    // atan(y/x) теряет информацию о знаках
    std::cout << "atan2(1, 1)  = " << std::atan2(1.0, 1.0) << " rad" << std::endl;   // PI/4
    std::cout << "atan2(1, -1) = " << std::atan2(1.0, -1.0) << " rad" << std::endl; // 3*PI/4

    return 0;
}
```

#### 4. Экспонента и логарифмы

```cpp
#include <iostream>
#include <cmath>
#include <windows.h>

int main() {
    SetConsoleCP(1251);
    SetConsoleOutputCP(1251);

    double x = 2.0;

    // exp(x): e в степени x
    std::cout << "exp(2.0)   = " << std::exp(x) << std::endl; // ~7.389

    // log(x): натуральный логарифм (основание e)
    // При x <= 0 возвращает -inf или NaN
    std::cout << "ln(2.0)    = " << std::log(x) << std::endl; // ~0.693

    // log10(x): десятичный логарифм
    std::cout << "log10(100) = " << std::log10(100.0) << std::endl; // 2

    // log2(x): двоичный логарифм (C++11)
    std::cout << "log2(8)    = " << std::log2(8.0) << std::endl; // 3

    // Логарифм по произвольному основанию через замену основания:
    // log_b(x) = ln(x) / ln(b)
    double base = 3.0, value = 81.0;
    double log_base3 = std::log(value) / std::log(base);
    std::cout << "log_3(81)  = " << log_base3 << std::endl; // 4

    return 0;
}
```

---

### Часть вторая: библиотека `<string>`

`std::string` — это специализация `std::basic_string<char>`. Это полноценный класс, который управляет памятью автоматически и предоставляет множество методов для поиска, извлечения, замены и модификации.

#### 1. Создание, доступ и итерация

```cpp
#include <iostream>
#include <string>
#include <windows.h>

int main() {
    SetConsoleCP(1251);
    SetConsoleOutputCP(1251);

    // Различные способы создания строк
    std::string s1;                          // Пустая строка
    std::string s2 = "Привет";               // Инициализация из C-строки
    std::string s3(5, 'A');                  // "AAAAA" (5 символов 'A')
    std::string s4 = s2;                     // Копия s2
    std::string s5(s2, 0, 3);                // "При" (подстрока s2 с позиции 0, 3 символа)

    std::cout << "s2 = " << s2 << std::endl;
    std::cout << "s3 = " << s3 << std::endl;
    std::cout << "s5 = " << s5 << std::endl;

    // Доступ к символам
    // operator[]: без проверки границ (быстро, но опасно)
    // .at(): с проверкой границ (бросает std::out_of_range)
    std::cout << "s2[0]   = " << s2[0] << std::endl;     // 'П'
    std::cout << "s2.at(1) = " << s2.at(1) << std::endl; // 'р'

    // Длина строки
    // .size() и .length() — синонимы
    std::cout << "Длина s2: " << s2.size() << " байт" << std::endl;
    // Внимание: для кириллицы в кодировке 1251 1 символ = 1 байт,
    // но в UTF-8 русская буква занимает 2 байта.

    // front() и back() (C++11): первый и последний символы
    std::cout << "Первый: " << s2.front() << ", последний: " << s2.back() << std::endl;

    // Проверка на пустоту: .empty() предпочтительнее, чем size() == 0
    if (!s1.empty()) {
        std::cout << "s1 не пуста" << std::endl;
    } else {
        std::cout << "s1 пуста" << std::endl;
    }

    return 0;
}
```

#### 2. Модификация: добавление, вставка, удаление

```cpp
#include <iostream>
#include <string>
#include <windows.h>

int main() {
    SetConsoleCP(1251);
    SetConsoleOutputCP(1251);

    std::string s = "Мир";

    // append / operator+=: добавление в конец
    s.append(" во");        // "Мир во"
    s += " всем";           // "Мир во всем"
    s.push_back('!');       // "Мир во всем!" (добавляет один символ)
    std::cout << "После добавления: " << s << std::endl;

    // insert(pos, str): вставка перед позицией pos
    // Позиция 0 — начало строки
    s.insert(0, ">> ");
    std::cout << "После insert(0): " << s << std::endl; // ">> Мир во всем!"

    // erase(pos, len): удаление len символов начиная с pos
    // Если len не указан — удаляет до конца
    s.erase(0, 3); // Удаляем ">> "
    std::cout << "После erase: " << s << std::endl; // "Мир во всем!"

    // replace(pos, len, new_str): замена части строки
    // Заменяем 3 символа начиная с позиции 0 (т.е. "Мир") на "Hello"
    s.replace(0, 3, "Hello");
    std::cout << "После replace: " << s << std::endl; // "Hello во всем!"

    // clear(): полная очистка
    s.clear();
    std::cout << "После clear, длина = " << s.size() << std::endl; // 0

    return 0;
}
```

#### 3. Поиск и извлечение подстрок

```cpp
#include <iostream>
#include <string>
#include <windows.h>

int main() {
    SetConsoleCP(1251);
    SetConsoleOutputCP(1251);

    std::string text = "Пример текста для поиска";

    // find(str, pos): поиск первого вхождения подстроки начиная с pos
    // Возвращает позицию или std::string::npos, если не найдено
    size_t pos = text.find("текста");
    if (pos != std::string::npos) {
        std::cout << "Найдено 'текста' на позиции: " << pos << std::endl;
    }

    // rfind(str): поиск последнего вхождения
    std::string repeated = "abcabc";
    std::cout << "Первое 'abc': " << repeated.find("abc") << std::endl;  // 0
    std::cout << "Последнее 'abc': " << repeated.rfind("abc") << std::endl; // 3

    // find_first_of(chars): первая позиция любого символа из набора
    // Полезно для поиска разделителей
    std::string csv = "яблоко,банан;вишня";
    size_t delim_pos = csv.find_first_of(",;");
    std::cout << "Первый разделитель на позиции: " << delim_pos << std::endl;

    // find_first_not_of(chars): первый символ, НЕ входящий в набор
    std::string spaces = "   текст";
    size_t start = spaces.find_first_not_of(" ");
    std::cout << "Первый непробельный символ на позиции: " << start << std::endl;

    // substr(pos, len): извлечение подстроки
    // Если len не указан — до конца строки
    std::string sub = text.substr(0, 6); // "Пример"
    std::cout << "Подстрока: " << sub << std::endl;

    return 0;
}
```

#### 4. Преобразование и сравнение

```cpp
#include <iostream>
#include <string>
#include <windows.h>

int main() {
    SetConsoleCP(1251);
    SetConsoleOutputCP(1251);

    // c_str(): получить C-строку (const char*) с завершающим нулём
    // Нужно для функций, ожидающих const char* (например, printf, старые API)
    std::string s = "Текст";
    const char* cstr = s.c_str();
    printf("Через printf: %s\n", cstr);

    // Внимание: указатель, возвращённый c_str(), действителен только пока
    // строка не будет модифицирована или уничтожена. Не сохраняйте его надолго.

    // Сравнение: операторы ==, !=, <, > и т.д. сравнивают лексикографически
    std::string a = "apple";
    std::string b = "banana";
    if (a < b) {
        std::cout << a << " < " << b << std::endl;
    }

    // compare(): возвращает 0 при равенстве, <0 если *this < other, >0 если *this > other
    int cmp = a.compare(b);
    std::cout << "compare(a,b) = " << cmp << std::endl; // отрицательное число

    // stoi, stod, stof: преобразование строки в число (C++11)
    // Бросают std::invalid_argument или std::out_of_range при ошибке
    std::string num_str = "123";
    int num = std::stoi(num_str);
    std::cout << "stoi(\"123\") = " << num << std::endl;

    std::string float_str = "3.14";
    double d = std::stod(float_str);
    std::cout << "stod(\"3.14\") = " << d << std::endl;

    // to_string: преобразование числа в строку (C++11)
    std::string from_int = std::to_string(42);
    std::string from_double = std::to_string(3.14159);
    std::cout << "to_string(42) = " << from_int << std::endl;
    std::cout << "to_string(3.14159) = " << from_double << std::endl;

    return 0;
}
```

#### 5. Чтение строк с пробелами

```cpp
#include <iostream>
#include <string>
#include <windows.h>

int main() {
    SetConsoleCP(1251);
    SetConsoleOutputCP(1251);

    std::string name;

    // cin >> name: читает до первого пробела/перевода строки
    // "Иван Петров" -> сохранит только "Иван"
    std::cout << "Введите имя и фамилию (cin >>): ";
    std::cin >> name;
    std::cout << "Прочитано: [" << name << "]" << std::endl;

    // Очистка буфера: после cin >> в потоке остаётся пробел и остаток строки
    std::cin.ignore(10000, '\n');

    // getline(cin, str): читает целую строку до перевода строки
    std::cout << "Введите имя и фамилию (getline): ";
    std::getline(std::cin, name);
    std::cout << "Прочитано: [" << name << "]" << std::endl;

    return 0;
}
```

---

### Полный пример: совместное использование

```cpp
#include <iostream>
#include <cmath>
#include <string>
#include <windows.h>

int main() {
    SetConsoleCP(1251);
    SetConsoleOutputCP(1251);

    // Задача: вычислить гипотенузу по двум катетам, введённым пользователем
    std::string input_a, input_b;

    std::cout << "Введите катет a: ";
    std::getline(std::cin, input_a);

    std::cout << "Введите катет b: ";
    std::getline(std::cin, input_b);

    // Преобразование строк в числа
    double a = std::stod(input_a);
    double b = std::stod(input_b);

    // Вычисление
    double c = std::hypot(a, b);

    // Формирование результата с помощью строковых операций
    std::string result = "Гипотенуза = " + std::to_string(c);

    std::cout << result << std::endl;

    return 0;
}
```

### Важные замечания

1. **`<cmath>` и целые числа**: `std::abs` для целых чисел находится в `<cstdlib>`, а не в `<cmath>`. Для `double` используйте `std::fabs` или перегрузку `std::abs` из `<cmath>`.
2. **`std::string` и производительность**: конкатенация через `+` создаёт новые строки. В циклах лучше использовать `+=` или `append`.
3. **Кодировка 1251**: каждый русский символ в Windows-1251 занимает ровно **1 байт**. Это упрощает `size()` и индексацию, но делает строку несовместимой с UTF-8. Для межплатформенного кода используйте `std::wstring` и `std::wcout` или UTF-8.
