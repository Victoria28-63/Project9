Разберу задачу пошагово — от создания интерфейса до публикации на GitHub.

## Шаг 1. Создание интерфейса и базовой логики (Python + Tkinter)

**1. Установите зависимости** (если нужно):
```bash
pip install tkinter
```

**2. Код приложения (`weather_diary.py`):**
```python
import tkinter as tk
from tkinter import ttk, messagebox
import json
from datetime import datetime

class WeatherDiary:
    def __init__(self, root):
        self.root = root
        self.root.title("Weather Diary")
        self.entries = []
        self.load_data()

        # Поля ввода
        tk.Label(root, text="Дата (YYYY-MM-DD):").grid(row=0, column=0, padx=5, pady=5)
        self.date_entry = tk.Entry(root)
        self.date_entry.grid(row=0, column=1, padx=5, pady=5)

        tk.Label(root, text="Температура (°C):").grid(row=1, column=0, padx=5, pady=5)
        self.temp_entry = tk.Entry(root)
        self.temp_entry.grid(row=1, column=1, padx=5, pady=5)

        tk.Label(root, text="Описание погоды:").grid(row=2, column=0, padx=5, pady=5)
        self.desc_entry = tk.Entry(root)
        self.desc_entry.grid(row=2, column=1, padx=5, pady=5)

        tk.Label(root, text="Осадки:").grid(row=3, column=0, padx=5, pady=5)
        self.precip_var = tk.StringVar(value="Нет")
        tk.Radiobutton(root, text="Да", variable=self.precip_var, value="Да").grid(row=3, column=1, sticky="w")
        tk.Radiobutton(root, text="Нет", variable=self.precip_var, value="Нет").grid(row=3, column=1, sticky="e")

        # Кнопки
        tk.Button(root, text="Добавить запись", command=self.add_entry).grid(row=4, column=0, pady=10)
        tk.Button(root, text="Сохранить в JSON", command=self.save_data).grid(row=4, column=1, pady=10)

        # Таблица для отображения записей
        self.tree = ttk.Treeview(root, columns=("Date", "Temp", "Desc", "Precip"), show="headings")
        self.tree.heading("Date", text="Дата")
        self.tree.heading("Temp", text="Температура")
        self.tree.heading("Desc", text="Описание")
        self.tree.heading("Precip", text="Осадки")
        self.tree.grid(row=5, column=0, columnspan=2, padx=5, pady=5)

        # Фильтры
        tk.Label(root, text="Фильтр по дате (YYYY-MM-DD):").grid(row=6, column=0, padx=5, pady=5)
        self.filter_date_entry = tk.Entry(root)
        self.filter_date_entry.grid(row=6, column=1, padx=5, pady=5)

        tk.Label(root, text="Фильтр по температуре (>):").grid(row=7, column=0, padx=5, pady=5)
        self.filter_temp_entry = tk.Entry(root)
        self.filter_temp_entry.grid(row=7, column=1, padx=5, pady=5)

        tk.Button(root, text="Применить фильтры", command=self.apply_filters).grid(row=8, column=0, columnspan=2, pady=10)

        self.refresh_table()

    def validate_input(self):
        try:
            date = datetime.strptime(self.date_entry.get(), "%Y-%m-%d")
        except ValueError:
            messagebox.showerror("Ошибка", "Неверный формат даты. Используйте YYYY-MM-DD.")
            return False

        try:
            temp = float(self.temp_entry.get())
        except ValueError:
            messagebox.showerror("Ошибка", "Температура должна быть числом.")
            return False

        if not self.desc_entry.get():
            messagebox.showerror("Ошибка", "Описание не может быть пустым.")
            return False

        return True

    def add_entry(self):
        if not self.validate_input():
            return

        entry = {
            "date": self.date_entry.get(),
            "temperature": float(self.temp_entry.get()),
            "description": self.desc_entry.get(),
            "precipitation": self.precip_var.get()
        }
        self.entries.append(entry)
        self.refresh_table()
        self.clear_inputs()

    def clear_inputs(self):
        self.date_entry.delete(0, tk.END)
        self.temp_entry.delete(0, tk.END)
        self.desc_entry.delete(0, tk.END)
        self.precip_var.set("Нет")

    def refresh_table(self, filtered_entries=None):
        for item in self.tree.get_children():
            self.tree.delete(item)

        entries_to_show = filtered_entries if filtered_entries is not None else self.entries
        for entry in entries_to_show:
            self.tree.insert("", "end", values=(
                entry["date"],
                entry["temperature"],
                entry["description"],
                entry["precipitation"]
            ))

    def apply_filters(self):
        filtered = self.entries

        date_filter = self.filter_date_entry.get()
        if date_filter:
            filtered = [e for e in filtered if e["date"] == date_filter]

        temp_filter = self.filter_temp_entry.get()
        if temp_filter:
            try:
                temp_threshold = float(temp_filter)
                filtered = [e for e in filtered if e["temperature"] > temp_threshold]
            except ValueError:
                messagebox.showerror("Ошибка", "Температура фильтра должна быть числом.")
                return

        self.refresh_table(filtered)

    def save_data(self):
        with open("weather_data.json", "w", encoding="utf-8") as f:
            json.dump(self.entries, f, ensure_ascii=False, indent=4)
        messagebox.showinfo("Успех", "Данные сохранены в weather_data.json")

    def load_data(self):
        try:
            with open("weather_data.json", "r", encoding="utf-8") as f:
                self.entries = json.load(f)
        except FileNotFoundError:
            self.entries = []

if __name__ == "__main__":
    root = tk.Tk()
    app = WeatherDiary(root)
    root.mainloop()
```

## Шаг 2. Настройка Git и создание репозитория

1. Создайте папку проекта и перейдите в неё:
```bash
mkdir WeatherDiary
cd WeatherDiary
```
2. Инициализируйте Git‑репозиторий:
```bash
git init
```
3. Создайте файл `.gitignore` для исключения временных файлов:
```
__pycache__/
*.pyc
*.json  # если не хотите сохранять данные в репозитории
```
4. Добавьте файлы в репозиторий и сделайте первый коммит:
```bash
git add .
git commit -m "Initial commit: Weather Diary GUI app"
```

## Шаг 3. Публикация на GitHub

1. Создайте новый репозиторий на GitHub (не добавляйте README или .gitignore — они уже есть у вас).
2. Свяжите локальный репозиторий с удалённым:
```bash
git remote add origin https://github.com/yourusername/WeatherDiary.git
```
3. Отправьте код на GitHub:
```bash
git push -u origin main
```

## Шаг 4. Создание README.md

Создайте файл `README.md` в корне проекта:
```markdown
# Weather Diary

Автор: [Кантлокова Крастина]

## Описание

Приложение "Weather Diary" позволяет вести дневник погоды с возможностью:
* добавления записей (дата, температура