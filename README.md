import tkinter as tk
import random
from tkinter import messagebox

class MemoryGame:
    def __init__(self, root):
        self.root = root
        self.root.title("Memory Match Game (Animated)")
        self.root.geometry("470x540")
        self.root.configure(bg="#121826")

        self.mode = None
        self.current_player = 1
        self.first = None
        self.second = None

        self.moves = 0
        self.scores = {1: 0, 2: 0}

        self.buttons = {}
        self.values = []

        self.show_menu()

    # ---------------- MENU ----------------
    def show_menu(self):
        self.clear()

        tk.Label(self.root, text="MEMORY GAME",
                 font=("Arial", 24, "bold"),
                 bg="#121826", fg="white").pack(pady=40)

        tk.Button(self.root, text="1 PLAYER",
                 font=("Arial", 14, "bold"),
                 bg="#22c55e", fg="white",
                 width=18,
                 command=lambda: self.start_game(1)).pack(pady=10)

        tk.Button(self.root, text="2 PLAYERS",
                 font=("Arial", 14, "bold"),
                 bg="#3b82f6", fg="white",
                 width=18,
                 command=lambda: self.start_game(2)).pack(pady=10)

    # ---------------- START GAME ----------------
    def start_game(self, mode):
        self.mode = mode
        self.current_player = 1
        self.moves = 0
        self.scores = {1: 0, 2: 0}
        self.first = None
        self.second = None

        self.clear()
        self.build_ui()
        self.build_board()

    # ---------------- UI ----------------
    def build_ui(self):
        self.title = tk.Label(self.root, text="Memory Match Game",
                              font=("Arial", 18, "bold"),
                              bg="#121826", fg="white")
        self.title.pack(pady=5)

        self.info = tk.Label(self.root, text="",
                             font=("Arial", 12),
                             bg="#121826", fg="#93c5fd")
        self.info.pack()

        self.score = tk.Label(self.root, text="",
                              font=("Arial", 12),
                              bg="#121826", fg="white")
        self.score.pack()

        self.frame = tk.Frame(self.root, bg="#121826")
        self.frame.pack(pady=20)

        tk.Button(self.root, text="Restart",
                 font=("Arial", 12),
                 bg="#ef4444", fg="white",
                 command=self.show_menu).pack(pady=10)

    # ---------------- BOARD ----------------
    def build_board(self):
        nums = list(range(1, 9)) * 2
        random.shuffle(nums)
        self.values = nums
        self.buttons = {}

        for i in range(4):
            for j in range(4):
                idx = i * 4 + j

                btn = tk.Button(
                    self.frame,
                    text="❓",
                    width=6,
                    height=3,
                    font=("Arial", 14, "bold"),
                    bg="#1f2937",
                    fg="white",
                    activebackground="#374151",
                    command=lambda i=idx: self.click(i)
                )
                btn.grid(row=i, column=j, padx=6, pady=6)
                self.buttons[idx] = btn

        self.update_ui()

    # ---------------- CLICK ----------------
    def click(self, idx):
        btn = self.buttons[idx]

        if btn["text"] != "❓":
            return

        if self.first is not None and self.second is not None:
            return

        # 🔹 Click animation (press effect)
        btn.config(bg="#2563eb")
        self.root.after(100, lambda: btn.config(bg="#1f2937"))

        btn["text"] = str(self.values[idx])

        if self.first is None:
            self.first = idx
        else:
            self.second = idx
            self.moves += 1
            self.root.after(600, self.check_match)

    # ---------------- MATCH CHECK ----------------
    def check_match(self):
        i, j = self.first, self.second

        if self.values[i] == self.values[j]:
            self.match_effect(i, j)

            if self.mode == 2:
                self.scores[self.current_player] += 1
            else:
                self.scores[1] += 1

        else:
            self.mismatch_effect(i, j)

            self.buttons[i]["text"] = "❓"
            self.buttons[j]["text"] = "❓"

            if self.mode == 2:
                self.current_player = 2 if self.current_player == 1 else 1

        self.first = None
        self.second = None

        self.update_ui()
        self.check_win()

    # ---------------- ANIMATIONS ----------------
    def match_effect(self, i, j):
        for idx in (i, j):
            self.buttons[idx].config(bg="#22c55e")

        self.root.after(300, lambda: [
            self.buttons[i].config(bg="#1f2937"),
            self.buttons[j].config(bg="#1f2937")
        ])

    def mismatch_effect(self, i, j):
        def shake(step=0):
            if step % 2 == 0:
                color = "#ef4444"
            else:
                color = "#1f2937"

            self.buttons[i].config(bg=color)
            self.buttons[j].config(bg=color)

            if step < 4:
                self.root.after(80, lambda: shake(step + 1))
            else:
                self.buttons[i].config(bg="#1f2937")
                self.buttons[j].config(bg="#1f2937")

        shake()

    # ---------------- UI UPDATE ----------------
    def update_ui(self):
        if self.mode == 2:
            self.info.config(text=f"Player Turn: Player {self.current_player}")

            self.score.config(
                text=f"P1: {self.scores[1]}   |   P2: {self.scores[2]}   |   Moves: {self.moves}"
            )
        else:
            self.info.config(text="Single Player Mode")
            self.score.config(text=f"Score: {self.scores[1]} | Moves: {self.moves}")

    # ---------------- WIN ----------------
    def check_win(self):
        for btn in self.buttons.values():
            if btn["text"] == "❓":
                return

        if self.mode == 2:
            if self.scores[1] > self.scores[2]:
                msg = "Player 1 Wins!"
            elif self.scores[2] > self.scores[1]:
                msg = "Player 2 Wins!"
            else:
                msg = "It's a Draw!"
        else:
            msg = f"Completed! Score: {self.scores[1]}"

        messagebox.showinfo("Game Over", msg)

    # ---------------- CLEAR ----------------
    def clear(self):
        for w in self.root.winfo_children():
            w.destroy()


if __name__ == "__main__":
    root = tk.Tk()
    game = MemoryGame(root)
    root.mainloop()
