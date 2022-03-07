# **Part 1: Local Buffer Overflow**

## Task 1 : Getting Familiar with the Shellcode ->

To do this task, I compiled the code using the command “make”.
I noticed that a couple of binaries were created, which were the 32 bit and 64 bit shellcode. I executed the binaries and each of the binaries opened their shell prompt. I also noticed that these shell had the User Id of 1000, which is the nyuappsec user.
![part1-task1-img1](Artifacts/hr2340-screenshot1.png)

## Task 2 :  Understanding the Vulnerable Program ->

To understand the vulnerable program, I ran the “make” command, which created eight binaries of the vulnerable program, which is “stack.c”.
This program reads in the content of a file into an array of characters, and calls the “bof” function. This function is the vulnerable point of the program since this function copies the content of the input into another character array (called ”buffer”) whose size is only 100 characters. The vulnerable piece of code is the strcpy() function. If the input to this function is more than 100 ( which is not validated anywhere), it will overflow the buffer and overwrite on the stack.
The output after running the make command is as follows :
![part1-task2-img1](Artifacts/hr2340-screenshot2.png)

## Task 3 : Launching attack on the 32 bit program ->

To complete this task, I had to figure out a way to overwrite the return address part of the stack, such that when the “bof” function returns, it starts to execute our malicious code which we provide as an input in the “badfile”.

Steps :

1. I created an empty bad file and used GDB to debug “stack-L1-debug”. In this, I set a breakpoint at “bof”, and do a run. This stops the gdb at the point when the EBP register is not set yet. So i used a “next” command to pass to the following instruction.
![part1-task2-img1](Artifacts/hr2340-screenshot3.png)

 2. We can now figure the value of the EBP register (the stack pointer). To check it, i ran the command “p $ebp”. The return address is actually 4 bytes after the EBP pointer. Following this, I figured the buffer pointer address by using the command “p &buffer”. Using these two values, I was able to calculate the offset, which was 112.
![part1-task2-img1](Artifacts/hr2340-screenshot4.png)

3. Now we know the offset (112) and the “ret” value of the exploit code (since this starts from the next address after the EBP+4 address). To account for the extra data added by GDB, through trial and error I found out that we need to add another 200 the return address, so that it hits our NOP sled. I created an exploit code from gathering all these information as such :
![part1-task2-img1](Artifacts/hr2340-screenshot5.png)

4. After running the exploit, I could see that I was able to achieve a root shell for the user.
![part1-task2-img1](Artifacts/exploit32.png)

## Task 5 : Launching Attack on the 64 Bit program

This task is very similar to the 32 Bit attack, other than the issue that all the relevant 64 bit addresses in stack start with a 0 byte. This is an issue because when the strcpy() function reads the 0 byte, it will stop its execution. After attending the Office Hours, I figured that we can not write our shell code after the return address.
To start this attack, I again had to debug the code similar to the previous task.
In this, I had to figure out the RBP register value first. The ret value is 8 bytes after the RBP value. I also determined the address of the Buffer using the same command in the previous task. Hence the offset was calculated as the difference of these two.
![part1-task2-img1](Artifacts/hr2340-screenshot7.png)

The next part was to figure out where to have our malicious code in the payload. After attending the Office Hours, I understood that we have to have our malicious code near the beginning of the payload ( as opposed to the previous task, where we had it towards the end of it).
So I decided to end my malicious code by the 100th address, hence the start of it was 100 - length of the shellcode.
Now to figure out the return address, I used the command ”x/500 buffer”, that listed me a bunch of addresses which could be a possible point such that our malicious code is executed.
![part1-task2-img1](Artifacts/hr2340-screenshot8.png)

After trial and error, I found the specific address which can be used to execute our malicious code. The output after the successful execution is as follows -
![part1-task2-img1](Artifacts/hr2340-screenshot9.png)

The exploit code is as follows -
![part1-task2-img1](Artifacts/exploit64.png)

## Task 7 : Defeating Dash Counter Measure

To achieve this task, I linked /bin/dash to /bin/sh.
I ran the previous exploits and noticed that I was not able to get root shell.
![part1-task2-img1](Artifacts/hr2340-screenshot11.png)

I further compiled the binaries by using the setuid flag on, using the command “make setuid”. Even after this, running the shell didnt provide me with a root shell.
![part1-task2-img1](Artifacts/hr2340-screenshot12.png)

I then copied the binary code for setuid into the shell code of both the 64 and 32 bit binaries. After compiling these, I ran the exploits and this time, I was able to access the root shell.
![part1-task2-img1](Artifacts/hr2340-screenshot13.png)
![part1-task2-img1](Artifacts/hr2340-screenshot14.png)

	
	
	

