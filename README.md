# Text-editor-and-random-password-generator-
import secrets
import string
import tkinter as tk
from tkinter import messagebox, scrolledtext, ttk, colorchooser, simpledialog
import pyperclip
import sqlite3
from fpdf import FPDF



def change_font():
    selected_font = font_combo.get()
    selected_size = size_combo.get()
    text_editor.configure(font=(selected_font, int(selected_size)))


def switch_window(window_name):
    if window_name == "password":
        text_editor_frame.pack_forget()
        password_frame.pack(fill=tk.BOTH, expand=True, padx=20, pady=20)
    else:
        password_frame.pack_forget()
        text_editor_frame.pack(fill=tk.BOTH, expand=True, padx=20, pady=20)



# Password Generator Functions
def generate_password():
    length = int(length_var.get())
    use_digits = digits_var.get()
    use_special = special_var.get()
    alphabet = string.ascii_letters
    if use_digits:
        alphabet += string.digits
    if use_special:
        alphabet += string.punctuation
    password = ''.join(secrets.choice(alphabet) for _ in range(length))
    password_var.set(password)

"""def apply_toggle_theme():
    global dark_mode
    dark_mode = not dark_mode
    new_theme = "#2E2E2E" if dark_mode else "#303F9F"
    text_color = "#FFFFFF" if dark_mode else "#000000"
    button_bg = "#555555" if dark_mode else "#FFEB3B"
    app.configure(bg=new_theme)
    main_frame.configure(bg=new_theme)
    menu_frame.configure(bg=button_bg)
    for widget in main_frame.winfo_children():
        widget.configure(bg=new_theme, fg=text_color)
    for widget in menu_frame.winfo_children():
        widget.configure(bg=button_bg, fg=text_color)
    for widget in password_frame.winfo_children():
        widget.configure(bg=new_theme, fg=text_color)
    for widget in text_editor_frame.winfo_children():
        widget.configure(bg=new_theme, fg=text_color)
    text_editor.configure(bg=new_theme, fg=text_color, insertbackground=text_color)

"""

def apply_toggle_theme():
    global dark_mode
    dark_mode = not dark_mode
    new_theme = "#2E2E2E" if dark_mode else "#303F9F"
    text_color = "#FFFFFF" if dark_mode else "#000000"
    button_bg = "#555555" if dark_mode else "#FFEC3B"

    app.configure(bg=new_theme)
    main_frame.configure(bg=new_theme)
    menu_frame.configure(bg=button_bg)



    def update_widget(widget):
        """Apply theme changes only to widgets that support fg"""
        try:
            widget.configure(bg=new_theme)
            if isinstance(widget, (tk.Label, tk.Button, tk.Entry, tk.Text, tk.Checkbutton)):
                widget.configure(fg=text_color)
        except tk.TclError:
            pass  # Ignore widgets that don't support fg

    # Apply theme changes to widgets
    for frame in [main_frame, menu_frame, password_frame, text_editor_frame]:
        for widget in frame.winfo_children():
            update_widget(widget)

    text_editor.configure(bg=new_theme, fg=text_color, insertbackground=text_color)


def edit_password_entry():
    password_entry.configure(state="normal")
    messagebox.showinfo("Edit", "You can now edit the password.")


def choose_text_color():
    color_code = colorchooser.askcolor()[1]  # Extract the hex color
    if color_code:
        text_color_var.set(color_code)
        text_editor.configure(fg=color_code)  # Directly update the text color

def copy_to_clipboard():
    pyperclip.copy(password_var.get())
    messagebox.showinfo("Success", "Password copied to clipboard!")

def save_password():
    conn = sqlite3.connect("passwords.db")
    cursor = conn.cursor()
    cursor.execute("""
    CREATE TABLE IF NOT EXISTS passwords (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT,
        password TEXT
    )
    """)
    try:
        cursor.execute("INSERT INTO passwords (name, password) VALUES (?, ?)",
                       (name_var.get(), password_var.get()))
        conn.commit()
        messagebox.showinfo("Success", "Password saved securely in database!")
    except sqlite3.OperationalError as e:
        messagebox.showerror("Database Error", f"Error: {e}")
    finally:
        conn.close()

# Text Editor Functions
def save_to_txt():
    text_content = text_editor.get("1.0", tk.END)
    if text_content.strip():
        with open("text_editor_content.txt", "w") as file:
            file.write(text_content)
        messagebox.showinfo("Success", "Text saved to text_editor_content.txt")
    else:
        messagebox.showwarning("Warning", "No content to save!")

def save_to_pdf():
    text_content = text_editor.get("1.0", tk.END)
    if text_content.strip():
        pdf = FPDF()
        pdf.add_page()
        pdf.set_font("Arial", size=12)
        pdf.multi_cell(0, 10, text_content)
        pdf_file = "text_editor_content.pdf"
        pdf.output(pdf_file)
        messagebox.showinfo("Success", f"Text saved to {pdf_file}")
    else:
        messagebox.showwarning("Warning", "No content to save!")

def toggle_bold():
    try:
        if text_editor.tag_ranges("sel"):  # Check if any text is selected
            current_tags = text_editor.tag_names("sel.first")
            if "bold" in current_tags:
                text_editor.tag_remove("bold", "sel.first", "sel.last")
            else:
                text_editor.tag_add("bold", "sel.first", "sel.last")
                text_editor.tag_configure("bold", font=(font_combo.get(), int(size_combo.get()), "bold"))
    except tk.TclError:
        messagebox.showwarning("Warning", "Please select text to apply formatting.")
def toggle_italic():
    try:
        if text_editor.tag_ranges("sel"):
            current_tags = text_editor.tag_names("sel.first")
            if "italic" in current_tags:
                text_editor.tag_remove("italic", "sel.first", "sel.last")
            else:
                text_editor.tag_add("italic", "sel.first", "sel.last")
                text_editor.tag_configure("italic", font=(font_combo.get(), int(size_combo.get()), "italic"))
    except tk.TclError:
        messagebox.showwarning("Warning", "Please select text to apply formatting.")


def change_font():
    selected_font = font_combo.get()
    selected_size = size_combo.get()
    selected_color = text_color_var.get()

    if selected_font and selected_size:
        text_editor.configure(font=(selected_font, int(selected_size)))

    if selected_color:
        text_editor.configure(fg=selected_color)


def toggle_underline():
    try:
        if text_editor.tag_ranges("sel"):
            current_tags = text_editor.tag_names("sel.first")
            if "underline" in current_tags:
                text_editor.tag_remove("underline", "sel.first", "sel.last")
            else:
                text_editor.tag_add("underline", "sel.first", "sel.last")
                text_editor.tag_configure("underline", underline=True)
    except tk.TclError:
        messagebox.showwarning("Warning", "Please select text to apply formatting.")


def align_text(alignment):
    text_editor.tag_configure("align", justify=alignment)
    text_editor.tag_add("align", "1.0", "end")

def clear_text():
    text_editor.delete("1.0", tk.END)

def undo():
    try:
        text_editor.edit_undo()
    except tk.TclError:
        pass

def redo():
    try:
        text_editor.edit_redo()
    except tk.TclError:
        pass

def update_line_numbers(event=None):
    line_numbers.config(state="normal")
    line_numbers.delete("1.0", "end")
    line_count = text_editor.get("1.0", "end").count("\n") + 1
    line_numbers.insert("end", "\n".join(str(i) for i in range(1, line_count + 1)))
    line_numbers.config(state="disabled")

def update_status_bar(event=None):
    text_content = text_editor.get("1.0", "end-1c")
    words = len(text_content.split())
    characters = len(text_content)
    status_bar.config(text=f"Words: {words} | Characters: {characters}")

def find_and_replace():
    find_text = simpledialog.askstring("Find", "Enter text to find:")
    if find_text:
        replace_text = simpledialog.askstring("Replace", "Enter text to replace with:")
        if replace_text:
            text_content = text_editor.get("1.0", "end")
            new_content = text_content.replace(find_text, replace_text)
            text_editor.delete("1.0", "end")
            text_editor.insert("1.0", new_content)

# GUI Setup
app = tk.Tk()
app.title("Secure Password Generator & Editor")
app.geometry("1000x700")
app.configure(bg="#303F9F")

dark_mode = False

main_frame = tk.Frame(app, bg="#303F9F")
main_frame.pack(fill=tk.BOTH, expand=True)

menu_frame = tk.Frame(main_frame, bg="#FFEB3B", height=50)
menu_frame.pack(fill=tk.X)

password_button = tk.Button(menu_frame, text="Password Generator", command=lambda: switch_window("password"), bg="#4CAF50", fg="white", font=("Arial", 14))
password_button.pack(side=tk.LEFT, padx=15, pady=5)
text_editor_button = tk.Button(menu_frame, text="Text Editor", command=lambda: switch_window("editor"), bg="#795548", fg="white", font=("Arial", 14))
text_editor_button.pack(side=tk.LEFT, padx=15, pady=5)

theme_button = tk.Button(menu_frame, text="Toggle Theme", command=apply_toggle_theme, bg="#9C27B0", fg="white", font=("Arial", 14))
theme_button.pack(side=tk.RIGHT, padx=15, pady=5)

# Password Generator Frame
password_frame = tk.Frame(main_frame, bg="#303F9F")
password_frame.pack(fill=tk.BOTH, expand=True, padx=20, pady=20)

name_label = tk.Label(password_frame, text="Enter Name:", bg="#303F9F", fg="white", font=("Arial", 14, "bold"))
name_label.grid(row=0, column=0, padx=10, pady=10)
name_var = tk.StringVar()
name_entry = tk.Entry(password_frame, textvariable=name_var, width=20, font=("Arial", 14))
name_entry.grid(row=0, column=1, padx=10, pady=10)

label = tk.Label(password_frame, text="Enter Password Length:", bg="#303F9F", fg="white", font=("Arial", 14, "bold"))
label.grid(row=1, column=0, padx=10, pady=10)
length_var = tk.StringVar(value="12")
entry = tk.Entry(password_frame, textvariable=length_var, width=5, font=("Arial", 14))
entry.grid(row=1, column=1, padx=10, pady=10)

digits_var = tk.BooleanVar(value=True)
special_var = tk.BooleanVar(value=True)

digits_check = tk.Checkbutton(password_frame, text="Digits", variable=digits_var, bg="#303F9F", fg="white", font=("Arial", 14))
digits_check.grid(row=2, column=0, padx=10, pady=10)
special_check = tk.Checkbutton(password_frame, text="Special Characters", variable=special_var, bg="#303F9F", fg="white", font=("Arial", 14))
special_check.grid(row=2, column=1, padx=10, pady=10)

gen_button = tk.Button(password_frame, text="Generate", command=generate_password, bg="#4CAF50", fg="white", font=("Arial", 14, "bold"))
gen_button.grid(row=3, column=0, columnspan=2, pady=10)

password_var = tk.StringVar()
password_entry = tk.Entry(password_frame, textvariable=password_var, width=30, font=("Arial", 14))
password_entry.grid(row=4, column=0, columnspan=2, padx=10, pady=10)
password_entry.bind("<Double-1>", lambda event: password_entry.configure(state="normal"))

edit_button = tk.Button(password_frame, text="Edit", command=edit_password_entry, bg="#FFC107", fg="black", font=("Arial", 14, "bold"))
edit_button.grid(row=5, column=0, pady=10)
copy_button = tk.Button(password_frame, text="Copy", command=copy_to_clipboard, bg="#2196F3", fg="white", font=("Arial", 14, "bold"))
copy_button.grid(row=5, column=1, pady=10)
save_button = tk.Button(password_frame, text="Save", command=save_password, bg="#FF5722", fg="white", font=("Arial", 14, "bold"))
save_button.grid(row=6, column=0, columnspan=2, pady=10)

# Text Editor Frame
text_editor_frame = tk.Frame(main_frame, bg="#303F9F")
text_editor_frame.pack(fill=tk.BOTH, expand=True, padx=20, pady=20)

# Line Numbers
line_numbers = tk.Text(text_editor_frame, width=4, padx=5, pady=5, takefocus=0, border=0, background="#2E2E2E", foreground="#FFFFFF", state="disabled")
line_numbers.pack(side=tk.LEFT, fill=tk.Y)

text_editor = scrolledtext.ScrolledText(text_editor_frame, wrap=tk.WORD, height=15, font=("Arial", 16), undo=True)
text_editor.pack(fill=tk.BOTH, expand=True, padx=20, pady=20)

# Bind events for line numbers and status bar
text_editor.bind("<KeyRelease>", update_line_numbers)
text_editor.bind("<KeyRelease>", update_status_bar)

# Font style controls frame
font_controls_frame = tk.Frame(text_editor_frame, bg="#303F9F")
font_controls_frame.pack(pady=10)

font_label = tk.Label(font_controls_frame, text="Font Style:", bg="#303F9F", fg="white", font=("Arial", 14))
font_label.pack(side=tk.LEFT, padx=10)

font_combo = ttk.Combobox(font_controls_frame, values=["Arial", "Times New Roman", "Courier New"], state="readonly", font=("Arial", 14))
font_combo.set("Arial")
font_combo.pack(side=tk.LEFT, padx=10)
font_combo.bind("<<ComboboxSelected>>", lambda event: change_font())

size_label = tk.Label(font_controls_frame, text="Font Size:", bg="#303F9F", fg="white", font=("Arial", 14))
size_label.pack(side=tk.LEFT, padx=10)

size_combo = ttk.Combobox(font_controls_frame, values=[str(i) for i in range(8, 25)], state="readonly", font=("Arial", 14))
size_combo.set("16")
size_combo.pack(side=tk.LEFT, padx=10)
size_combo.bind("<<ComboboxSelected>>", lambda event: change_font())

color_button = tk.Button(font_controls_frame, text="Choose Text Color", command=choose_text_color, bg="#FFC107", fg="black", font=("Arial", 14))
color_button.pack(side=tk.LEFT, padx=10)

text_color_var = tk.StringVar(value="black")

# Save as TXT and PDF buttons
save_txt_button = tk.Button(font_controls_frame, text="Save as TXT", command=save_to_txt, bg="#4CAF50", fg="white", font=("Arial", 14))
save_txt_button.pack(side=tk.LEFT, padx=10)

save_pdf_button = tk.Button(font_controls_frame, text="Save as PDF", command=save_to_pdf, bg="#4CAF50", fg="white", font=("Arial", 14))
save_pdf_button.pack(side=tk.LEFT, padx=10)

# Text formatting buttons
bold_button = tk.Button(font_controls_frame, text="Bold", command=toggle_bold, bg="#FFC107", fg="black", font=("Arial", 14))
bold_button.pack(side=tk.LEFT, padx=10)

italic_button = tk.Button(font_controls_frame, text="Italic", command=toggle_italic, bg="#FFC107", fg="black", font=("Arial", 14))
italic_button.pack(side=tk.LEFT, padx=10)

underline_button = tk.Button(font_controls_frame, text="Underline", command=toggle_underline, bg="#FFC107", fg="black", font=("Arial", 14))
underline_button.pack(side=tk.LEFT, padx=10)

# Text alignment buttons
align_left_button = tk.Button(font_controls_frame, text="Left", command=lambda: align_text("left"), bg="#FFC107", fg="black", font=("Arial", 14))
align_left_button.pack(side=tk.LEFT, padx=10)

align_center_button = tk.Button(font_controls_frame, text="Center", command=lambda: align_text("center"), bg="#FFC107", fg="black", font=("Arial", 14))
align_center_button.pack(side=tk.LEFT, padx=10)

align_right_button = tk.Button(font_controls_frame, text="Right", command=lambda: align_text("right"), bg="#FFC107", fg="black", font=("Arial", 14))
align_right_button.pack(side=tk.LEFT, padx=10)

# Clear, Undo, and Redo buttons
clear_button = tk.Button(font_controls_frame, text="Clear", command=clear_text, bg="#FF5722", fg="white", font=("Arial", 14))
clear_button.pack(side=tk.LEFT, padx=10)

undo_button = tk.Button(font_controls_frame, text="Undo", command=undo, bg="#2196F3", fg="white", font=("Arial", 14))
undo_button.pack(side=tk.LEFT, padx=10)

redo_button = tk.Button(font_controls_frame, text="Redo", command=redo, bg="#4CAF50", fg="white", font=("Arial", 14))
redo_button.pack(side=tk.LEFT, padx=10)

# Find and Replace button
find_replace_button = tk.Button(font_controls_frame, text="Find & Replace", command=find_and_replace, bg="#9C27B0", fg="white", font=("Arial", 14))
find_replace_button.pack(side=tk.LEFT, padx=10)

# Status Bar
status_bar = tk.Label(text_editor_frame, text="Words: 0 | Characters: 0", bd=1, relief=tk.SUNKEN, anchor=tk.W, bg="#303F9F", fg="white", font=("Arial", 12))
status_bar.pack(side=tk.BOTTOM, fill=tk.X)

# Initializing the window
switch_window("password")


# Create Menu Bar
menu_bar = tk.Menu(app)
app.config(menu=menu_bar)

# Format Menu
format_menu = tk.Menu(menu_bar, tearoff=0)
menu_bar.add_cascade(label="Format", menu=format_menu)

# Add Formatting Options
format_menu.add_command(label="Bold", command=toggle_bold)
format_menu.add_command(label="Italic", command=toggle_italic)
format_menu.add_command(label="Underline", command=toggle_underline)

# Add Alignment Submenu
align_menu = tk.Menu(format_menu, tearoff=0)
format_menu.add_cascade(label="Align", menu=align_menu)
align_menu.add_command(label="Left", command=lambda: align_text("left"))
align_menu.add_command(label="Center", command=lambda: align_text("center"))
align_menu.add_command(label="Right", command=lambda: align_text("right"))

# Start the main event loop
app.mainloop()
