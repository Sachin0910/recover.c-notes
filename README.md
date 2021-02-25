# recover.c-notes
Detailed notes on how to approach Harvard CS50x assignment recover.c. Hope they help.

Accidentally deleted the pictures, we are to recover those images.

1)Open memory card 
2)Look for the beginning of the JPEG file
3)Once found, you will open a new JPEG file 
4)And keep writing data of 512 Bytes, until I find a new JPEG file
5)Close the old one, and start writing a new one
6)Repeat this process until I reach the end of the file.

Opening a memory card file
FILE *file = fopen(“card.raw”, "r");
R stands for read in file
As we read in the file, we want to be on the lookout for the JPEGs.
How are we going to know if an image is a JPEG?
Every image has a distinct header
First byte: “0xff”
Second byte: “0xd8”
Third byte: “0xff”
Fourth byte: anything that starts with “0xe”

If I notice the beginning of a file with these four bytes then I can assume that it is a JPEG file



JPEGs
Each JPEG will start with a distinct header.
First byte: “0xff”
Second byte: “0xd8”
Third byte: “0xff”
Fourth byte: anything that starts with “0xe”

NOTE: This is the header that we are going to be looking for.

All JPEGs are stored back to back in the memory card.
After one ends, another starts until the end of the file.

Each block is 512 bytes.


How do you look through the memory card for JPEGs?

Think of the memory card as a whole bunch of these 512 bytes blocks. 

Start at the first block, and check if you found the JPEG header. One by one.
If not, then go to the next block, until you find it.
Once found, 
that block is the beginning of the JPEG file.
Open up a new JPEG file that I will start writing to.
Start writing block after block after block as I find more and more of this JPEG file.
Eventually, I will run into another block with a header that has the start of the JPEG file.
Once I detect that
Realize that this is a start of a new JPEG file
Close the last file,
Open a new file to write into which the next sequence of data will go.

Repeat this process 
of finding a JPEG header,
 and writing it a new file.
Closing it.
Do this until I reach the end of the memory card.
By this time, we would have found all the JPEG files.
 
How am I going to read data from the memory card?

To do that we use the fread function
fread(data, size, number, inptr);
Take four-parameter.
Data: pointer where we are storing data that we are reading. Use a “buffer” as an array.
Size: the size of each element to read
Number: Number of each element to read.
inptr* : FILE* to read from
Recall that you want to read from the memory card file in 512 bytes chunk.
Once read the 512-byte chunk, how do we figure out if it is a start of a JPEG file?

Recall the file has a distinct header
If I have read the 512 bytes chunk into a buffer of some sort of array, now check if:
buffer[0]: “0xff”
buffer[1]: “0xd8”
buffer[2]: “0xff”
Checking the fourth byte will be a little bit harder
Use bitwise arithmetic
(Buffer[3] & 0xf0) == 0xe0, which means that just look at the first four bits of this 8-bit byte, and seat the remaining four bits to 0. This means that 0xe0, 0xe1, 0xe2 all become 0xe0 because we have cleared out the last four bits.
Now we compare the result to 0xe0 to check if this byte is the byte that appears as the fourth byte inside the JPEG. (6 min: 15 seconds of the video.)
If all of the following conditions are true, then it is pretty likely that the 512-byte block that I found is the beginning of the new JPEG file. 
Once found the start of the JPEG file, create a new file where I will write this data to. 



How do you make a new JPEG file?

Each file should have ###.jpeg, starting with 000.jpeg. Then increment.
Keep order of how many JPEGs you found, so you can write them in the correct order.

How Do you create a string of the format ###.jpeg?

Use “sprintf”
sprintf(filename, “%.03i.jpg”, 2);
Sprintf is not printing to the terminal, but to a string, 
The first parameter is the name of the string you want to write to.
The second parameter is the format string, “%.03i” means print an integer with three digits to represent it
The last parameter is the name of the file, in our case above, the file name will be 002.jpg.
Make sure that the file name has enough memory, has enough characters, to represent the entire file name. 
After creating the file name, you can open a new file with  that file name,
FILE *img = fopen(filename, “w”);
The first argument is the name of the file you want to open. 
The second argument is telling us that we are going to write in it. “W”. So we can begin to write to the new file all the data that I will find in the memory card.

How do you write data to a file?

Use fwrite(data, size, number, outptr);
Take four-parameter.
Data: pointer to bytes that will be written to the file.
Size: the size of each element to write.
Number: Number of elements to write.
outptr*: FILE* to write from. Likely the JPEG that you just opened for the purpose of writing to it.
Continue writing data until I reach the end of the file.

How do you detect when the end of the file is?

Let’s look at fread.
fread(data, size, number, inptr);
Take four-parameter.
Data: pointer where we are storing data that we are reading. Use a “buffer” as an array.
Size: the size of each element to read
Number: Number of each element to read.
inptr* : FILE* to read from
What does fread return?
It returns the number of items of size “size” were read.
We currently try to read the number elements of size “size”. So most likely if fread is able to read all the data, it will return to me a number. If I am trying to read 255 elements, it will return to me 255. 
Once we come to reach the end, we might not have 255 bytes to read and have fewer than that. In that case, fread will return a number less than 255.
Write some conditions to determine if the fread has gotten to the end of the file or not. 

PseudoCode to help get started:

Open memory card
Repeat until the end of the card:
Read 512 bytes(using fread) into a buffer(some space in the computer memory that I have 512 bytes worth of storage that I can read that data into).
If a start of a JPEG(detect using the first four bytes to determine if it a jPEG header or not) 
If the first JPEG, 
Write the 000.jpeg, and starting the very first file.
Else(if you have already found the JPEG)
Close the file that you have been writing to, and open up the new file that I will continue writing to.
This is what to do if the 512 bytes you read is the start of a new JPEG.
Else(What to do if it is not the start of a new JPEG, what should you do instead?)
If already found JPEG, and you have been writing to it, then you should keep writing to it. This is the next 512 bytes block of this current JPEG that I have been writing to. We might repeat this process multiple times, because every JPEG might take up multiple blocks of memory inside of the memory card. 
After reaching the end of the memory card, Close any remaining files. Now you should see that I have generated a number of JPEG files, each of which contains image data that you can open up and view visually. 

Repeat this process
Reading 512 bytes and then checking. 
If it is the start of a new jpeg, then you might need to do something.
Else if it is not the start of a new jpeg, then you might need to do something else. 



