
import tkinter as tk
import ast
import operator
import re

# Calculator colors
BG = "#171a24"
BUTTON_BG = "#292d3b"
UTILITY_BG = "#3a4052"
OPERATOR_BG = "#253c5a"
EQUALS_BG = "#4d8df7"
WHITE = "#ffffff"

root = tk.Tk()
root.title("Python Calculator")
root.geometry("360x540")
root.minsize(300, 450)
root.configure(bg=BG)

expression = ""
just_calculated = False

display = tk.StringVar(value="0")
history = tk.StringVar(value="")

# Safe arithmetic evaluator: no eval() or arbitrary code execution.
BINARY_OPERATORS = {
    ast.Add: operator.add,
    ast.Sub: operator.sub,
    ast.Mult: operator.mul,
    ast.Div: operator.truediv,
    ast.Mod: operator.mod,
}

UNARY_OPERATORS = {
    ast.UAdd: operator.pos,
    ast.USub: operator.neg,
}


def safe_calculate(text):
    # Translate calculator symbols into Python operators.
    text = text.replace("×", "*").replace("÷", "/")
    tree = ast.parse(text, mode="eval")

    def evaluate(node):
        if isinstance(node, ast.Expression):
            return evaluate(node.body)

        if isinstance(node, ast.Constant) and type(node.value) in (int, float):
            return node.value

        if isinstance(node, ast.BinOp) and type(node.op) in BINARY_OPERATORS:
            left = evaluate(node.left)
            right = evaluate(node.right)

            if isinstance(node.op, ast.Div) and right == 0:
                raise ZeroDivisionError

            result = BINARY_OPERATORS[type(node.op)](left, right)

            if abs(result) == float("inf") or result != result:
                raise ValueError("Invalid result")

            return result

        if isinstance(node, ast.UnaryOp) and type(node.op) in UNARY_OPERATORS:
            return UNARY_OPERATORS[type(node.op)](evaluate(node.operand))

        raise ValueError("Invalid expression")

    result = evaluate(tree)

    if isinstance(result, float) and result.is_integer():
        return str(int(result))

    return f"{result:.10g}" if isinstance(result, float) else str(result)


def update_display():
    display.set(expression if expression else "0")


def press_number(number):
    global expression, just_calculated

    if just_calculated:
        expression = ""
        history.set("")
        just_calculated = False

    if len(expression) < 100:
        if expression == "0":
            expression = number
        else:
            expression += number

    update_display()


def press_decimal():
    global expression, just_calculated

    if just_calculated:
        expression = ""
        history.set("")
        just_calculated = False

    # Allow a decimal point in the current number only once.
    current = re.split(r"[+\-*/%]", expression)[-1]

    if "." not in current:
        if not expression or expression[-1] in "+-*/%":
            expression += "0."
        else:
            expression += "."

    update_display()


def press_operator(op):
    global expression, just_calculated

    if not expression:
        if op == "-":
            expression = "-"
        else:
            return

    elif expression[-1] in "+-*/%":
        expression = expression[:-1] + op

    else:
        expression += op

    just_calculated = False
    update_display()


def clear():
    global expression, just_calculated

    expression = ""
    just_calculated = False
    display.set("0")
    history.set("")


def delete():
    global expression, just_calculated

    if just_calculated:
        clear()
        return

    expression = expression[:-1]
    update_display()


def toggle_sign():
    global expression, just_calculated

    if not expression:
        expression = "-"
    elif expression.startswith("-"):
        expression = expression[1:]
    else:
        expression = "-" + expression

    just_calculated = False
    update_display()


def percent():
    global expression, just_calculated

    if not expression:
        return

    try:
        value = safe_calculate(expression)
        expression = safe_calculate(f"({value})/100")
        just_calculated = True
        update_display()
    except (ValueError, SyntaxError, ZeroDivisionError, OverflowError):
        display.set("Error")
        expression = ""
        just_calculated = True


def calculate():
    global expression, just_calculated

    if not expression:
        return

    try:
        original = expression
        result = safe_calculate(expression)

        history.set(original + " =")
        expression = result
        display.set(result)
        just_calculated = True

    except (ValueError, SyntaxError, ZeroDivisionError,
            OverflowError, RecursionError):
        display.set("Cannot calculate")
        expression = ""
        just_calculated = True


# Layout
tk.Label(
    root,
    text="CALCULATOR",
    bg=BG,
    fg=WHITE,
    font=("Arial", 16, "bold")
).pack(anchor="w", padx=20, pady=(20, 8))

screen = tk.Frame(root, bg=BG)
screen.pack(fill="both", expand=True, padx=20)

tk.Label(
    screen,
    textvariable=history,
    bg=BG,
    fg="#9298aa",
    font=("Arial", 12),
    anchor="e"
).pack(fill="x", pady=(15, 5))

tk.Label(
    screen,
    textvariable=display,
    bg=BG,
    fg=WHITE,
    font=("Arial", 30),
    anchor="e",
    justify="right",
    wraplength=310
).pack(fill="both", expand=True, pady=(0, 15))

button_frame = tk.Frame(root, bg=BG)
button_frame.pack(fill="both", expand=True, padx=15, pady=10)

buttons = [
    ("AC", clear, UTILITY_BG),
    ("DEL", delete, UTILITY_BG),
    ("%", percent, UTILITY_BG),
    ("÷", lambda: press_operator("/"), OPERATOR_BG),

    ("7", lambda: press_number("7"), BUTTON_BG),
    ("8", lambda: press_number("8"), BUTTON_BG),
    ("9", lambda: press_number("9"), BUTTON_BG),
    ("×", lambda: press_operator("*"), OPERATOR_BG),

    ("4", lambda: press_number("4"), BUTTON_BG),
    ("5", lambda: press_number("5"), BUTTON_BG),
    ("6", lambda: press_number("6"), BUTTON_BG),
    ("−", lambda: press_operator("-"), OPERATOR_BG),

    ("1", lambda: press_number("1"), BUTTON_BG),
    ("2", lambda: press_number("2"), BUTTON_BG),
    ("3", lambda: press_number("3"), BUTTON_BG),
    ("+", lambda: press_operator("+"), OPERATOR_BG),

    ("+/−", toggle_sign, UTILITY_BG),
    ("0", lambda: press_number("0"), BUTTON_BG),
    (".", press_decimal, BUTTON_BG),
    ("=", calculate, EQUALS_BG),
]

for col in range(4):
    button_frame.columnconfigure(col, weight=1, uniform="col")

for row in range(5):
    button_frame.rowconfigure(row, weight=1, uniform="row")

for index, (label, command, color) in enumerate(buttons):
    row, col = divmod(index, 4)

    btn = tk.Button(
        button_frame,
        text=label,
        command=command,
        bg=color,
        fg=WHITE,
        activebackground="#4a5268",
        activeforeground=WHITE,
        font=("Arial", 16, "bold"),
        relief="flat",
        bd=0,
        cursor="hand2"
    )
    btn.grid(
        row=row,
        column=col,
        sticky="nsew",
        padx=4,
        pady=4,
        ipadx=2,
        ipady=8
    )

# Keyboard support
def keyboard_input(event):
    key = event.char

    if key in "0123456789":
        press_number(key)
    elif key in "+-*/":
        press_operator(key)
    elif key == ".":
        press_decimal()
    elif key == "%":
        percent()
    elif key in "\r=":
        calculate()
    elif event.keysym == "BackSpace":
        delete()
    elif event.keysym == "Escape":
        clear()

root.bind("<Key>", keyboard_input)

root.mainloop()
