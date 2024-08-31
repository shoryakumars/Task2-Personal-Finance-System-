# Task2-Personal-Finance-System-
import tkinter as tk
from tkinter import messagebox
import sqlite3
import matplotlib.pyplot as plt

# Database file
DATABASE_FILE = 'finance.db'


class FinanceManager:
    def _init_(self, root):
        self.root = root
        self.root.title("Personal Finance Management System")
        self.root.geometry("500x500")

        self.conn = sqlite3.connect(DATABASE_FILE)
        self.create_table()

        self.income_label = tk.Label(root, text="Income")
        self.income_label.pack()

        self.income_entry = tk.Entry(root)
        self.income_entry.pack()

        self.expense_label = tk.Label(root, text="Expense")
        self.expense_label.pack()

        self.expense_entry = tk.Entry(root)
        self.expense_entry.pack()

        self.expense_desc_label = tk.Label(root, text="Description")
        self.expense_desc_label.pack()

        self.expense_desc_entry = tk.Entry(root)
        self.expense_desc_entry.pack()

        self.add_income_button = tk.Button(root, text="Add Income", command=self.add_income)
        self.add_income_button.pack()

        self.add_expense_button = tk.Button(root, text="Add Expense", command=self.add_expense)
        self.add_expense_button.pack()

        self.view_summary_button = tk.Button(root, text="View Summary", command=self.view_summary)
        self.view_summary_button.pack()

        self.plot_expenses_button = tk.Button(root, text="Plot Expenses", command=self.plot_expenses)
        self.plot_expenses_button.pack()

    def create_table(self):
        with self.conn:
            self.conn.execute('''
                CREATE TABLE IF NOT EXISTS finance (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    type TEXT,
                    amount REAL,
                    description TEXT,
                    date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                )
            ''')

    def add_income(self):
        try:
            amount = float(self.income_entry.get().strip())
            with self.conn:
                self.conn.execute('INSERT INTO finance (type, amount) VALUES (?, ?)', ('income', amount))
            messagebox.showinfo("Success", "Income added successfully!")
            self.clear_entries()
        except ValueError:
            messagebox.showerror("Error", "Please enter a valid amount.")

    def add_expense(self):
        try:
            amount = float(self.expense_entry.get().strip())
            description = self.expense_desc_entry.get().strip()
            with self.conn:
                self.conn.execute('INSERT INTO finance (type, amount, description) VALUES (?, ?, ?)', ('expense', amount, description))
            messagebox.showinfo("Success", "Expense added successfully!")
            self.clear_entries()
        except ValueError:
            messagebox.showerror("Error", "Please enter a valid amount.")

    def view_summary(self):
        with self.conn:
            income = self.conn.execute('SELECT SUM(amount) FROM finance WHERE type="income"').fetchone()[0] or 0
            expenses = self.conn.execute('SELECT SUM(amount) FROM finance WHERE type="expense"').fetchone()[0] or 0

        savings = income - expenses

        summary = f"Total Income: ${income:.2f}\nTotal Expenses: ${expenses:.2f}\nSavings: ${savings:.2f}"
        messagebox.showinfo("Financial Summary", summary)

    def plot_expenses(self):
        with self.conn:
            data = self.conn.execute('SELECT description, amount FROM finance WHERE type="expense"').fetchall()

        if data:
            descriptions, amounts = zip(*data)

            plt.figure(figsize=(10, 6))
            plt.bar(descriptions, amounts, color='salmon')
            plt.xlabel('Description')
            plt.ylabel('Amount ($)')
            plt.title('Expenses Breakdown')
            plt.xticks(rotation=45, ha='right')
            plt.tight_layout()
            plt.show()
        else:
            messagebox.showinfo("Plot Expenses", "No expenses to plot.")

    def clear_entries(self):
        self.income_entry.delete(0, tk.END)
        self.expense_entry.delete(0, tk.END)
        self.expense_desc_entry.delete(0, tk.END)


if _name_ == "_main_":
    root = tk.Tk()
    app = FinanceManager(root)
    root.mainloop()
