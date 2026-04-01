---
layout: post
title: Creating a Quiz Game with Python
image: "/posts/Quiz_Game.png"
tags: [Python, Quiz]
---

As a personal project to test my Python skills, I decided to create a fun quiz game, with varying levels of difficulty, selectable categories, a progress counter and a final score calculator.

Whilst this is a simple version of a quiz game, it begins to show just how easily very powerful tools can be created in Python, and demonstrates some useful tricks and concepts within the program.

---

# Initial Setup

We begin by defining the questions! A dictionary is perfect for the concept of a quiz; dictionaries hold information in the form of key-value pairs, which intuitively connects perfectly with questions and their corresponding answers.

Here we will use a nested dictionary; this will effectively function as a library to be searched through sequentially when different criteria are called during the quiz. At the highest level, each category is a key-value pair (the name of the category and the information within it. Within the category come the difficulty dictionaries (the name of the difficulty and the questions within that). Then, as alluded to above, each question/answer combo has its own dictionary: in this case, the words "question" and "answer" as the two keys to be called on in the game, and the actual answers written as strings as the values.

The questions are then entered manually, and so can be modified at any time, while all variables are defined using dynamic logic to allow for changes in either the content of questions, or the number of questions.

An example of this, for the sport category, is below (I won't print the whole codeblock as it is fairly monotonous):

```python
questions = {
    "sport": {
        "easy": [
            {"question": "How many players are in a rugby union team?", "answer": "15"},
            {"question": "What sport uses a racket and a shuttlecock?", "answer": "badminton"}
        ],
        "medium": [
            {"question": "Which country won the 2006 FIFA World Cup?", "answer": "Italy"},
            {"question": "In cricket, how many runs does a batter score for hitting the ball over the boundary without bouncing?", "answer": "6"}
        ],
        "hard": [
            {"question": "Who has won the most Formula 1 World Championships?", "answer": "Lewis Hamilton"},
            {"question": "Which country hosted the 2018 Winter Olympics?", "answer": "South Korea"}
        ]
    }
```

Next, we will define some functions that will be used in our final quiz function that will helo broaden the range of answers accepted by the quiz. For example, a user inputting any of 'Fifteen', '15' and 'fifteen' should be given the same result.

Firstly, we'll create a function called that will clean text, i.e. remove any dependence on capitalisation or punctuation, and leading/trailing spaces:

```python
# function for allowing for different types of text inputs
def clean_text(text):
    text = text.lower()           # capitalisations/lack of capitalisations won't matter
    text = text.replace(".", "")  # points/lack of points won't matter
    text = text.replace(",", "")  # commas remnoved
    text = text.replace("'", "")  # apostrophes removed
    text = text.strip()           # removes leading and trailing spaces
    return text
```

Then, another function (numeric_number) that turns the written version of any number from 1-1000 into its numeric form:

```python
def number_to_words(n):  # function that creates a dictionary from 1:n containing numbers in written and numeric form
    
    # define all potential outcomes in word form first
    ones = ["zero", "one", "two", "three", "four", "five", "six", "seven", "eight", "nine"]
    teens = ["ten", "eleven", "twelve", "thirteen", "fourteen", "fifteen",
             "sixteen", "seventeen", "eighteen", "nineteen"]
    tens = ["", "", "twenty", "thirty", "forty", "fifty",
            "sixty", "seventy", "eighty", "ninety"]

    if n < 10:
        return ones[n]  # moves sequentially along 'ones' for each n < 10
    elif n < 20:
        return teens[n - 10]  # moves sequentially along 'teens' if 10 <= n < 20
    elif n < 100:
        t = tens[n // 10]     # if 20 <= n < 100, t = index for 'tens' list i.e. when n = 20, return 2nd value in 'tens' (hence why 0th adn 1st values are blank)
        o = n % 10            # remainder left over after n is divided by 10
        return t if o == 0 else f"{t} {ones[o]}"  # if o is non-zero, find t (e.g. twenty) and add it to o's corresponding value from 'ones'
    elif n < 1000:
        h = ones[n // 100] + " hundred"  # if 100 <= n < 1000, h = index for ones list, then added to word 'hundred'
        r = n % 100                      # remainder left over after n is divided by 100
        if r == 0:    
            return h
        return h + " " + number_to_words(r) # if r is non-zero, find h (e.g. a hundred) and add it to r's corresponding value from all 3 of the above stages 
    elif n == 1000:
        return "one thousand"
        return "one thousand"

number_words = {}                # empty dictionary to append our word:number key-value pairs to

for i in range(0, 1001):
    word = number_to_words(i)
    number_words[word] = str(i)  # defines 'word' as the key and the corresponding number as the value, stored as a string

# creating a function that returns the numeric value when the written form is input
def numeric_number(text):
    text = text.lower().strip()   # ensures the input has everything lower case and removes spaces
    if text in number_words:
        return number_words[text]
    return text                   # spits the text back if no match for numbers 1-1000 (avoids errors)
```

Then, we'll create a dictionary that allows for a difficulty-based scoring system:

```python
difficulty_points = {
    "easy": 1,
    "medium": 2,
    "hard": 3
}
```

Finally, we'll need the functionality that the 'random' module in Python provides later, so let's import that:

```python
import random
```

---

# Defining the Quiz Logic

First, we will define the logic that will make the quiz run at a *category* level first. This way we can test the logic, and if it this works, scale it up. This entails: 

> 1. Defining a function so this logic can be called later
> 2. Asking the user which difficulty they want to try, and generating an error message if they return an invalid difficulty
> 3. Randomly selecting a question (under the given category and difficulty constraints)
> 4. Creating a score object and setting that to 0
> 5. Asking the user the question and defining their answer as a variable
> 6. 'Cleaning' their answer using the functions defined above
> 7. Creating a list of acceptable answers to the given question, and setting the correct_answer as the 0th element of that list (this is used later in a print statement if the user gets the wrong answer)
> 8. 'Cleaning' the list of acceptable answers
> 9. Checking the cleaned answer against the cleaned acceptables, and returning correct if there's a match, and incorrect if not
> 10. Updating the user's score accordingly
> 11. Updating the user on their progress after the category

In terms of code, this looks like:

```python
          def quiz_logic():
                print("\nDifficulty levels: easy, medium, hard. \n\n1 point for each correct answer to an easy question, 2 points for medium, 3 points for hard!")
                difficulty = input("\nChoose a difficulty: ").lower()

                if difficulty not in questions[category]:
                    print("Invalid difficulty. Please choose a category again.")
                    continue

                selected_questions = questions[category][difficulty]            # searching through questions dict to find the relevant category dict, then the relevant difficulty dict
                random.shuffle(selected_questions)

                score = 0

                for q in selected_questions:                                    # ask all qs in category first
                    answer = input(q["question"] + " ")

                    clean_answer = numeric_number(clean_text(answer))           # cleans user answer

                    acceptable = q["answer"]
                    if isinstance(acceptable, str):                             # creating list of acceptable answers in string form
                        acceptable = [acceptable]

                    correct_answer = acceptable[0]                              # returns the first element of the list of acceptable answers

                    cleaned_acceptables = [numeric_number(clean_text(a)) for a in acceptable]

                    correct = False
                    for a in cleaned_acceptables:                               # checks answer against acceptables
                        if clean_answer == a:
                            correct = True
                        elif clean_answer in a:
                            correct = True

                    if correct:
                        print("Correct!")
                        score += difficulty_points[difficulty]   # difficulty-based scoring counter
                    else:
                        print(f"Incorrect. The correct answer was {correct_answer}.")

                print(f"\nYou scored {score} out of {len(selected_questions)*difficulty_points

```

Now that we know this works, we can build a repeatable framework that can run the above over every category. 

Firstly, we set up a menu functionality that can continue on to the quiz and has some error handling:

> 1. Define our function, quiz_game, and create menu, and give user the option to play or exit
> 2. If they choose to play, continue (see next section)
> 3. If they choose to leave, print exit message, and break loop
> 4. If they enter an invalid input, print error message and repeat framework

```python

def quiz_game():
    while True:                                # while True: so continues unless we/user breaks it
        print("\n=== QUIZ GAME MENU ===")
        print("1. Play Quiz")
        print("2. Exit")

        choice = input("Choose an option: ")

        """ Code for when user 'continues' goes in here """

        elif choice == "2":
            print("Goodbye!")
            break

        else:
            print("Invalid choice. Please try again.")
```

Secondly, we can define the logic for when the user continues. This will contain a while loop that loops over all categories (asking the user if they wish to continue each time), until the categories are used up. Of course, as each category is looped over, the quiz logic that we coded above runs each time.

> 1. Create if loop and define remaining_categories, total_score and completed_categories to keep track of later
> 2. Create while loop to allow the quiz to stop once remaining_categories goes to 0
> 3. Define  choice of category logic - user can choose the next category or exit, plus add error handling if the user gives an invalid response
> 4. Run quiz logic from above
> 5. Once the quiz logic has run, append the score from this category to the user's total score, add 1 to the completed_categories object, and remove the completed category from the remaining_categories object
> 6. If there are remaining_categories, return to the category choice section, if not print an exit statement and the user's final score
> 7. Print progress update after conditional break statement so it only prints if user continues

```python

 if choice == "1":

            remaining_categories = list(questions.keys())
            total_score = 0                                     # total score feature to accumlate user's score over categories
            completed_categories = 0                            # feature to count categories user has completed

            while remaining_categories:                         # runs until remaining_categories is empty
                print("\nRemaining categories:")
                for c in remaining_categories:
                    print("-", c)

                category = input("\nChoose a category (or type 'exit' to stop): ").lower()

                if category == "exit":
                    break

                if category not in remaining_categories:
                    print("Invalid or already completed category.")
                    continue

                quiz_logic()                                                      # run quiz logic from above
                    
                total_score += score                                              # accumulates score after each category loop completed
                completed_categories += 1                                         # adds category to list of completed categories after each loop

                remaining_categories.remove(category)                             # then remove category only after category loop

                if remaining_categories:
                    print("\nReturning to category selection...")
                else:
                    print("\nAll categories completed!")
                    print(f"Final total score: {total_score} points out of a maximum possible score of {len(selected_questions)*len(difficulty_points)*len(questions)}!")
                    break

                print(f"\nProgress: {completed_categories}/{len(questions)} categories completed.")  # progress update
                print(f"Total score so far: {total_score} points.")
```

We now have everything in place for a simple but flexible, repeatable and fun quiz game! Provided we define the functions and questions above, the quiz itself all together looks like:

```python

def quiz_game():
    while True:
        print("\n=== QUIZ GAME MENU ===")
        print("1. Play Quiz")
        print("2. Exit")

        choice = input("Choose an option: ")

        if choice == "1":

            remaining_categories = list(questions.keys())
            total_score = 0                                     
            completed_categories = 0                           

            while remaining_categories:
                print("\nRemaining categories:")
                for c in remaining_categories:
                    print("-", c)

                category = input("\nChoose a category (or type 'exit' to stop): ").lower()

                if category == "exit":
                    break

                if category not in remaining_categories:
                    print("Invalid or already completed category.")
                    continue

                print("\nDifficulty levels: easy, medium, hard. \n\n1 point for each correct answer to an easy question, 2 points for medium, 3 points for hard!")
                difficulty = input("\nChoose a difficulty: ").lower()

                if difficulty not in questions[category]:
                    print("Invalid difficulty. Please choose a category again.")
                    continue

                selected_questions = questions[category][difficulty]
                random.shuffle(selected_questions)

                score = 0

                for q in selected_questions:                                   
                    answer = input(q["question"] + " ")

                    clean_answer = numeric_number(clean_text(answer))

                    acceptable = q["answer"]
                    if isinstance(acceptable, str):
                        acceptable = [acceptable]

                    correct_answer = acceptable[0]                              

                    cleaned_acceptables = [numeric_number(clean_text(a)) for a in acceptable]

                    correct = False
                    for a in cleaned_acceptables:
                        if clean_answer == a:
                            correct = True
                        elif clean_answer in a:
                            correct = True

                    if correct:
                        print("Correct!")
                        score += difficulty_points[difficulty]  
                    else:
                        print(f"Incorrect. The correct answer was {correct_answer}.")

                print(f"\nYou scored {score} out of {len(selected_questions)*difficulty_points[difficulty]}.")  

                total_score += score                                             
                completed_categories += 1                                         

                remaining_categories.remove(category)                        

                if remaining_categories:
                    print("\nReturning to category selection...")
                else:
                    print("\nAll categories completed!")
                    print(f"Final total score: {total_score} points out of a maximum possible score of {len(selected_questions)*len(difficulty_points)*len(questions)}!")
                    break

                print(f"\nProgress: {completed_categories}/{len(questions)} categories completed.")  
                print(f"Total score so far: {total_score} points.")


        elif choice == "2":
            print("Goodbye!")
            break

        else:
            print("Invalid choice. Please try again.")

# calling function to run game 
quiz_game()

```

For a fun task that's meant as nothing more as a learning tool and hobby, I think this more than serves a purpose!

However, further improvements to this could include:

1. A GUI to allow a better user experience
2. More questions/categories, or perhaps something to call on that dynamically updates questions every once in a while
3. A random category or random difficulty mode
4. A question timer
5. More of the code to be contained within functions - this could use some tidying for readability
6. Better input validation loops - these are currently bit clunky
7. A "back" or "return" option once a user is into a category

Thanks for reading!

