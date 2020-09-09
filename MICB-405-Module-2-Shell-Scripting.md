# Module 2 Worksheet
## Bash & Scripting
### *Axel Hauduc - 18 September 2020*

# Breakout Room Session #1
### Exercise 1
Write a shell script that prints “Shell Scripting is Fun!” on the screen

### Exercise 2
Write a shell script to check to see if the file “file_path” exists. If it does exist, display “file_path passwords are enabled.” Next, check to see if you can write to the file. If you can, display “You have permissions to edit “file_path.””If you cannot, display “You do NOT have permissions to edit “file_path””

### Exercise 3
Write a shell script that displays “man”, ”bear”, ”pig”, ”dog”, ”cat”, and “sheep” on the screen with each appearing on a separate line. Try to do this in as few lines as possible.

### Exercise 4
Write a shell script that prompts the user for a name of a file or directory and reports if it is a regular file, a directory, or another type of file. Also perform an ls command against the file or directory with the long listing option.

### Exercise 5
Modify the previous script to that it accepts the file or directory name as an argument instead of prompting the user to enter it.


# Breakout Room Session #2
Excercise 6: Create a list of files by executing the below command in a blank folder:
```bash
touch {1..5}.fa; echo "this is a fasta file" | tee {1..5}.fa &> /dev/null; touch {1..3}.txt; echo "this is one of three text files" | tee {1..3}.txt &> /dev/null
```
You've created 5 .fa files and 3 .txt files. Create a script that loops through only text file, and adds the content of each to a new file called combined.txt.

