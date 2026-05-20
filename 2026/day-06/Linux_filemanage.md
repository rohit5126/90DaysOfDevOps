Today’s goal is to practice basic file read/write using only fundamental commands.

You will create a small text file and practice:

Creating a file
Writing text to a file
Appending new lines
Reading the file back

## creating a file.
```bash
touch practice.txt
```

## adding lines to a file.
```bash
echo "this is the first line in a file" >> practice.txt # >> appends new line in the file

echo "I am here to practice linux fundamental commands for file management" > practice.txt # > replace all the text with this line.

echo "this is second  line in the file" >> practice.txt

echo "I have pledged to stay consistant, and work hard to gain success.
I am not going to stop until I find a high paying job.
my sole motive is to make my family proud.
I want to take care of everything and make them happu.
for that whatever I will ahve to do I will do it.
thanks for reading my file"  >> practice.txt
```

## reading this file
```bash
head -n 3 practice.txt #shows top 3 lines

tail -n 3 practice.txt #shows last 3 lines

grep family practice.txt #search for family work in the file

less practice.txt #open a file to view text.

cat practice.txt #see the text in terminal

tee diku.txt #replaces old text and prints the new text at the same time.
  
echo "this is first line" | tee -a diku.txt  #add text in the next line

tee -a diku.txt # -a flag is used to add the text in  next line
```




