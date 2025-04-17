import streamlit as st
import sqlite3
from datetime import datetime

# Initialize DB
def init_db():
    conn = sqlite3.connect("database.db")
    c = conn.cursor()
    c.execute('''
        CREATE TABLE IF NOT EXISTS expenses (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            title TEXT NOT NULL,
            amount REAL NOT NULL,
            date TEXT NOT NULL
        )
    ''')
    conn.commit()
    conn.close()

# Add new expense
def add_expense(title, amount, date):
    conn = sqlite3.connect("database.db")
    c = conn.cursor()
    c.execute("INSERT INTO expenses (title, amount, date) VALUES (?, ?, ?)",
              (title, amount, date))
    conn.commit()
    conn.close()

# Get all expenses
def get_expenses():
    conn = sqlite3.connect("database.db")
    c = conn.cursor()
    c.execute("SELECT * FROM expenses ORDER BY date DESC")
    data = c.fetchall()
    conn.close()
    return data

# Delete expense
def delete_expense(expense_id):
    conn = sqlite3.connect("database.db")
    c = conn.cursor()
    c.execute("DELETE FROM expenses WHERE id = ?", (expense_id,))
    conn.commit()
    conn.close()

# Main app
init_db()
st.title("💸 Expense Tracker")

menu = ["Add Expense", "View Expenses"]
choice = st.sidebar.selectbox("Menu", menu)

if choice == "Add Expense":
    st.subheader("Add a New Expense")
    with st.form(key="expense_form"):
        title = st.text_input("Expense Title")
        amount = st.number_input("Amount", min_value=0.0, format="%.2f")
        date = st.date_input("Date", value=datetime.today())
        submit = st.form_submit_button("Add Expense")

    if submit:
        add_expense(title, amount, date.strftime('%Y-%m-%d'))
        st.success(f"Added: {title} - ${amount}")

elif choice == "View Expenses":
    st.subheader("All Expenses")
    expenses = get_expenses()
    for exp in expenses:
        st.markdown(f"**{exp[1]}** - ${exp[2]} on *{exp[3]}*")
        if st.button(f"Delete {exp[0]}", key=exp[0]):
            delete_expense(exp[0])
            st.warning("Deleted")
            st.experimental_rerun()
