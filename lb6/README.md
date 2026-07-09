
# Лабораторная работа №6

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

Лабораторная работа №6  
**«Модернизация ToDo-списка: RecyclerView, CardView, удаление свайпом»**  
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

## Цель работы

Модернизировать существующее приложение ToDo-списка (Лабораторная работа №5): заменить статическое отображение задач на динамический `RecyclerView` с использованием `CardView`, добавить чекбоксы для визуальной отметки выполнения и реализовать удаление задачи свайпом с возможностью отмены через `Snackbar`.

## Индивидуальное задание

Реализовано **удаление задачи свайпом** влево или вправо. При свайпе задача удаляется из списка, появляется `Snackbar` с кнопкой «Отменить». При нажатии на кнопку задача восстанавливается на прежнее место. Весь функционал из лабораторной работы №5 (счётчик, поле ввода текста, добавление задач, удаление по индексу, очистка списка) сохранён.


## Листинги

### 1. `item_task.xml`

```xml
<androidx.cardview.widget.CardView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="8dp"
    app:cardCornerRadius="8dp"
    app:cardElevation="4dp">

    <LinearLayout ...>
        <TextView android:id="@+id/textTask" ... />
        <CheckBox android:id="@+id/checkTask" ... />
    </LinearLayout>
</androidx.cardview.widget.CardView>
```

Полный файл: `app/src/main/res/layout/item_task.xml`.

### 2. `TaskAdapter.kt`

```kotlin
class TaskAdapter(
    private val tasks: MutableList<TaskItem>,
    private val onCheckedChanged: () -> Unit,
    private val onItemLongClick: (Int) -> Unit
) : RecyclerView.Adapter<TaskAdapter.TaskViewHolder>() {

    class TaskViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        val textTask: TextView = itemView.findViewById(R.id.textTask)
        val checkTask: CheckBox = itemView.findViewById(R.id.checkTask)
    }

    override fun onBindViewHolder(holder: TaskViewHolder, position: Int) {
        val task = tasks[position]
        holder.textTask.text = task.text
        holder.checkTask.setOnCheckedChangeListener(null)
        holder.checkTask.isChecked = task.isDone
        // перечёркивание + обновление счётчика
        holder.itemView.setOnLongClickListener {
            onItemLongClick(holder.bindingAdapterPosition)
            true
        }
    }
}
```

### 3. `MainActivity.kt` (фрагмент)

```kotlin
recyclerView.layoutManager = LinearLayoutManager(this)
adapter = TaskAdapter(tasks, onCheckedChanged = { updateCompletedCount() }, ...)
recyclerView.adapter = adapter

buttonAddTask.setOnClickListener {
    tasks.add(TaskItem(text))
    adapter.notifyItemInserted(tasks.lastIndex)
    updateCompletedCount()
}

private fun updateCompletedCount() {
    val done = tasks.count { it.isDone }
    textCompletedCount.text = getString(R.string.completed_count, done, tasks.size)
}
```


## Ответы на контрольные вопросы

**1. Для чего нужен `RecyclerView`? Чем он лучше `ListView`?**  

`RecyclerView` – это более гибкий и производительный компонент для отображения больших списков. Он лучше `ListView` по следующим причинам:
- **Принудительное использование `ViewHolder`** – уменьшает количество вызовов `findViewById`.
- **Разделение ответственности** – `LayoutManager` отвечает за расположение элементов, `Adapter` – за данные, `ItemAnimator` – за анимации.
- **Встроенная поддержка разных типов элементов** (например, заголовки и строки).
- **Эффективная работа с большими списками** за счёт переиспользования views.
- **Гибкая настройка анимаций** добавления/удаления.

**2. Какие компоненты необходимы для работы `RecyclerView`?**  

Для работы `RecyclerView` необходимы три ключевых компонента:
- **`RecyclerView`** – сам контейнер, размещаемый в разметке.
- **`LayoutManager`** – определяет способ расположения элементов (LinearLayoutManager, GridLayoutManager, StaggeredGridLayoutManager).
- **`Adapter`** – подготавливает данные и создаёт ViewHolder'ы.
- **`ViewHolder`** – внутренний класс адаптера, кэширующий ссылки на виджеты элемента списка (обязателен, но реализуется внутри адаптера).

**3. Что такое `ViewHolder` и для чего он используется?**  

`ViewHolder` – это объект-обёртка, который хранит ссылки на дочерние View одного элемента списка (например, `TextView` и `CheckBox`). Он используется для избежания частых вызовов `findViewById()` при прокрутке списка. Без `ViewHolder` при каждом связывании данных система искала бы компоненты по id, что замедляет работу. `RecyclerView` требует явного создания `ViewHolder` в адаптере.

**4. Чем отличается `notifyDataSetChanged()` от `notifyItemInserted()`?**  

- **`notifyDataSetChanged()`** – уведомляет, что изменился весь набор данных. `RecyclerView` перерисовывает все видимые элементы, что неэффективно. Не использует анимации по умолчанию.
- **`notifyItemInserted(int position)`** – сообщает, что в конкретной позиции вставлен новый элемент. `RecyclerView` запускает анимацию появления и перерисовывает только затронутые элементы, что оптимально.

Аналогично есть `notifyItemRemoved()`, `notifyItemChanged()` и другие.

**5. Как добавить обработку кликов на элементы `RecyclerView`?**  

В `RecyclerView` нет встроенного метода `setOnItemClickListener`. Обработку кликов нужно реализовать самостоятельно, обычно через интерфейс обратного вызова в адаптере:

```kotlin
class TaskAdapter(
    private val tasks: List<String>,
    private val onItemClick: (Int) -> Unit
) : RecyclerView.Adapter<...>() {
    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.itemView.setOnClickListener { onItemClick(position) }
    }
}
```

Затем в `MainActivity` передать лямбду: `TaskAdapter(tasks) { position -> ... }`. Аналогично обрабатываются долгие нажатия, свайпы и другие жесты.

## Вывод

В ходе выполнения лабораторной работы проведена модернизация приложения ToDo-списка. Вместо устаревшего `TextView` внутри `ScrollView` внедрён современный компонент `RecyclerView` с использованием `CardView` для каждого элемента. Добавлены чекбоксы, позволяющие визуально отмечать выполненные задачи (перечёркивание текста). Реализовано удаление задачи свайпом с отменой через `Snackbar` – одно из самых популярных и удобных UX-решений. 

При этом полностью сохранён весь функционал предыдущей лабораторной работы:
- счётчик нажатий с сохранением через `SharedPreferences`;
- поле ввода произвольного текста и его отображение;
- добавление новых задач;
- удаление задачи по индексу (индивидуальное задание №5);
- очистка всего списка.

Благодаря использованию `RecyclerView` и `notifyItemInserted()` / `notifyItemRemoved()` приложение работает быстрее и анимации удаления/добавления выглядят плавно. `Snackbar` с кнопкой «Отменить» улучшает пользовательский опыт: случайно удалённую задачу можно мгновенно восстановить.
