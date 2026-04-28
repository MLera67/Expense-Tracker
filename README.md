Expense Tracker

Автор: Кундетова Даяна Курс: Python с нуля: первые шаги в программировании (начальный уровень) Дата: 27.04.2026г.
Ссылка: https://github.com/MLera67/Expense-Tracker

## Как использовать
1. Убедитесь, что у вас установлен Python 3.
2. Запустите файл `app.py`.
3. Введите сумму, категорию и дату в соответствующие поля.
4. Нажмите кнопку **«Добавить расход»**.
5. Для подсчёта суммы расходов за период введите даты в поля «С» и «По» и нажмите **«Посчитать сумму»**.
6. Для сохранения данных нажмите **«Сохранить в JSON»**. Для загрузки ранее сохранённых данных нажмите **«Загрузить из JSON»**.

## Примеры использования
*   **Добавить расход:** Сумма — `500`, Категория — `еда`, Дата — `2026-04-23`.
*   **Посчитать сумму:** Введите даты начала и конца недели, чтобы увидеть общие траты.
*   **Фильтрация:** Введите название категории (например, `транспорт`), чтобы увидеть только эти расходы.
### Полное руководство по созданию Expense Tracker


#### Шаг 1. Структура проекта

Создайте папку проекта со следующей структурой:
```
expense_tracker/
├── main.py           # Основной код приложения
├── expenses_data.json # Файл для хранения данных
├── README.md        # Документация проекта
└── .gitignore       # Файл игнорирования для Git
```

#### Шаг 2. Основной код приложения (main.py)

```python
import tkinter as tk
from tkinter import ttk, messagebox, simpledialog
import json
import os
from datetime import datetime

class ExpenseTracker:
    def __init__(self, root):
        self.root = root
        self.root.title("Expense Tracker")
        self.expenses = []
        self.load_data()

        self.create_widgets()

    def create_widgets(self):
        # Форма ввода расходов
        input_frame = tk.Frame(self.root)
        input_frame.pack(pady=10, padx=10, fill="x")

        tk.Label(input_frame, text="Сумма:").grid(row=0, column=0, sticky="w")
        self.amount_entry = tk.Entry(input_frame, width=15)
        self.amount_entry.grid(row=0, column=1, padx=5)

        tk.Label(input_frame, text="Категория:").grid(row=0, column=2, sticky="w")
        self.category_var = tk.StringVar()
        self.category_combo = ttk.Combobox(input_frame, textvariable=self.category_var,
                                      values=["Еда", "Транспорт", "Развлечения", "Жильё", "Другое"])
        self.category_combo.grid(row=0, column=3, padx=5)

        tk.Label(input_frame, text="Дата (ГГГГ-ММ-ДД):").grid(row=0, column=4, sticky="w")
        self.date_entry = tk.Entry(input_frame, width=12)
        self.date_entry.insert(0, datetime.now().strftime("%Y-%m-%d"))
        self.date_entry.grid(row=0, column=5, padx=5)

        # Кнопка добавления расхода
        tk.Button(input_frame, text="Добавить расход",
               command=self.add_expense).grid(row=0, column=6, padx=10)

        # Таблица расходов
        columns = ("Amount", "Category", "Date")
        self.tree = ttk.Treeview(self.root, columns=columns, show="headings")
        self.tree.heading("Amount", text="Сумма")
        self.tree.heading("Category", text="Категория")
        self.tree.heading("Date", text="Дата")
        self.tree.column("Amount", width=100)
        self.tree.column("Category", width=120)
        self.tree.column("Date", width=100)
        self.tree.pack(padx=10, pady=5, fill="both", expand=True)

        # Фильтры и подсчёт суммы
        filter_frame = tk.Frame(self.root)
        filter_frame.pack(pady=5, padx=10, fill="x")

        tk.Label(filter_frame, text="Фильтр по категории:").grid(row=0, column=0)
        self.filter_category_var = tk.StringVar()
        self.filter_category_combo = ttk.Combobox(filter_frame,
                                          textvariable=self.filter_category_var,
                                          values=["Все", "Еда", "Транспорт", "Развлечения", "Жильё", "Другое"])
        self.filter_category_combo.set("Все")
        self.filter_category_combo.grid(row=0, column=1, padx=5)

        tk.Label(filter_frame, text="Период с:").grid(row=0, column=2)
        self.start_date_entry = tk.Entry(filter_frame, width=12)
        self.start_date_entry.insert(0, "2023-01-01")
        self.start_date_entry.grid(row=0, column=3, padx=5)

        tk.Label(filter_frame, text="по:").grid(row=0, column=4)
        self.end_date_entry = tk.Entry(filter_frame, width=12)
        self.end_date_entry.insert(0, datetime.now().strftime("%Y-%m-%d"))
        self.end_date_entry.grid(row=0, column=5, padx=5)

        tk.Button(filter_frame, text="Применить фильтр",
               command=self.apply_filters).grid(row=0, column=6, padx=5)
        tk.Button(filter_frame, text="Сбросить фильтр",
               command=self.reset_filters).grid(row=0, column=7, padx=5)

        # Подсчёт суммы за период
        self.total_label = tk.Label(self.root, text="Общая сумма за период: 0 руб.",
                               font=("Arial", 12, "bold"))
        self.total_label.pack(pady=5)

        # Кнопки сохранения и загрузки
        btn_frame = tk.Frame(self.root)
        btn_frame.pack(pady=10)

        tk.Button(btn_frame, text="Сохранить данные",
               command=self.save_data).pack(side="left", padx=5)
        tk.Button(btn_frame, text="Загрузить данные",
               command=self.load_data).pack(side="left", padx=5)


        self.update_table()

    def add_expense(self):
        # Проверка корректности ввода
        try:
            amount = float(self.amount_entry.get())
            if amount <= 0:
                raise ValueError("Сумма должна быть положительной")
        except ValueError:
            messagebox.showerror("Ошибка", "Введите корректную сумму (положительное число)")
            return

        category = self.category_var.get()
        if not category:
            messagebox.showerror("Ошибка", "Выберите категорию")
            return

        date_str = self.date_entry.get()
        try:
            datetime.strptime(date_str, "%Y-%m-%d")
        except ValueError:
            messagebox.showerror("Ошибка", "Неверный формат даты (используйте ГГГГ-ММ-ДД)")
            return

        # Добавляем расход
        expense = {"amount": amount, "category": category, "date": date_str}
        self.expenses.append(expense)

        # Очищаем поля ввода
        self.amount_entry.delete(0, tk.END)
        self.category_combo.set("")
        self.date_entry.delete(0, tk.END)
        self.date_entry.insert(0, datetime.now().strftime("%Y-%m-%d"))

        self.update_table()

    def update_table(self, filtered_expenses=None):
        # Обновляем таблицу
        for item in self.tree.get_children():
            self.tree.delete(item)

        display_expenses = filtered_expenses if filtered_expenses is not None else self.expenses

        for expense in display_expenses:
            self.tree.insert("", "end", values=(
                f"{expense['amount']:.2f} руб.",
                expense["category"],
                expense["date"]
            ))

        # Обновляем общую сумму
        self.calculate_total(display_expenses)

    def calculate_total(self, expenses=None):
        total = sum(expense["amount"] for expense in (expenses or self.expenses))
        self.total_label.config(text=f"Общая сумма за период: {total:.2f} руб.")

    def apply_filters(self):
        filtered = self.expenses

        # Фильтр по категории
        selected_category = self.filter_category_var.get()
        if selected_category != "Все":
            filtered = [exp for exp in filtered if exp["category"] == selected_category]

        # Фильтр по дате
        try:
            start_date = datetime.strptime(self.start_date_entry.get(), "%Y-%m-%d")
            end_date = datetime.strpt
