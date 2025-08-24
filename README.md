import random
import time
import tkinter as tk
from tkinter import messagebox, ttk

class QuizApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Quiz Application")
        self.root.geometry("650x450")
        self.questions = {}
        self.score = 0
        self.hints_used = 0
        self.incorrect_answers = []
        self.current_question_index = 0
        self.current_category = None
        self.start_time = None
        self.difficulty = None
        self.time_limit = 10
        self.load_sample_questions()
        self.create_main_menu()

    def load_sample_questions(self):
        self.questions = {
            'AI': [
                {'question': 'Who is known as the father of AI?', 'options': ['Alan Turing', 'John McCarthy', 'Geoffrey Hinton', 'Elon Musk'], 'answer': 1, 'hint': 'He coined the term "Artificial Intelligence".'},
                {'question': 'What is the main objective of AI?', 'options': ['Data Processing', 'Automation', 'Mimicking Human Intelligence', 'Networking'], 'answer': 2, 'hint': 'It involves cognitive functions similar to humans.'},
            ],
            'Coding': [
                {'question': 'Which language is considered the backbone of web development?', 'options': ['Python', 'Java', 'JavaScript', 'C++'], 'answer': 2, 'hint': 'It is commonly used for front-end and back-end web development.'},
                {'question': 'Which of the following is a version control system?', 'options': ['Git', 'Docker', 'Linux', 'Flask'], 'answer': 0, 'hint': 'It helps manage code changes in collaborative development.'},
            ],
            'Python': [
                {'question': 'Which data structure is used to store key-value pairs in Python?', 'options': ['List', 'Tuple', 'Dictionary', 'Set'], 'answer': 2, 'hint': 'It uses curly braces {}.'},
                {'question': 'Which keyword is used to define a function in Python?', 'options': ['func', 'def', 'lambda', 'define'], 'answer': 1, 'hint': 'It is followed by the function name.'},
            ]
        }

    def create_main_menu(self):
        self.clear_window()

        label = tk.Label(self.root, text="🎓 Welcome to the Smart Quiz Application", font=("Arial", 20, "bold"))
        label.pack(pady=20)

        start_button = tk.Button(self.root, text="Start Quiz", command=self.start_quiz, font=("Arial", 14), bg="#4CAF50", fg="white")
        start_button.pack(pady=10)

        exit_button = tk.Button(self.root, text="Exit", command=self.root.quit, font=("Arial", 14), bg="red", fg="white")
        exit_button.pack(pady=10)

    def start_quiz(self):
        self.clear_window()
        self.select_category()

    def select_category(self):
        label = tk.Label(self.root, text="📂 Select a Category", font=("Arial", 16, "bold"))
        label.pack(pady=20)

        categories = list(self.questions.keys())
        for category in categories:
            button = tk.Button(self.root, text=category, font=("Arial", 14),
                               command=lambda c=category: self.select_difficulty(c))
            button.pack(pady=5)

    def select_difficulty(self, category):
        self.current_category = category
        self.clear_window()

        label = tk.Label(self.root, text="⚡ Select Difficulty Level", font=("Arial", 16, "bold"))
        label.pack(pady=20)

        difficulties = {'Easy': 15, 'Medium': 10, 'Hard': 5}
        for level, time_limit in difficulties.items():
            button = tk.Button(self.root, text=level, font=("Arial", 14),
                               command=lambda d=level, t=time_limit: self.begin_quiz(d, t))
            button.pack(pady=5)

    def begin_quiz(self, difficulty, time_limit):
        self.difficulty = difficulty
        self.time_limit = time_limit
        self.score = 0
        self.hints_used = 0
        self.incorrect_answers = []
        self.current_question_index = 0
        self.shuffle_questions(self.current_category)
        self.show_question()

    def shuffle_questions(self, category):
        random.shuffle(self.questions[category])

    def show_question(self):
        if self.current_question_index < len(self.questions[self.current_category]):
            self.clear_window()
            question_data = self.questions[self.current_category][self.current_question_index]

            # Progress
            progress_label = tk.Label(self.root, text=f"Question {self.current_question_index + 1}/{len(self.questions[self.current_category])}", font=("Arial", 14))
            progress_label.pack(pady=5)

            # Progress Bar
            progress = ttk.Progressbar(self.root, length=400, maximum=len(self.questions[self.current_category]), value=self.current_question_index+1)
            progress.pack(pady=5)

            # Score so far
            score_label = tk.Label(self.root, text=f"Score: {self.score}", font=("Arial", 12))
            score_label.pack(pady=5)

            label = tk.Label(self.root, text=f"Q{self.current_question_index + 1}: {question_data['question']}", font=("Arial", 16), wraplength=500, justify="center")
            label.pack(pady=20)

            for i, option in enumerate(question_data['options']):
                button = tk.Button(self.root, text=option, font=("Arial", 14),
                                   command=lambda ans=i: self.check_answer(ans, question_data['answer']))
                button.pack(pady=5)

            hint_button = tk.Button(self.root, text="💡 Hint", font=("Arial", 14),
                                    command=lambda: self.provide_hint(question_data))
            hint_button.pack(pady=5)

            self.start_time = time.time()
        else:
            self.show_result()

    def provide_hint(self, question):
        self.hints_used += 1
        messagebox.showinfo("Hint", question['hint'])

    def check_answer(self, user_answer, correct_answer):
        elapsed_time = time.time() - self.start_time
        if elapsed_time > self.time_limit:
            messagebox.showinfo("⏳ Time's up!", "You ran out of time for this question.")
            self.incorrect_answers.append(self.questions[self.current_category][self.current_question_index])
        elif user_answer == correct_answer:
            messagebox.showinfo("✅ Correct", "Well done!")
            self.score += 1
        else:
            correct_option = self.questions[self.current_category][self.current_question_index]['options'][correct_answer]
            messagebox.showinfo("❌ Wrong", f"Oops! The correct answer was: {correct_option}")
            self.incorrect_answers.append(self.questions[self.current_category][self.current_question_index])

        self.current_question_index += 1
        self.show_question()

    def show_result(self):
        self.clear_window()

        label = tk.Label(self.root, text="🎉 Quiz Completed!", font=("Arial", 20, "bold"))
        label.pack(pady=20)

        total_questions = len(self.questions[self.current_category])
        percentage = (self.score / total_questions) * 100

        result_label = tk.Label(self.root, text=f"Your final score: {self.score}/{total_questions}\nAccuracy: {percentage:.2f}%", font=("Arial", 16))
        result_label.pack(pady=10)

        self.award_badge(percentage)

        review_button = tk.Button(self.root, text="📖 Review Incorrect Answers", command=self.review_incorrect_answers, font=("Arial", 14))
        review_button.pack(pady=5)

        exit_button = tk.Button(self.root, text="🏠 Back to Main Menu", command=self.create_main_menu, font=("Arial", 14))
        exit_button.pack(pady=5)

    def award_badge(self, percentage):
        if percentage == 100:
            badge = "🌟 Excellent! Perfect Score Badge! 🌟"
        elif percentage >= 80:
            badge = "🎉 Great Job! High Achiever Badge! 🎉"
        elif percentage >= 50:
            badge = "👍 Good Effort! Keep Learning Badge 👍"
        else:
            badge = "📚 Don’t worry! Practice makes perfect."

        messagebox.showinfo("Badge Awarded", badge)

    def review_incorrect_answers(self):
        if self.incorrect_answers:
            review_window = tk.Toplevel(self.root)
            review_window.title("Review Incorrect Answers")
            review_window.geometry("600x400")

            label = tk.Label(review_window, text="Review your incorrect answers:", font=("Arial", 16, "bold"))
            label.pack(pady=20)

            for question in self.incorrect_answers:
                question_label = tk.Label(review_window, text=f"Q: {question['question']}", font=("Arial", 14), wraplength=500, justify="left")
                question_label.pack(pady=5)

                answer_label = tk.Label(review_window, text=f"✅ Correct Answer: {question['options'][question['answer']]}", font=("Arial", 14), fg="green")
                answer_label.pack(pady=5)

            close_button = tk.Button(review_window, text="Close", command=review_window.destroy, font=("Arial", 14))
            close_button.pack(pady=10)
        else:
            messagebox.showinfo("No Mistakes", "👏 You answered everything correctly!")

    def clear_window(self):
        for widget in self.root.winfo_children():
            widget.destroy()

def main():
    root = tk.Tk()
    app = QuizApp(root)
    root.mainloop()

if __name__ == "__main__":
    main()
# code-project-
code project description 
