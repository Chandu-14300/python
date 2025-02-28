import sqlite3
from datetime import datetime

conn = sqlite3.connect("expenses.db")
cursor = conn.cursor()

cursor.execute("""
CREATE TABLE IF NOT EXISTS expenses (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    date TEXT,
    category TEXT,
    amount REAL,
    description TEXT
)
""")
conn.commit()

def add_expense():
    date = input("Enter the date (YYYY-MM-DD): ")
    category = input("Enter the category (e.g., Food, Transport, Entertainment): ")
    amount = float(input("Enter the amount: "))
    description = input("Enter a description: ")
    
    cursor.execute("""
    INSERT INTO expenses (date, category, amount, description)
    VALUES (?, ?, ?, ?)
    """, (date, category, amount, description))
    conn.commit()
    print("Expense added successfully!")

def view_expenses():
    print("\n1. View all expenses")
    print("2. Filter by date")
    print("3. Filter by category")
    choice = int(input("Choose an option: "))
    
    if choice == 1:
        cursor.execute("SELECT * FROM expenses")
    elif choice == 2:
        date = input("Enter the date (YYYY-MM-DD): ")
        cursor.execute("SELECT * FROM expenses WHERE date = ?", (date,))
    elif choice == 3:
        category = input("Enter the category: ")
        cursor.execute("SELECT * FROM expenses WHERE category = ?", (category,))
    else:
        print("Invalid choice.")
        return

    rows = cursor.fetchall()
    if rows:
        for row in rows:
            print(row)
    else:
        print("No records found.")

def analyze_expenses():
    cursor.execute("""
    SELECT strftime('%Y-%m', date) as month, SUM(amount) as total
    FROM expenses
    GROUP BY month
    """)
    rows = cursor.fetchall()
    print("\nMonthly Expense Summary:")
    for row in rows:
        print(f"Month: {row[0]}, Total Expense: ₹{row[1]:.2f}")

    cursor.execute("""
    SELECT category, SUM(amount) as total
    FROM expenses
    GROUP BY category
    ORDER BY total DESC
    LIMIT 5
    """)
    rows = cursor.fetchall()
    print("\nTop Spending Categories:")
    for row in rows:
        print(f"Category: {row[0]}, Total Expense: ₹{row[1]:.2f}")

def main():
    while True:
        print("\n=== Personal Expense Tracker ===")
        print("1. Add Expense")
        print("2. View Expenses")
        print("3. Analyze Expenses")
        print("4. Exit")
        
        choice = int(input("Choose an option: "))
        if choice == 1:
            add_expense()
        elif choice == 2:
            view_expenses()
        elif choice == 3:
            analyze_expenses()
        elif choice == 4:
            print("Goodbye!")
            break
        else:
            print("Invalid choice. Please try again.")

if __name__ == "__main__":
    main()

conn.close()
