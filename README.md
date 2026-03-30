# -postavki-apteka
Управление поставок в аптечный пункт 
import tkinter as tk
from tkinter import ttk, messagebox, simpledialog
import json
import os
from datetime import datetime, date
import hashlib
# ───────────────────────────── DATA LAYER ──────────────────────────────
DATA_FILE = "pharmacy_data.json"
ADMIN_USERS = {
"admin": hashlib.sha256("admin123".encode()).hexdigest(),
"manager": hashlib.sha256("manager456".encode()).hexdigest(),
}
DEFAULT_DATA = {
"supplies": [
{
"id": 1,
"name": "Аспирин",
"supplier": "ФармаГрупп",
"category": "Анальгетики",
"quantity": 150,
"unit": "уп.",
"price": 45.00,
"delivery_date": "2025-02-01",
"expiry_date": "2027-01-01",
"status": "Доставлено",
},
{
"id": 2,
"name": "Амоксициллин 500мг",
"supplier": "МедСинтез",
"category": "Антибиотики",
"quantity": 80,
"unit": "уп.",
"price": 210.50,
"delivery_date": "2025-02-05",
"expiry_date": "2026-06-01",
"status": "Доставлено",
},
{
"id": 3,
"name": "Витамин C 1000мг",
"supplier": "ВитаФарм",
"category": "Витамины",
"quantity": 0,
"unit": "уп.",
"price": 89.90,
"delivery_date": "2025-02-10",
"expiry_date": "2026-12-01",
"status": "В пути",
},
{
"id": 4,
"name": "Но-Шпа 40мг",
"supplier": "ФармаГрупп",
"category": "Спазмолитики",
"quantity": 200,
"unit": "уп.",
"price": 175.00,
"delivery_date": "2025-01-20",
"expiry_date": "2027-03-01",
"status": "Доставлено",
},
{
"id": 5,
"name": "Омепразол 20мг",
"supplier": "ГастроФарм",
"category": "Гастроэнтерология",
"quantity": 45,
"unit": "уп.",
"price": 130.00,
"delivery_date": "2025-02-12",
"expiry_date": "2026-09-01",
"status": "Ожидается",
},
],
"next_id": 6,
}
CATEGORIES = [
"Анальгетики", "Антибиотики", "Витамины", "Спазмолитики",
"Гастроэнтерология", "Кардиология", "Неврология", "Прочее",
]
STATUSES = ["Ожидается", "В пути", "Доставлено", "Отменено"]
UNITS = ["уп.", "шт.", "фл.", "амп.", "кг", "л"]
def load_data():
if os.path.exists(DATA_FILE):
with open(DATA_FILE, "r", encoding="utf-8") as f:
return json.load(f)
return DEFAULT_DATA.copy()
def save_data(data):
with open(DATA_FILE, "w", encoding="utf-8") as f:
json.dump(data, f, ensure_ascii=False, indent=2)
def hash_password(password):
return hashlib.sha256(password.encode()).hexdigest()
# ─────────────────────────── COLORS & FONTS ────────────────────────────
BG_DARK = "#0f1923"
BG_PANEL = "#16232e"
BG_CARD = "#1c2e3d"
BG_ROW_ODD = "#192533"
BG_ROW_EVEN = "#1c2e3d"
ACCENT = "#00b4d8"
ACCENT2 = "#48cae4"
SUCCESS = "#2dc653"
WARNING = "#f4a261"
DANGER = "#e63946"
TEXT_MAIN = "#e0f0ff"
TEXT_MUTED = "#7ea8be"
TEXT_HEADER = "#ffffff"
BORDER = "#2a4a5e"
FONT_TITLE = ("Segoe UI", 18, "bold")
FONT_HEADER = ("Segoe UI", 12, "bold")
FONT_BODY = ("Segoe UI", 10)
FONT_SMALL = ("Segoe UI", 9)
FONT_BTN = ("Segoe UI", 10, "bold")
FONT_MONO = ("Consolas", 10)
# ─────────────────────────── MAIN APP ──────────────────────────────────
class PharmacyApp(tk.Tk):
def __init__(self):
super().__init__()
self.title("Аптечный пункт — Управление поставками")
self.geometry("1200x720")
self.minsize(900, 600)
self.configure(bg=BG_DARK)
self.resizable(True, True)
self.role = None self.username = ""
self.data = load_data()
# "admin" | "user"
self._apply_style()
self._show_start_screen()
# ─────────── TTK STYLE ───────────
def _apply_style(self):
style = ttk.Style(self)
style.theme_use("clam")
style.configure(".", background=BG_DARK, foreground=TEXT_MAIN,
font=FONT_BODY, borderwidth=0)
style.configure("TFrame", background=BG_DARK)
style.configure("Card.TFrame", background=BG_CARD)
style.configure("Panel.TFrame", background=BG_PANEL)
style.configure("TLabel", background=BG_DARK, foreground=TEXT_MAIN, font=FONT_BODY)
style.configure("Header.TLabel", background=BG_PANEL, foreground=TEXT_HEADER, font=FO
style.configure("Muted.TLabel", background=BG_CARD, foreground=TEXT_MUTED, font=FONT_
style.configure("Title.TLabel", background=BG_DARK, foreground=ACCENT, font=FONT_TITL
style.configure("TEntry", fieldbackground=BG_CARD, foreground=TEXT_MAIN,
insertcolor=TEXT_MAIN, relief="flat", font=FONT_BODY)
style.map("TEntry", fieldbackground=[("focus", "#243d52")])
style.configure("TCombobox", fieldbackground=BG_CARD, foreground=TEXT_MAIN,
background=BG_CARD, selectbackground=ACCENT, font=FONT_BODY)
style.map("TCombobox", fieldbackground=[("readonly", BG_CARD)])
style.configure("Accent.TButton", background=ACCENT, foreground=BG_DARK,
font=FONT_BTN, padding=(14, 7), relief="flat")
style.map("Accent.TButton",
background=[("active", ACCENT2), ("pressed", "#0090b0")],
foreground=[("active", BG_DARK)])
style.configure("Success.TButton", background=SUCCESS, foreground=BG_DARK,
font=FONT_BTN, padding=(12, 6), relief="flat")
style.map("Success.TButton", background=[("active", "#3de66a")])
style.configure("Danger.TButton", background=DANGER, foreground=TEXT_HEADER,
font=FONT_BTN, padding=(12, 6), relief="flat")
style.map("Danger.TButton", background=[("active", "#ff4d58")])
style.configure("Ghost.TButton", background=BG_PANEL, foreground=TEXT_MUTED,
font=FONT_BODY, padding=(12, 6), relief="flat")
style.map("Ghost.TButton", background=[("active", BG_CARD)],
foreground=[("active", TEXT_MAIN)])
# Treeview
style.configure("Treeview", background=BG_ROW_EVEN, foreground=TEXT_MAIN,
fieldbackground=BG_ROW_EVEN, rowheight=30,
font=FONT_BODY, borderwidth=0)
style.configure("Treeview.Heading", background=BG_PANEL, foreground=ACCENT,
font=FONT_HEADER, relief="flat")
style.map("Treeview", background=[("selected", ACCENT)],
foreground=[("selected", BG_DARK)])
style.map("Treeview.Heading", background=[("active", BG_CARD)])
style.configure("TScrollbar", background=BG_PANEL, troughcolor=BG_DARK,
arrowcolor=TEXT_MUTED, borderwidth=0)
style.configure("TSeparator", background=BORDER)
# ─────────── START SCREEN ───────────
def _show_start_screen(self):
self._clear()
frame = tk.Frame(self, bg=BG_DARK)
frame.place(relx=.5, rely=.5, anchor="center")
# Logo area
logo_frame = tk.Frame(frame, bg=BG_PANEL, bd=0)
logo_frame.pack(pady=(0, 30))
tk.Label(logo_frame, text="⚕", font=("Segoe UI", 48), bg=BG_PANEL,
fg=ACCENT).pack(padx=40, pady=(20, 5))
tk.Label(logo_frame, text="Аптечный пункт", font=("Segoe UI", 20, "bold"),
bg=BG_PANEL, fg=TEXT_HEADER).pack()
tk.Label(logo_frame, text="Система управления поставками", font=FONT_BODY,
bg=BG_PANEL, fg=TEXT_MUTED).pack(pady=(2, 20))
# Buttons
btn_frame = tk.Frame(frame, bg=BG_DARK)
btn_frame.pack(pady=10)
ttk.Button(btn_frame, text=" Войти как администратор ",
style="Accent.TButton",
command=self._show_login).pack(pady=8, ipadx=10)
ttk.Button(btn_frame, text=" Продолжить без авторизации ",
style="Ghost.TButton",
command=self._enter_as_user).pack(pady=4)
tk.Label(frame,
text="Вход без авторизации позволяет только просматривать данные",
font=FONT_SMALL, bg=BG_DARK, fg=TEXT_MUTED).pack(pady=(12, 0))
# ─────────── LOGIN ───────────
def _show_login(self):
dlg = tk.Toplevel(self)
dlg.title("Вход для администратора")
dlg.geometry("380x300")
dlg.resizable(False, False)
dlg.configure(bg=BG_PANEL)
dlg.grab_set()
dlg.transient(self)
tk.Label(dlg, text="Авторизация", font=FONT_TITLE,
bg=BG_PANEL, fg=ACCENT).pack(pady=(28, 4))
tk.Label(dlg, text="Введите учётные данные администратора",
font=FONT_SMALL, bg=BG_PANEL, fg=TEXT_MUTED).pack(pady=(0, 20))
form = tk.Frame(dlg, bg=BG_PANEL)
form.pack(padx=40, fill="x")
tk.Label(form, text="Логин", font=FONT_SMALL, bg=BG_PANEL,
fg=TEXT_MUTED).grid(row=0, column=0, sticky="w", pady=(0, 3))
login_var = tk.StringVar()
login_entry = tk.Entry(form, textvariable=login_var, font=FONT_BODY,
bg=BG_CARD, fg=TEXT_MAIN, insertbackground=TEXT_MAIN,
relief="flat", bd=6)
login_entry.grid(row=1, column=0, sticky="ew", pady=(0, 12), ipady=5)
tk.Label(form, text="Пароль", font=FONT_SMALL, bg=BG_PANEL,
fg=TEXT_MUTED).grid(row=2, column=0, sticky="w", pady=(0, 3))
pass_var = tk.StringVar()
pass_entry = tk.Entry(form, textvariable=pass_var, show="•", font=FONT_BODY,
bg=BG_CARD, fg=TEXT_MAIN, insertbackground=TEXT_MAIN,
relief="flat", bd=6)
pass_entry.grid(row=3, column=0, sticky="ew", ipady=5)
form.columnconfigure(0, weight=1)
err_label = tk.Label(dlg, text="", font=FONT_SMALL, bg=BG_PANEL, fg=DANGER)
err_label.pack(pady=6)
def attempt():
u = login_var.get().strip()
p = pass_var.get()
if u in ADMIN_USERS and ADMIN_USERS[u] == hash_password(p):
self.role = "admin"
self.username = u
dlg.destroy()
self._enter_main()
else:
err_label.config(text="Неверный логин или пароль")
pass_entry.delete(0, "end")
btn_row = tk.Frame(dlg, bg=BG_PANEL)
btn_row.pack(pady=8)
ttk.Button(btn_row, text="Войти", style="Accent.TButton",
command=attempt).pack(side="left", padx=6)
ttk.Button(btn_row, text="Отмена", style="Ghost.TButton",
command=dlg.destroy).pack(side="left")
login_entry.focus_set()
dlg.bind("<Return>", lambda e: attempt())
# Hint
tk.Label(dlg, text="Подсказка: admin / admin123", font=FONT_SMALL,
bg=BG_PANEL, fg=BORDER).pack(pady=(0, 8))
def _enter_as_user(self):
self.role = "user"
self.username = "Гость"
self._enter_main()
# ─────────── MAIN WINDOW ───────────
def _enter_main(self):
self._clear()
# Root grid
self.rowconfigure(0, weight=0)
self.rowconfigure(1, weight=1)
self.columnconfigure(0, weight=0)
self.columnconfigure(1, weight=1)
self._build_topbar()
self._build_sidebar()
self._build_content_area()
self._show_dashboard()
def _build_topbar(self):
bar = tk.Frame(self, bg=BG_PANEL, height=56)
bar.grid(row=0, column=0, columnspan=2, sticky="ew")
bar.grid_propagate(False)
tk.Label(bar, text="⚕ Аптечный пункт", font=("Segoe UI", 14, "bold"),
bg=BG_PANEL, fg=ACCENT).pack(side="left", padx=20)
role_text = " Администратор" if self.role == "admin" else " Гость"
role_color = WARNING if self.role == "admin" else TEXT_MUTED
tk.Label(bar, text=f" {self.username} • {role_text}",
font=FONT_SMALL, bg=BG_PANEL, fg=role_color).pack(side="right", padx=16)
ttk.Button(bar, text="Выйти", style="Ghost.TButton",
command=self._logout).pack(side="right", padx=4)
def _build_sidebar(self):
self.sidebar = tk.Frame(self, bg=BG_PANEL, width=190)
self.sidebar.grid(row=1, column=0, sticky="nsew")
self.sidebar.grid_propagate(False)
nav_items = [
(" ", "Дашборд", self._show_dashboard),
(" ", "Поставки", self._show_supplies),
(" ", "Поиск", self._show_search),
(" ", "Статистика", self._show_stats),
]
if self.role == "admin":
nav_items += [
(" ", "Добавить", self._show_add),
(" ", "Редактировать", self._edit_selected),
(" ", "Удалить", self._delete_selected),
]
self.nav_btns = {}
for icon, label, cmd in nav_items:
btn = tk.Button(
self.sidebar, text=f" {icon} {label}",
font=FONT_BODY, bg=BG_PANEL, fg=TEXT_MUTED,
activebackground=BG_CARD, activeforeground=TEXT_MAIN,
relief="flat", anchor="w", pady=12, cursor="hand2",
command=cmd,
)
btn.pack(fill="x", padx=8, pady=2)
self.nav_btns[label] = btn
tk.Frame(self.sidebar, bg=BORDER, height=1).pack(fill="x", padx=10, pady=10)
if self.role != "admin":
info = tk.Label(self.sidebar,
text=" Режим просмотра\n Авторизуйтесь\n для изменений",
font=FONT_SMALL, bg=BG_PANEL, fg=TEXT_MUTED,
justify="left")
info.pack(anchor="w", padx=10)
def _build_content_area(self):
self.content = tk.Frame(self, bg=BG_DARK)
self.content.grid(row=1, column=1, sticky="nsew", padx=12, pady=12)
self.content.rowconfigure(0, weight=1)
self.content.columnconfigure(0, weight=1)
def _clear_content(self):
for w in self.content.winfo_children():
w.destroy()
def _set_active_nav(self, label):
for lbl, btn in self.nav_btns.items():
if lbl == label:
btn.config(bg=BG_CARD, fg=ACCENT, font=FONT_BTN)
else:
btn.config(bg=BG_PANEL, fg=TEXT_MUTED, font=FONT_BODY)
# ─────────── DASHBOARD ───────────
def _show_dashboard(self):
self._set_active_nav("Дашборд")
self._clear_content()
supplies = self.data["supplies"]
frame = tk.Frame(self.content, bg=BG_DARK)
frame.pack(fill="both", expand=True)
tk.Label(frame, text="Дашборд", font=FONT_TITLE,
bg=BG_DARK, fg=TEXT_HEADER).pack(anchor="w", pady=(0, 16))
# KPI cards
total = len(supplies)
delivered = sum(1 for s in supplies if s["status"] == "Доставлено")
on_way = sum(1 for s in supplies if s["status"] == "В пути")
expected = sum(1 for s in supplies if s["status"] == "Ожидается")
out_of_stock = sum(1 for s in supplies if s["quantity"] == 0)
cards_frame = tk.Frame(frame, bg=BG_DARK)
cards_frame.pack(fill="x")
self._kpi_card(cards_frame, " Всего поставок", str(total), ACCENT)
self._kpi_card(cards_frame, " Доставлено", str(delivered), SUCCESS)
self._kpi_card(cards_frame, " В пути", str(on_way), WARNING)
self._kpi_card(cards_frame, " Ожидается", str(expected), ACCENT2)
self._kpi_card(cards_frame, " Нет на складе", str(out_of_stock), DANGER)
# Recent deliveries table
tk.Label(frame, text="Последние поставки", font=FONT_HEADER,
bg=BG_DARK, fg=TEXT_HEADER).pack(anchor="w", pady=(20, 8))
recent = sorted(supplies, key=lambda x: x["delivery_date"], reverse=True)[:5]
self._small_table(frame, recent)
def _kpi_card(self, parent, title, value, color):
card = tk.Frame(parent, bg=BG_CARD, bd=0)
card.pack(side="left", padx=6, pady=4, ipadx=16, ipady=12, expand=True, fill="x")
tk.Label(card, text=value, font=("Segoe UI", 26, "bold"),
bg=BG_CARD, fg=color).pack()
tk.Label(card, text=title, font=FONT_SMALL,
bg=BG_CARD, fg=TEXT_MUTED).pack()
def _small_table(self, parent, rows):
cols = ("Название", "Поставщик", "Кол-во", "Статус", "Дата")
tree = ttk.Treeview(parent, columns=cols, show="headings", height=5)
widths = [200, 140, 70, 110, 100]
for col, w in zip(cols, widths):
tree.heading(col, text=col)
tree.column(col, width=w, anchor="center" if col != "Название" else "w")
for s in rows:
status_tag = {"Доставлено": "ok", "В пути": "warn",
"Ожидается": "info", "Отменено": "err"}.get(s["status"], "")
tree.insert("", "end",
values=(s["name"], s["supplier"], f'{s["quantity"]} {s["unit"]}',
s["status"], s["delivery_date"]),
tags=(status_tag,))
tree.tag_configure("ok", foreground=SUCCESS)
tree.tag_configure("warn", foreground=WARNING)
tree.tag_configure("info", foreground=ACCENT2)
tree.tag_configure("err", foreground=DANGER)
tree.pack(fill="x")
# ─────────── SUPPLIES TABLE ───────────
def _show_supplies(self):
self._set_active_nav("Поставки")
self._clear_content()
frame = tk.Frame(self.content, bg=BG_DARK)
frame.pack(fill="both", expand=True)
header = tk.Frame(frame, bg=BG_DARK)
header.pack(fill="x", pady=(0, 10))
tk.Label(header, text="Все поставки", font=FONT_TITLE,
bg=BG_DARK, fg=TEXT_HEADER).pack(side="left")
tk.Label(header, text=f" ({len(self.data['supplies'])} записей)",
font=FONT_BODY, bg=BG_DARK, fg=TEXT_MUTED).pack(side="left", pady=8)
if self.role == "admin":
ttk.Button(header, text="+ Добавить", style="Success.TButton",
command=self._show_add).pack(side="right", padx=4)
ttk.Button(header, text=" Удалить", style="Danger.TButton",
command=self._delete_selected).pack(side="right", padx=4)
ttk.Button(header, text="✏ Изменить", style="Accent.TButton",
command=self._edit_selected).pack(side="right", padx=4)
self.tree_frame = tk.Frame(frame, bg=BG_DARK)
self.tree_frame.pack(fill="both", expand=True)
self._build_tree(self.tree_frame, self.data["supplies"])
def _build_tree(self, parent, rows):
for w in parent.winfo_children():
w.destroy()
cols = ("ID", "Название", "Категория", "Поставщик", "Кол-во",
"Цена", "Дата поставки", "Срок годн.", "Статус")
self.tree = ttk.Treeview(parent, columns=cols, show="headings", selectmode="browse")
widths = [40, 200, 130, 140, 80, 80, 110, 110, 100]
for col, w in zip(cols, widths):
self.tree.heading(col, text=col,
command=lambda c=col: self._sort_tree(c))
anchor = "center" if col not in ("Название", "Поставщик", "Категория") else "w"
self.tree.column(col, width=w, anchor=anchor, minwidth=w - 10)
vsb = ttk.Scrollbar(parent, orient="vertical", command=self.tree.yview)
hsb = ttk.Scrollbar(parent, orient="horizontal", command=self.tree.xview)
self.tree.configure(yscrollcommand=vsb.set, xscrollcommand=hsb.set)
self.tree.grid(row=0, column=0, sticky="nsew")
vsb.grid(row=0, column=1, sticky="ns")
hsb.grid(row=1, column=0, sticky="ew")
parent.rowconfigure(0, weight=1)
parent.columnconfigure(0, weight=1)
self._populate_tree(rows)
if self.role == "admin":
self.tree.bind("<Double-1>", lambda e: self._edit_selected())
def _populate_tree(self, rows=None):
if rows is None:
rows = self.data["supplies"]
self.tree.delete(*self.tree.get_children())
for i, s in enumerate(rows):
tag = "odd" if i % 2 else "even"
status_tag = {"Доставлено": "ok", "В пути": "warn",
"Ожидается": "info", "Отменено": "err"}.get(s["status"], "")
self.tree.insert("", "end", iid=str(s["id"]),
values=(
s["id"], s["name"], s["category"], s["supplier"],
f'{s["quantity"]} {s["unit"]}',
f'₽{s["price"]:.2f}',
s["delivery_date"], s["expiry_date"], s["status"]
),
tags=(tag, status_tag))
self.tree.tag_configure("odd", background=BG_ROW_ODD)
self.tree.tag_configure("even", background=BG_ROW_EVEN)
self.tree.tag_configure("ok", foreground=SUCCESS)
self.tree.tag_configure("warn", foreground=WARNING)
self.tree.tag_configure("info", foreground=ACCENT2)
self.tree.tag_configure("err", foreground=DANGER)
_sort_reverse = {}
def _sort_tree(self, col):
col_map = {
"ID": "id", "Название": "name", "Категория": "category",
"Поставщик": "supplier", "Цена": "price",
"Дата поставки": "delivery_date", "Срок годн.": "expiry_date",
"Статус": "status",
}
key = col_map.get(col, "id")
rev = self._sort_reverse.get(col, False)
self._sort_reverse[col] = not rev
rows = sorted(self.data["supplies"],
key=lambda x: str(x.get(key, "")), reverse=rev)
self._populate_tree(rows)
# ─────────── SEARCH ───────────
def _show_search(self):
self._set_active_nav("Поиск")
self._clear_content()
frame = tk.Frame(self.content, bg=BG_DARK)
frame.pack(fill="both", expand=True)
tk.Label(frame, text="Поиск поставок", font=FONT_TITLE,
bg=BG_DARK, fg=TEXT_HEADER).pack(anchor="w", pady=(0, 12))
# Filters
filter_frame = tk.Frame(frame, bg=BG_CARD)
filter_frame.pack(fill="x", pady=(0, 12), ipadx=10, ipady=10)
def lbl(parent, text):
return tk.Label(parent, text=text, font=FONT_SMALL,
bg=BG_CARD, fg=TEXT_MUTED)
r1 = tk.Frame(filter_frame, bg=BG_CARD)
r1.pack(fill="x", padx=10, pady=4)
lbl(r1, "Поиск:").pack(side="left")
self.search_var = tk.StringVar()
tk.Entry(r1, textvariable=self.search_var, font=FONT_BODY,
bg=BG_PANEL, fg=TEXT_MAIN, insertbackground=TEXT_MAIN,
relief="flat", bd=6, width=30).pack(side="left", padx=8, ipady=4)
lbl(r1, "Категория:").pack(side="left", padx=(12, 0))
self.cat_var = tk.StringVar(value="Все")
cat_combo = ttk.Combobox(r1, textvariable=self.cat_var,
values=["Все"] + CATEGORIES, state="readonly", width=16)
cat_combo.pack(side="left", padx=6)
lbl(r1, "Статус:").pack(side="left", padx=(12, 0))
self.status_var = tk.StringVar(value="Все")
st_combo = ttk.Combobox(r1, textvariable=self.status_var,
values=["Все"] + STATUSES, state="readonly", width=14)
st_combo.pack(side="left", padx=6)
ttk.Button(r1, text="Найти", style="Accent.TButton",
command=self._do_search).pack(side="left", padx=10)
ttk.Button(r1, text="Сбросить", style="Ghost.TButton",
command=self._reset_search).pack(side="left")
self.search_tree_frame = tk.Frame(frame, bg=BG_DARK)
self.search_tree_frame.pack(fill="both", expand=True)
self.tree = None
self._build_tree(self.search_tree_frame, self.data["supplies"])
self.search_var.bind("<Return>", lambda e: self._do_search())
def _do_search(self):
q = self.search_var.get().lower()
cat = self.cat_var.get()
status = self.status_var.get()
results = []
for s in self.data["supplies"]:
if q and q not in s["name"].lower() and q not in s["supplier"].lower():
continue
if cat != "Все" and s["category"] != cat:
continue
if status != "Все" and s["status"] != status:
continue
results.append(s)
self._build_tree(self.search_tree_frame, results)
def _reset_search(self):
self.search_var.set("")
self.cat_var.set("Все")
self.status_var.set("Все")
self._build_tree(self.search_tree_frame, self.data["supplies"])
# ─────────── STATS ───────────
def _show_stats(self):
self._set_active_nav("Статистика")
self._clear_content()
frame = tk.Frame(self.content, bg=BG_DARK)
frame.pack(fill="both", expand=True)
tk.Label(frame, text="Статистика", font=FONT_TITLE,
bg=BG_DARK, fg=TEXT_HEADER).pack(anchor="w", pady=(0, 16))
supplies = self.data["supplies"]
total_val = sum(s["price"] * s["quantity"] for s in supplies)
# Summary cards
top = tk.Frame(frame, bg=BG_DARK)
top.pack(fill="x", pady=(0, 16))
self._kpi_card(top, " Сумма запаса", f"₽{total_val:,.2f}", SUCCESS)
self._kpi_card(top, " Наименований", str(len(supplies)), ACCENT)
avg_price = sum(s["price"] for s in supplies) / max(len(supplies), 1)
self._kpi_card(top, " Средняя цена", f"₽{avg_price:.2f}", ACCENT2)
# By category
cat_frame = tk.Frame(frame, bg=BG_CARD)
cat_frame.pack(fill="x", pady=4, ipadx=10, ipady=10)
tk.Label(cat_frame, text="По категориям", font=FONT_HEADER,
bg=BG_CARD, fg=TEXT_HEADER).pack(anchor="w", padx=10, pady=6)
from collections import Counter
cats = Counter(s["category"] for s in supplies)
statuses = Counter(s["status"] for s in supplies)
cols_frame = tk.Frame(cat_frame, bg=BG_CARD)
cols_frame.pack(fill="x", padx=10)
def mini_stat(parent, title, items):
box = tk.Frame(parent, bg=BG_PANEL)
box.pack(side="left", fill="both", expand=True, padx=6, pady=4, ipadx=8, ipady=6)
tk.Label(box, text=title, font=FONT_HEADER, bg=BG_PANEL,
fg=ACCENT).pack(anchor="w", pady=(0, 6))
for name, count in sorted(items.items(), key=lambda x: -x[1]):
row = tk.Frame(box, bg=BG_PANEL)
row.pack(fill="x", pady=1)
tk.Label(row, text=name, font=FONT_BODY, bg=BG_PANEL,
fg=TEXT_MAIN, width=22, anchor="w").pack(side="left")
bar_w = max(int(count / max(items.values()) * 100), 4)
tk.Frame(row, bg=ACCENT, height=14, width=bar_w).pack(side="left", padx=4)
tk.Label(row, text=str(count), font=FONT_SMALL, bg=BG_PANEL,
fg=TEXT_MUTED).pack(side="left")
mini_stat(cols_frame, "Категории", dict(cats))
mini_stat(cols_frame, "Статусы", dict(statuses))
# ─────────── ADD / EDIT FORM ───────────
def _show_add(self, supply=None):
if self.role != "admin":
messagebox.showwarning("Нет доступа", "Только администратор может добавлять поста
return
editing = supply is not None
dlg = tk.Toplevel(self)
dlg.title("Редактировать поставку" if editing else "Новая поставка")
dlg.geometry("520x560")
dlg.resizable(False, False)
dlg.configure(bg=BG_PANEL)
dlg.grab_set()
dlg.transient(self)
tk.Label(dlg, text="Редактировать поставку" if editing else "Добавить поставку",
font=FONT_TITLE, bg=BG_PANEL, fg=ACCENT).pack(pady=(20, 4))
tk.Label(dlg, text="Заполните поля формы ниже",
font=FONT_SMALL, bg=BG_PANEL, fg=TEXT_MUTED).pack(pady=(0, 16))
form = tk.Frame(dlg, bg=BG_PANEL)
form.pack(padx=30, fill="both", expand=True)
fields = {}
def field(label, row, var_type="entry", options=None, default=""):
tk.Label(form, text=label, font=FONT_SMALL, bg=BG_PANEL,
fg=TEXT_MUTED).grid(row=row * 2, column=0, columnspan=2,
sticky="w", pady=(6, 2))
if var_type == "combo":
var = tk.StringVar(value=default)
w = ttk.Combobox(form, textvariable=var, values=options,
state="readonly", font=FONT_BODY, width=38)
else:
var = tk.StringVar(value=default)
w = tk.Entry(form, textvariable=var, font=FONT_BODY,
bg=BG_CARD, fg=TEXT_MAIN,
insertbackground=TEXT_MAIN, relief="flat", bd=6, width=40)
w.grid(row=row * 2 + 1, column=0, columnspan=2, sticky="ew", ipady=5)
return var
s = supply or {}
name_var = field("Название препарата *", 0, default=s.get("name", ""))
cat_var = field("Категория *", 1, "combo", CATEGORIES,
default=s.get("category", CATEGORIES[0]))
supplier_var = field("Поставщик *", 2, default=s.get("supplier", ""))
price_var = field("Цена (₽) *", 3, default=str(s.get("price", "")))
# Row: quantity + unit
tk.Label(form, text="Количество *", font=FONT_SMALL, bg=BG_PANEL,
fg=TEXT_MUTED).grid(row=8, column=0, sticky="w", pady=(6, 2))
tk.Label(form, text="Единица", font=FONT_SMALL, bg=BG_PANEL,
fg=TEXT_MUTED).grid(row=8, column=1, sticky="w", pady=(6, 2))
qty_var = tk.StringVar(value=str(s.get("quantity", "")))
unit_var = tk.StringVar(value=s.get("unit", UNITS[0]))
tk.Entry(form, textvariable=qty_var, font=FONT_BODY,
bg=BG_CARD, fg=TEXT_MAIN, insertbackground=TEXT_MAIN,
relief="flat", bd=6, width=20).grid(row=9, column=0, sticky="ew",
padx=(0, 5), ipady=5)
ttk.Combobox(form, textvariable=unit_var, values=UNITS,
state="readonly", font=FONT_BODY, width=10).grid(row=9, column=1,
sticky="ew", ipady=2)
del_var = field("Дата поставки (ГГГГ-ММ-ДД) *", 5,
default=s.get("delivery_date", str(date.today())))
exp_var = field("Срок годности (ГГГГ-ММ-ДД)", 6, default=s.get("expiry_date", ""))
stat_var = field("Статус", 7, "combo", STATUSES,
default=s.get("status", STATUSES[0]))
form.columnconfigure(0, weight=1)
form.columnconfigure(1, weight=1)
err_lbl = tk.Label(dlg, text="", font=FONT_SMALL, bg=BG_PANEL, fg=DANGER)
err_lbl.pack()
def save():
nm = name_var.get().strip()
sup = supplier_var.get().strip()
try:
price = float(price_var.get())
qty = int(qty_var.get())
except ValueError:
err_lbl.config(text="Цена и количество должны быть числами")
return
if not nm or not sup:
err_lbl.config(text="Заполните обязательные поля (*)")
return
try:
datetime.strptime(del_var.get(), "%Y-%m-%d")
except ValueError:
err_lbl.config(text="Неверный формат даты поставки (ГГГГ-ММ-ДД)")
return
if editing:
s["name"] = nm
s["category"] = cat_var.get()
s["supplier"] = sup
s["price"] = price
s["quantity"] = qty
s["unit"] = unit_var.get()
s["delivery_date"] = del_var.get()
s["expiry_date"] = exp_var.get()
s["status"] = stat_var.get()
else:
new_supply = {
"id": self.data["next_id"],
"name": nm,
"category": cat_var.get(),
"supplier": sup,
"price": price,
"quantity": qty,
"unit": unit_var.get(),
"delivery_date": del_var.get(),
"expiry_date": exp_var.get(),
"status": stat_var.get(),
}
self.data["supplies"].append(new_supply)
self.data["next_id"] += 1
save_data(self.data)
dlg.destroy()
self._show_supplies()
btn_row = tk.Frame(dlg, bg=BG_PANEL)
btn_row.pack(pady=10)
ttk.Button(btn_row, text="Сохранить", style="Success.TButton",
command=save).pack(side="left", padx=6)
ttk.Button(btn_row, text="Отмена", style="Ghost.TButton",
command=dlg.destroy).pack(side="left")
# ─────────── EDIT SELECTED ───────────
def _edit_selected(self):
if self.role != "admin":
messagebox.showwarning("Нет доступа", "Только администратор может редактировать п
return
if not hasattr(self, "tree") or not self.tree:
messagebox.showinfo("Подсказка", "Сначала перейдите на вкладку «Поставки».")
return
sel = self.tree.selection()
if not sel:
messagebox.showinfo("Выбор", "Выберите строку для редактирования.")
return
sid = int(sel[0])
supply = next((s for s in self.data["supplies"] if s["id"] == sid), None)
if supply:
self._show_add(supply)
# ─────────── DELETE SELECTED ───────────
def _delete_selected(self):
if self.role != "admin":
messagebox.showwarning("Нет доступа", "Только администратор может удалять поставк
return
if not hasattr(self, "tree") or not self.tree:
messagebox.showinfo("Подсказка", "Сначала перейдите на вкладку «Поставки».")
return
sel = self.tree.selection()
if not sel:
messagebox.showinfo("Выбор", "Выберите строку для удаления.")
return
sid = int(sel[0])
supply = next((s for s in self.data["supplies"] if s["id"] == sid), None)
if supply:
if messagebox.askyesno("Подтверждение",
f"Удалить поставку «{supply['name']}»?"):
self.data["supplies"] = [s for s in self.data["supplies"]
if s["id"] != sid]
save_data(self.data)
self._show_supplies()
# ─────────── LOGOUT ───────────
def _logout(self):
self.role = None
self.username = ""
self.tree = None
self._clear()
self._show_start_screen()
def _clear(self):
for w in self.winfo_children():
w.destroy()
# ─────────────────────────── ENTRY POINT ───────────────────────────────
if __name__ == "__main__":
app = PharmacyApp()
app.mainloop()
