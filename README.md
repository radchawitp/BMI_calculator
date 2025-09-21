# BMI_calculator
# Radchawit Prukthaveesak

import gradio as gr
import pandas as pd
import os
from datetime import datetime

USER_DATA_FILE = "user_data.csv"
FOOD_DATA_FILE = "food_log.csv"

def init_csv_files():
    if not os.path.exists(USER_DATA_FILE):
        pd.DataFrame(columns=["Timestamp", "Name", "Age", "Height_cm", "Weight_kg", "BMI", "Category"]).to_csv(USER_DATA_FILE, index=False)
    if not os.path.exists(FOOD_DATA_FILE):
        pd.DataFrame(columns=["Timestamp", "Name", "Food_Item", "Calories"]).to_csv(FOOD_DATA_FILE, index=False)

def calculate_bmi(name, age, height_cm, weight_kg):
    try:
        height_m = height_cm / 100
        bmi = round(weight_kg / (height_m ** 2), 2)

        if bmi < 18.5:
            category = "Underweight"
        elif 18.5 <= bmi < 25:
            category = "Normal weight"
        elif 25 <= bmi < 30:
            category = "Overweight"
        else:
            category = "Obese"

        timestamp = datetime.now().isoformat()
        new_entry = pd.DataFrame([[timestamp, name, age, height_cm, weight_kg, bmi, category]],
                                 columns=["Timestamp", "Name", "Age", "Height_cm", "Weight_kg", "BMI", "Category"])
        new_entry.to_csv(USER_DATA_FILE, mode='a', header=False, index=False)

        return f"{name}, your BMI is {bmi} ({category})"
    except Exception as e:
        return f"Error: {str(e)}"

def log_food(name, food_item, calories):
    try:
        timestamp = datetime.now().isoformat()
        new_food_entry = pd.DataFrame([[timestamp, name, food_item, calories]],
                                      columns=["Timestamp", "Name", "Food_Item", "Calories"])
        new_food_entry.to_csv(FOOD_DATA_FILE, mode='a', header=False, index=False)
        return f"{food_item} ({calories} kcal) logged for {name}."
    except Exception as e:
        return f"Error: {str(e)}"

def view_logs(name):
    try:
        bmi_logs = pd.read_csv(USER_DATA_FILE)
        food_logs = pd.read_csv(FOOD_DATA_FILE)

        user_bmi_logs = bmi_logs[bmi_logs["Name"] == name]
        user_food_logs = food_logs[food_logs["Name"] == name]

        result = "---- BMI Logs ----\n"
        result += user_bmi_logs.to_string(index=False)
        result += "\n\n---- Food Logs ----\n"
        result += user_food_logs.to_string(index=False)

        return result
    except Exception as e:
        return f"Error reading logs: {str(e)}"

init_csv_files()

with gr.Blocks(theme=gr.themes.Default(primary_hue="pink", secondary_hue="blue")) as iface:
    gr.Markdown("# 🧮 BMI & 🍎 Food Tracker")

    with gr.Tab("Calculate BMI"):
        name = gr.Textbox(label="Name")
        age = gr.Number(label="Age", precision=0)
        height = gr.Number(label="Height (cm)")
        weight = gr.Number(label="Weight (kg)")
        bmi_button = gr.Button("Calculate BMI")
        bmi_output = gr.Textbox(label="BMI Result")

        bmi_button.click(fn=calculate_bmi,
                         inputs=[name, age, height, weight],
                         outputs=bmi_output)

    with gr.Tab("Log Food"):
        name2 = gr.Textbox(label="Name")
        food = gr.Textbox(label="Food Item")
        calories = gr.Number(label="Calories")
        log_button = gr.Button("Log Food")
        log_output = gr.Textbox(label="Log Result")

        log_button.click(fn=log_food,
                         inputs=[name2, food, calories],
                         outputs=log_output)

    with gr.Tab("View Logs"):
        name3 = gr.Textbox(label="Name")
        view_button = gr.Button("View Logs")
        logs_output = gr.Textbox(lines=20, label="Your Logs")

        view_button.click(fn=view_logs,
                          inputs=name3,
                          outputs=logs_output)

iface.launch(share=True)

import sqlite3

conn = sqlite3.connect("bmi.db")
cursor = conn.cursor()

cursor.execute("""
CREATE TABLE IF NOT EXISTS bmi_records (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    weight REAL,
    height REAL,
    bmi REAL,
    category TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
)
""")

conn.commit()
conn.close()

def get_history():
    conn = sqlite3.connect("bmi.db")
    cursor = conn.cursor()
    cursor.execute("SELECT weight, height, bmi, category, created_at FROM bmi_records ORDER BY created_at DESC LIMIT 10")
    rows = cursor.fetchall()
    conn.close()
    return rows
