
# Лабораторная работа №3

<div align="center">

**МИНИСТЕРСТВО НАУКИ И ВЫСШЕГО ОБРАЗОВАНИЯ РОССИЙСКОЙ ФЕДЕРАЦИИ**  
**ФЕДЕРАЛЬНОЕ ГОСУДАРСТВЕННОЕ БЮДЖЕТНОЕ ОБРАЗОВАТЕЛЬНОЕ УЧРЕЖДЕНИЕ ВЫСШЕГО ОБРАЗОВАНИЯ**  
**«САХАЛИНСКИЙ ГОСУДАРСТВЕННЫЙ УНИВЕРСИТЕТ»**

<br>
<br>

Институт естественных наук и техносферной безопасности  
Кафедра информатики  
**Спивакова Екатерина Максимовна**

<br>
<br>
<br>
<br>

Лабораторная работа №3  
**«Реализация списка объектов с фильтрацией с использованием .map, .filter, .sortedBy»**  
01.03.02 Прикладная математика и информатика  
3 Курс

<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>

<div align="right">
Научный руководитель<br>
Соболев Евгений Игоревич
</div>

<br>
<br>
<br>

г. Южно-Сахалинск  
2026 г.

</div>

---

## Цель Работы

Изучить функциональные методы обработки коллекций в Kotlin (`filter`, `map`, `sortedBy`) на примере списка объектов и вывести результаты в интерфейс Android-приложения.

## Индивидуальное задание: Список фильмов

Было выбрано индивидуальное задание **«Список фильмов»**. Требовалось создать список аниме-фильмов 2025-2026 годов с рейтингами IMDb, отфильтровать фильмы с рейтингом выше 8.0, отсортировать их по году выпуска и вывести список названий с рейтингами.

## Скриншоты

![Результат выполнения приложения](AppScreenshot.png)  
*Рисунок 1 – Работа приложения с отфильтрованными и отсортированными данными*

<br>

![Структура проекта](ProjectStructure.png)  
*Рисунок 2 – Структура проекта с пакетом models*

<br>

![Код с цепочками вызовов](CodeChains.png)
*Рисунок 3 – Пример цепочки filter + sortedBy + map в коде*

## Листинги

### 1. Класс данных AnimeFilm (`AnimeFilm.kt`)

```kotlin
package com.example.myfirstapp.models

data class AnimeFilm(
    val name: String,
    val genre: String,
    val rating: Double,
    val year: Int
)
```

### 2. Разметка activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:padding="16dp"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Все фильмы:"
        android:textStyle="bold"
        android:textSize="18sp"/>

    <TextView
        android:id="@+id/textOriginal"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginBottom="16dp"/>

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Фильмы с рейтингом выше 8.0 (по году выпуска):"
        android:textStyle="bold"
        android:textSize="18sp"/>

    <TextView
        android:id="@+id/textInStock"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginBottom="16dp"/>

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Фильмы 2025 года по рейтингу:"
        android:textStyle="bold"
        android:textSize="18sp"/>

    <TextView
        android:id="@+id/textSorted"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

</LinearLayout>
```

### 3. MainActivity.kt

```kotlin
package com.example.myfirstapp

import android.os.Bundle
import android.widget.TextView
import androidx.activity.enableEdgeToEdge
import androidx.appcompat.app.AppCompatActivity
import androidx.core.view.ViewCompat
import androidx.core.view.WindowInsetsCompat
import com.example.myfirstapp.models.AnimeFilm

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContentView(R.layout.activity_main)
        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main)) { v, insets ->
            val systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars())
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom)
            insets
        }

        val films = getFilms()

        val originalText = films.joinToString("\n") {
            "${it.name} (${it.year}) - ${it.genre} - ★ ${it.rating}"
        }
        findViewById<TextView>(R.id.textOriginal).text = originalText

        // Фильмы выше 8 звёзд
        val taskResult = films
            .filter { it.rating > 8.0 }
            .sortedByDescending { it.year }
            .map { "${it.name} - ★ ${it.rating} (${it.year})" }

        findViewById<TextView>(R.id.textInStock).text = taskResult.joinToString("\n")

        // Фильмы по рейтингу
        val filmsRated = films
            .filter {it.year == 2025}
            .sortedByDescending { it.rating }
            .map { "${it.name} - ★ ${it.rating}" }

            findViewById<TextView>(R.id.textSorted).text = filmsRated.joinToString("\n")
    }

    private fun getFilms(): List<AnimeFilm> {
        return listOf(
            AnimeFilm("Человек-бензопила: Арка Резе", "Тёмное фэнтези, экшен", 8.6, 2025),
            AnimeFilm("Истребитель демонов: Бесконечная крепость", "Экшен, фэнтези", 8.5, 2025),
            AnimeFilm("Магическая битва: Казнь", "Боевик, ужасы, фэнтези", 8.5, 2026),
            AnimeFilm("Кейпоп-охотницы на демонов", "Мюзикл, фэнтези, боевик", 7.5, 2025),
            AnimeFilm("Зверополис 2", "Детектив, комедия", 7.4, 2025),
            AnimeFilm("Плохие парни 2", "Комедия, криминал", 7.0, 2025),
            AnimeFilm("Элио", "Фантастика, приключения", 6.7, 2025)
        )
    }
}
```

## Ответы на контрольные вопросы

**1. Что возвращает функция `filter` – новый список или изменяет существующий?**  

`filter` всегда возвращает **новый список**. Она не меняет исходный, а создаёт отдельный список только с элементами, которые подходят под условие. Например, `list.filter { it > 5 }` вернёт новый список с числами больше 5, а старый останется без изменений.

**2. В чём разница между `sortedBy` и `sortedByDescending`?**  

- `sortedBy` сортирует **по возрастанию** (от меньшего к большему).
- `sortedByDescending` сортирует **по убыванию** (от большего к меньшему).

**Пример:** `sortedBy { it.year }` → 2025, 2026. `sortedByDescending { it.year }` → 2026, 2025.

**3. Как можно объединить несколько условий в `filter`?**  

Через логические операторы:
- `&&` (И) — оба условия должны выполняться
- `||` (ИЛИ) — хотя бы одно условие должно выполняться

**Пример:** `filter { it.rating > 8.0 && it.year == 2025 }` — фильмы 2025 года с рейтингом выше 8.0.

**4. Для чего используется функция `map`? Приведите пример.**  

`map` преобразует каждый элемент коллекции во что-то другое и возвращает новый список с результатами. Например, было `[фильм1, фильм2]`, после `map` стало `["Название1", "Название2"]`. Удобно, когда нужно из объектов достать только конкретные поля или переделать их в строки для вывода.

**5. Что такое `joinToString` и как она работает?**  

`joinToString` склеивает список в одну строку. Можно указать, что ставить между элементами — запятую, пробел, перенос строки (`\n`). В работе используется `joinToString("\n")`, чтобы каждый элемент списка выводился с новой строки в TextView.

## Вывод

В ходе лабораторной работы был создан Android-проект, демонстрирующий применение функциональных методов обработки коллекций в Kotlin.

Были изучены и применены на практике:

- **Функция `filter`** – для отбора фильмов с рейтингом выше 8.0
- **Функция `sortedByDescending`** – для сортировки отфильтрованных фильмов по году выпуска (от новых к старым)
- **Функция `map`** – для преобразования объектов в строки с названиями и рейтингами
- **Цепочки вызовов** – комбинирование нескольких методов для решения комплексных задач
- **Функция `joinToString`** – для форматированного вывода списков в TextView

Для индивидуального задания **«Список фильмов»** был создан data class `AnimeFilm` с полями: название, жанр, рейтинг, год выпуска. Сформирован тестовый список из 7 популярных аниме-фильмов 2025-2026 годов с реальными рейтингами IMDb.

В интерфейсе приложения реализовано три блока вывода:
1. **Все фильмы** – исходный список для наглядности
2. **Фильмы с рейтингом выше 8.0 (по году выпуска)** – результат выполнения индивидуального задания (цепочка `filter` → `sortedByDescending` → `map`)
3. **Фильмы 2025 года по рейтингу** – дополнительный пример, показывающий все фильмы 2025, отсортированные по рейтингу от наилучших к худшим.

Приложение успешно запускается на эмуляторе и корректно отображает все три списка, что подтверждает правильность применения функциональных методов обработки коллекций.

Таким образом, цель работы достигнута: получены практические навыки использования `filter`, `map`, `sortedBy` для обработки списков объектов в Android-приложении на Kotlin.
