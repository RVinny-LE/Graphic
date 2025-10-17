import tkinter as tk
from tkinter import ttk
import colorsys
import time

# Глобальные переменные для параметров системы Лоренца
sigma = 10.0      # Параметр сигма (σ)
r = 28.0          # Параметр r (ρ) - критическое значение для хаоса
b = 8.0/3.0       # Параметр b (β)
x, y, z = 0.0, 1.0, 1.05  # Более интересные начальные условия
scale = 10        # Масштаб для отображения
dt = 0.02         # Увеличенный шаг времени для более быстрого движения
trail = []        # Список для хранения координат
hue = 0           # Текущее значение оттенка цвета (0-1)
line_id = None    # Идентификатор линии на canvas

def lorenz_step():
    """Вычисление одного шага системы уравнений Лоренца"""
    global x, y, z
    # Классические уравнения Лоренца
    dx = sigma * (y - x)
    dy = x * (r - z) - y
    dz = x * y - b * z
    
    # Интегрирование методом Эйлера
    x += dx * dt
    y += dy * dt
    z += dz * dt
    return x, y, z

def hsv_to_hex(h, s=1.0, v=1.0):
    """Преобразование цвета из HSV в HEX формат"""
    r, g, b = colorsys.hsv_to_rgb(h, s, v)
    return f'#{int(r*255):02x}{int(g*255):02x}{int(b*255):02x}'

def update_animation():
    """Основная функция анимации"""
    global trail, hue, line_id
    
    # Вычисляем новые координаты точки
    x, y, z = lorenz_step()
    
    # Получаем текущие размеры canvas
    canvas_width = canvas.winfo_width()
    canvas_height = canvas.winfo_height()
    
    # Преобразование 3D координат в 2D экранные координаты
    # Используем изометрическую проекцию для лучшей визуализации
    screen_x = canvas_width / 2 + (x - y) * scale * 0.7
    screen_y = canvas_height / 2 - (x + y) * 0.5 * scale * 0.7 - z * scale * 0.3
    
    # Добавляем новую точку в траекторию
    trail.append((screen_x, screen_y))
    
    # Ограничиваем длину траектории
    max_trail_length = 2000
    if len(trail) > max_trail_length:
        trail = trail[-max_trail_length:]
    
    # Плавно изменяем оттенок цвета
    hue = (hue + 0.002) % 1.0
    color = hsv_to_hex(hue)
    
    # Обновляем линию
    if len(trail) > 2:
        if line_id is None:
            # Создаем линию в первый раз
            line_id = canvas.create_line(trail, fill=color, width=2, tags="trail", smooth=True)
        else:
            # Обновляем существующую линию
            canvas.coords(line_id, *sum(trail, ()))
            canvas.itemconfig(line_id, fill=color)
    
    # Планируем следующий кадр анимации
    root.after(10, update_animation)

def reset_simulation():
    """Сброс simulation к начальному состоянию"""
    global x, y, z, trail, hue, line_id
    x, y, z = 0.0, 1.0, 1.05  # Возвращаем начальные координаты
    trail = []               # Очищаем траекторию
    hue = 0                  # Сбрасываем цвет
    if line_id:
        canvas.delete(line_id)
        line_id = None

def update_params(_=None):
    """Обновление параметров системы"""
    global sigma, r, b
    sigma = scale_sigma.get()
    r = scale_r.get()
    b = scale_b.get()
    entry_sigma.delete(0, tk.END)
    entry_sigma.insert(0, f"{sigma:.2f}")
    entry_r.delete(0, tk.END)
    entry_r.insert(0, f"{r:.2f}")
    entry_b.delete(0, tk.END)
    entry_b.insert(0, f"{b:.2f}")
    reset_simulation()

def update_from_entry(event=None):
    """Обновление параметров из текстовых полей"""
    global sigma, r, b
    try:
        sigma_val = float(entry_sigma.get())
        r_val = float(entry_r.get())
        b_val = float(entry_b.get())
        if 0.1 <= sigma_val <= 50.0 and 0.1 <= r_val <= 100.0 and 0.1 <= b_val <= 10.0:
            sigma = sigma_val
            r = r_val
            b = b_val
            scale_sigma.set(sigma)
            scale_r.set(r)
            scale_b.set(b)
            reset_simulation()
    except ValueError:
        pass

def on_resize(event):
    """Обработчик изменения размера окна"""
    # При изменении размера перерисовываем линию с новыми координатами
    if line_id and trail:
        canvas.coords(line_id, *sum(trail, ()))

# Создание главного окна
root = tk.Tk()
root.title("Аттрактор Лоренца с градиентом")
root.geometry("800x600")
# Создание canvas
canvas = tk.Canvas(root, bg="black")
canvas.pack(fill=tk.BOTH, expand=True)
canvas.bind("<Configure>", on_resize)

# Фрейм для элементов управления
frame = ttk.Frame(root)
frame.pack(pady=10)

# Элементы управления для параметров
ttk.Label(frame, text="σ (Sigma):").grid(row=0, column=0, padx=5, pady=2)
scale_sigma = tk.Scale(frame, from_=0.1, to=50.0, resolution=0.1, orient=tk.HORIZONTAL, command=update_params, length=200)
scale_sigma.set(sigma)
scale_sigma.grid(row=0, column=1, padx=5, pady=2)
entry_sigma = ttk.Entry(frame, width=8)
entry_sigma.insert(0, f"{sigma:.2f}")
entry_sigma.grid(row=0, column=2, padx=5, pady=2)
entry_sigma.bind("<Return>", update_from_entry)

ttk.Label(frame, text="r (Rho):").grid(row=1, column=0, padx=5, pady=2)
scale_r = tk.Scale(frame, from_=0.1, to=100.0, resolution=0.1, orient=tk.HORIZONTAL, command=update_params, length=200)
scale_r.set(r)
scale_r.grid(row=1, column=1, padx=5, pady=2)
entry_r = ttk.Entry(frame, width=8)
entry_r.insert(0, f"{r:.2f}")
entry_r.grid(row=1, column=2, padx=5, pady=2)
entry_r.bind("<Return>", update_from_entry)

ttk.Label(frame, text="b (Beta):").grid(row=2, column=0, padx=5, pady=2)
scale_b = tk.Scale(frame, from_=0.1, to=10.0, resolution=0.1, orient=tk.HORIZONTAL, command=update_params, length=200)
scale_b.set(b)
scale_b.grid(row=2, column=1, padx=5, pady=2)
entry_b = ttk.Entry(frame, width=8)
entry_b.insert(0, f"{b:.2f}")
entry_b.grid(row=2, column=2, padx=5, pady=2)
entry_b.bind("<Return>", update_from_entry)

# Кнопка сброса
reset_btn = ttk.Button(frame, text="Сброс", command=reset_simulation)
reset_btn.grid(row=3, columnspan=3, pady=10)

# Добавим информационную метку
info_label = ttk.Label(root, text="Аттрактор Лоренца - классическая хаотическая система")
info_label.pack(pady=5)

# Запуск анимации
root.after(100, update_animation)

# Запуск приложения
root.mainloop()
