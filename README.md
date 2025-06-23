# Hangman
import tkinter as tk
import random

# Word list to choose from
word_list = [
    "holiday", "reindeer", "chimney", "present",
    "snowman", "santa", "candy", "ornament"
]

# Tree stages - as individual lines for animation
tree_stages = [
    "",
    "    *",
    "   ***",
    "  *****",
    " *******",
    "*********",
    "   |||",
    "<- Tree fully hung!"
]

max_wrong_guesses = len(tree_stages) - 2  # Exclude "<- Tree fully hung!" from stages count

class HangmanGame:
    def __init__(self, root):
        self.root = root
        self.root.title("Holiday Hangman Game with Animation")
        self.root.geometry("500x500")
        self.reset_game()

    def reset_game(self):
        self.chosen_word = random.choice(word_list)
        self.display = ["_" for _ in self.chosen_word]
        self.guessed_letters = []
        self.wrong_guesses = 0
        self.current_tree = []

        for widget in self.root.winfo_children():
            widget.destroy()

        self.tree_label = tk.Label(self.root, text="", font=("Courier", 14), justify="left")
        self.tree_label.pack(pady=10)

        self.word_label = tk.Label(self.root, text=" ".join(self.display), font=("Helvetica", 20))
        self.word_label.pack(pady=10)

        self.guesses_label = tk.Label(self.root, text="Guessed Letters: ", font=("Helvetica", 12))
        self.guesses_label.pack(pady=5)

        self.buttons_frame = tk.Frame(self.root)
        self.buttons_frame.pack()

        for i in range(26):
            letter = chr(97 + i)
            btn = tk.Button(self.buttons_frame, text=letter.upper(), width=3, command=lambda l=letter: self.guess_letter(l))
            btn.grid(row=i//9, column=i%9, padx=2, pady=2)

        self.message_label = tk.Label(self.root, text="", font=("Helvetica", 14))
        self.message_label.pack(pady=10)

    def guess_letter(self, letter):
        if letter in self.guessed_letters:
            return

        self.guessed_letters.append(letter)
        self.guesses_label.config(text="Guessed Letters: " + ", ".join(self.guessed_letters))

        if letter in self.chosen_word:
            for i, char in enumerate(self.chosen_word):
                if char == letter:
                    self.display[i] = letter
            self.word_label.config(text=" ".join(self.display))
            if "_" not in self.display:
                self.end_game(True)
        else:
            self.animate_tree()

    def animate_tree(self):
        if self.wrong_guesses < max_wrong_guesses:
            next_line = tree_stages[self.wrong_guesses + 1]
            self.current_tree.append(next_line)
            self.wrong_guesses += 1
            self.update_tree_display()
        else:
            # Final stage: Show the "fully hung" message
            self.current_tree.append(tree_stages[-2])  # Add "   |||" last line
            self.update_tree_display()
            self.current_tree.append(tree_stages[-1])  # Add "<- Tree fully hung!"
            self.tree_label.config(text="\n".join(self.current_tree))
            self.end_game(False)

    def update_tree_display(self):
        self.tree_label.config(text="\n".join(self.current_tree))

    def end_game(self, won):
        for btn in self.buttons_frame.winfo_children():
            btn.config(state="disabled")

        if won:
            self.message_label.config(text="🎉 You Saved the Tree!", fg="green")
        else:
            self.message_label.config(text=f"Oh no! The word was '{self.chosen_word}'", fg="red")

        restart_btn = tk.Button(self.root, text="Play Again", command=self.reset_game)
        restart_btn.pack(pady=10)

# Run the GUI
root = tk.Tk()
game = HangmanGame(root)
root.mainloop()
