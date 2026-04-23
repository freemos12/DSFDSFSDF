# DSFDSFSDF
import tkinter as tk
from tkinter import ttk, messagebox
import random
import json
import os

class TaskGenerator:
    def __init__(self, root):
        self.root = root
        self.root.title("Random Task Generator")
        self.root.geometry("500x400")

        # Предопределённые задачи с типами
        self.tasks = {
            "учёба": ["Прочитать статью", "Решить 5 задач", "Повторить конспект"],
            "спорт": ["Сделать зарядку", "Пробежать 3 км", "Позаниматься йогой"],
            "работа": ["Проверить почту", "Составить отчёт", "Провести встречу"]
        }

        # Загрузка истории
        self.history = self.load_history()

        self.setup_ui()

    def setup_ui(self):
        # Поле для добавления новой задачи
        tk.Label(self.root, text="Добавить новую задачу:").pack()
        self.task_entry = tk.Entry(self.root, width=40)
        self.task_entry.pack(pady=5)

        # Выбор типа задачи
        tk.Label(self.root, text="Тип задачи:").pack()
        self.type_var = tk.StringVar(value="учёба")
        types = ["учёба", "спорт", "работа"]
        for t in types:
            tk.Radiobutton(self.root, text=t, variable=self.type_var, value=t).pack()

        # Кнопка добавления задачи
        tk.Button(self.root, text="Добавить задачу", command=self.add_task).pack(pady=5)

        # Кнопка генерации
        tk.Button(self.root, text="Сгенерировать задачу", command=self.generate_task, bg="lightblue").pack(pady=10)

        # Отображение текущей задачи
        self.current_task_label = tk.Label(self.root, text="", font=("Arial", 12), wraplength=400)
        self.current_task_label.pack(pady=10)

        # Фильтрация истории
        tk.Label(self.root, text="Фильтр по типу:").pack()
        self.filter_var = tk.StringVar(value="все")
        filter_options = ["все", "учёба", "спорт", "работа"]
        filter_menu = ttk.Combobox(self.root, textvariable=self.filter_var, values=filter_options)
        filter_menu.pack(pady=5)
        filter_menu.bind("<<ComboboxSelected>>", self.update_history_display)

        # Список истории
        tk.Label(self.root, text="История задач:").pack()
        self.history_listbox = tk.Listbox(self.root, height=10, width=60)
        self.history_listbox.pack(pady=5, fill=tk.BOTH, expand=True)

        self.update_history_display()

    def add_task(self):
        task = self.task_entry.get().strip()
        task_type = self.type_var.get()

        if not task:
            messagebox.showerror("Ошибка", "Задача не может быть пустой!")
            return

        if task_type not in self.tasks:
            self.tasks[task_type] = []

        self.tasks[task_type].append(task)
        self.task_entry.delete(0, tk.END)
        messagebox.showinfo("Успех", "Задача добавлена!")

    def generate_task(self):
        all_tasks = []
        for task_list in self.tasks.values():
            all_tasks.extend(task_list)

        if not all_tasks:
            messagebox.showwarning("Предупреждение", "Нет задач для генерации!")
            return

        selected_task = random.choice(all_tasks)
        self.current_task_label.config(text=f"Сгенерированная задача: {selected_task}")
        self.history.append(selected_task)
        self.save_history()
        self.update_history_display()

    def update_history_display(self, event=None):
        self.history_listbox.delete(0, tk.END)
        filter_type = self.filter_var.get()

        if filter_type == "все":
            filtered_history = self.history
        else:
            filtered_history = [task for task in self.history if any(task in tasks for tasks in self.tasks.get(filter_type, []))]

        for task in filtered_history:
            self.history_listbox.insert(tk.END, task)

    def save_history(self):
        with open("history.json", "w", encoding="utf-8") as f:
            json.dump(self.history, f, ensure_ascii=False, indent=2)

    def load_history(self):
        if os.path.exists("history.json"):
            with open("history.json", "r", encoding="utf-8") as f:
                return json.load(f)
        return []

if __name__ == "__main__":
    root = tk.Tk()
    app = TaskGenerator(root)
    root.mainloop()
