# Пример реализации Currency Converter на Python (Tkinter)

Ниже представлен полный код приложения «Currency Converter», который реализует все указанные требования: выбор валют, конвертация через внешний API, сохранение истории в JSON, валидация ввода и структура для работы с Git.

## Структура проекта
```
currency_converter/
│
├── converter.py        # Основной файл приложения
├── history.json        # Файл для хранения истории конвертаций
├── .gitignore          # Игнорируемые файлы для Git
└── README.md           # Описание проекта
```

## Код converter.py

```python
import tkinter as tk
from tkinter import ttk, messagebox
import requests
import json
import os

HISTORY_FILE = 'history.json'
API_KEY = 'YOUR_API_KEY'  # Замените на свой ключ
API_URL = 'https://v6.exchangerate-api.com/v6/{}/pair/{}/{}'

CURRENCIES = [
    "USD", "EUR", "RUB", "GBP", "JPY", "CNY", "CHF", "CAD", "AUD", "NZD"
]

class CurrencyConverterApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Currency Converter")
        self.history = []
        self.load_history()

        # --- Интерфейс ---
        # Валюта "Из"
        tk.Label(root, text="Из:").grid(row=0, column=0, padx=5, pady=5, sticky='e')
        self.from_currency = ttk.Combobox(root, values=CURRENCIES, width=5)
        self.from_currency.set("USD")
        self.from_currency.grid(row=0, column=1, padx=5, pady=5)

        # Валюта "В"
        tk.Label(root, text="В:").grid(row=0, column=2, padx=5, pady=5, sticky='e')
        self.to_currency = ttk.Combobox(root, values=CURRENCIES, width=5)
        self.to_currency.set("RUB")
        self.to_currency.grid(row=0, column=3, padx=5, pady=5)

        # Сумма
        tk.Label(root, text="Сумма:").grid(row=1, column=0, padx=5, pady=5, sticky='e')
        self.amount_entry = tk.Entry(root, width=15)
        self.amount_entry.grid(row=1, column=1, columnspan=3, padx=5, pady=5, sticky='w')

        # Кнопка конвертации
        self.convert_btn = tk.Button(root, text="Конвертировать", command=self.convert)
        self.convert_btn.grid(row=2, column=0, columnspan=4, pady=10)

        # Результат
        self.result_label = tk.Label(root, text="", font=("Arial", 12))
        self.result_label.grid(row=3, column=0, columnspan=4, pady=5)

        # Таблица истории
        self.tree = ttk.Treeview(root, columns=("Из", "В", "Сумма", "Результат"), show='headings')
        self.tree.heading("Из", text="Из")
        self.tree.heading("В", text="В")
        self.tree.heading("Сумма", text="Сумма")
        self.tree.heading("Результат", text="Результат")
        self.tree.grid(row=4, column=0, columnspan=4, padx=5, pady=5, sticky='nsew')

        # Заполнение истории при старте
        self.update_history_tree()

    def load_history(self):
        if os.path.exists(HISTORY_FILE):
            with open(HISTORY_FILE, 'r', encoding='utf-8') as f:
                self.history = json.load(f)
        else:
            self.history = []
            self.save_history()

    def save_history(self):
        with open(HISTORY_FILE, 'w', encoding='utf-8') as f:
            json.dump(self.history, f, ensure_ascii=False, indent=4)

    def update_history_tree(self):
        for i in self.tree.get_children():
            self.tree.delete(i)
        for entry in self.history:
            self.tree.insert('', 'end', values=(entry['from'], entry['to'], entry['amount'], entry['result']))

    def convert(self):
        from_curr = self.from_currency.get()
        to_curr = self.to_currency.get()
        
        try:
            amount = float(self.amount_entry.get())
            if amount <= 0:
                raise ValueError("Сумма должна быть положительной")
        except ValueError as e:
            messagebox.showerror("Ошибка ввода", str(e))
            return

        url = API_URL.format(API_KEY, from_curr, to_curr)
        
        try:
            response = requests.get(url)
            response.raise_for_status()
            data = response.json()
            
            if data.get('result') == 'error':
                messagebox.showerror("Ошибка API", data.get('error-type', 'Неизвестная ошибка'))
                return

            rate = data['conversion_rate']
            result = round(amount * rate, 2)
            
            # Сохранение в историю
            entry = {
                "from": from_curr,
                "to": to_curr,
                "amount": amount,
                "result": result,
                "rate": rate,
                "date": data['time_last_update_utc']
            }
            self.history.insert(0, entry)  # Добавляем в начало списка
            if len(self.history) > 20:  # Ограничиваем историю 20 записями
                self.history.pop()
            self.save_history()
            self.update_history_tree()
            
            self.result_label.config(text=f"Результат: {result} {to_curr}")

        except requests.RequestException as e:
            messagebox.showerror("Ошибка сети", f"Не удалось получить данные от API: {e}")

if __name__ == "__main__":
    root = tk.Tk()
    app = CurrencyConverterApp(root)
    root.mainloop()
```

## Пример .gitignore (для Python)
```
# Byte-compiled / optimized / DLL files
__pycache__/
*.py[cod]
*$py.class

# C extensions
*.so

# Distribution / packaging
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg

# PyInstaller
*.manifest
*.spec

# Installer logs
pip-log.txt

# Unit test / coverage reports
htmlcov/
.tox/
.coverage

# Translations
*.mo

# Visual Studio files (Windows)
.vscode/
```
## Пример README.md

# Currency Converter

## Автор: Иванов Иван

## Описание программы

Приложение «Currency Converter» позволяет быстро конвертировать валюты с использованием актуальных курсов через внешний API. История операций сохраняется в файл и отображается в таблице.

## Как получить API-ключ

1. Перейдите на сайт [exchangerate-api.com](https://www.exchangerate-api.com/).
2. Зарегистрируйтесь и получите бесплатный API-ключ.
3. Вставьте ключ в переменную `API_KEY` в файле `converter.py`.

## Инструкция по запуску

1. Установите Python 3.x.
2. Установите библиотеку `requests`:
   `pip install requests`
3. Запустите приложение:
   `python converter.py`
4. Введите сумму и выберите валюты для конвертации.
5. Результат появится под кнопкой и будет добавлен в историю.

## Примеры использования

- **Конвертация:** выберите валюты (например, из USD в RUB), введите сумму (например, 100) и нажмите «Конвертировать».
- **История:** все операции сохраняются в `history.json` и отображаются в таблице.
- **Валидация:** если ввести отрицательное число или нечисловое значение — появится сообщение об ошибке.
- **Ограничение истории:** хранится до 20 последних операций.
```
> *Код и структура полностью соответствуют требованиям задачи. Для публикации на GitHub создайте репозиторий и следуйте инструкциям в README.*
