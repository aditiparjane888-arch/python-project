# python-project
Simple Python project made by Aditi
import sqlite3
from colorama import init, Fore

init(autoreset=True)


class Database:

    def __init__(self):
        self.conn = sqlite3.connect("database.db")
        self.cursor = self.conn.cursor()
        self.create_tables()

    def create_tables(self):

        self.cursor.execute("""
        CREATE TABLE IF NOT EXISTS users(
            username TEXT PRIMARY KEY,
            password TEXT
        )
        """)

        self.cursor.execute("""
        CREATE TABLE IF NOT EXISTS students(
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT,
            age INTEGER,
            subjects TEXT,
            total REAL,
            percentage REAL
        )
        """)

        self.conn.commit()

        self.cursor.execute(
            "SELECT * FROM users WHERE username=?",
            ("admin",)
        )

        if not self.cursor.fetchone():
            self.cursor.execute(
                "INSERT INTO users VALUES (?,?)",
                ("admin", "admin123")
            )

            self.conn.commit()


class Login:

    def __init__(self, db):
        self.db = db

    def authenticate(self):

        print(Fore.CYAN + "\n===== LOGIN =====")

        username = input("Username: ")
        password = input("Password: ")

        self.db.cursor.execute(
            "SELECT * FROM users WHERE username=? AND password=?",
            (username, password)
        )

        user = self.db.cursor.fetchone()

        if user:
            print(Fore.GREEN + "\nLogin Successful")
            return True

        print(Fore.RED + "\nInvalid Credentials")
        return False


class Student:

    def __init__(self, name, age, subjects):

        self.name = name
        self.age = age
        self.subjects = subjects

        self.total = sum(subjects.values())

        self.percentage = round(
            self.total / len(subjects),
            2
        )


class StudentManagement:

    def __init__(self, db):
        self.db = db

    def add_student(self):

        print(Fore.YELLOW + "\n===== Add Student =====")

        name = input("Name: ")
        age = int(input("Age: "))

        subjects = {}

        count = int(input("No of subjects: "))

        for i in range(count):

            subject = input(f"Subject {i+1}: ")
            marks = float(input("Marks: "))

            subjects[subject] = marks

        student = Student(name, age, subjects)

        self.db.cursor.execute("""
        INSERT INTO students
        (
        name,
        age,
        subjects,
        total,
        percentage
        )

        VALUES(?,?,?,?,?)
        """,
        (
            student.name,
            student.age,
            str(student.subjects),
            student.total,
            student.percentage
        ))

        self.db.conn.commit()

        print(Fore.GREEN + "\nStudent Added")

    def view_students(self):

        print(Fore.BLUE + "\n===== Records =====")

        self.db.cursor.execute("SELECT * FROM students")

        data = self.db.cursor.fetchall()

        for row in data:

            print(Fore.WHITE)
            print("-" * 30)

            print(f"ID: {row[0]}")
            print(f"Name: {row[1]}")
            print(f"Age: {row[2]}")
            print(f"Subjects: {row[3]}")
            print(f"Total: {row[4]}")
            print(f"Percentage: {row[5]}%")

    def delete_student(self):

        sid = int(input("Enter Student ID: "))

        self.db.cursor.execute(
            "DELETE FROM students WHERE id=?",
            (sid,)
        )

        self.db.conn.commit()

        print(Fore.RED + "\nStudent Deleted")


# Main Program

db = Database()
login = Login(db)

if login.authenticate():

    system = StudentManagement(db)

    while True:

        print(Fore.MAGENTA + "\n===== MENU =====")
        print("1. Add Student")
        print("2. View Students")
        print("3. Delete Student")
        print("4. Exit")

        choice = input("Enter Choice: ")

        if choice == "1":
            system.add_student()

        elif choice == "2":
            system.view_students()

        elif choice == "3":
            system.delete_student()

        elif choice == "4":
            print(Fore.CYAN + "\nThank You")
            break

        else:
            print(Fore.RED + "\nInvalid Choice")