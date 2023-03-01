# WRITEUP FOR RISSTER REVERSE CHALLENGE IDEH V4 CTF

## DESCRIPTION




## INITIAL ANALYSIS

After downloading the challenge file, we get a 64bit ELF binary that contains debug info and not stripped

![image](https://user-images.githubusercontent.com/61362146/222203357-4c881206-f7fa-4068-a529-90d3d6a3836c.png)

Executing the binary leaves us with the following results:

![image](https://user-images.githubusercontent.com/61362146/222203566-a2394010-8930-46ed-a870-44fc04bb9c04.png)

and executing the binary with arguments doesn't seem to change anything

Runing : <pre><code> strings ./Risster | grep -i "ideh" </pre></code> gives the following results:

![image](https://user-images.githubusercontent.com/61362146/222203970-5db8faa7-7da2-4fbf-b31f-5c15dcfc854f.png)

From this I understood that the challenge will probably be about reversing a binary that will dynamically make the flag on execution, and that the binary is written in Rust because of the hint <strong> "{main.rs}" </strong> and I knew that Im gonna have a bad time.

## Dissassembly with IDA

Runing Ida with the binary gives us the following screen:

![image](https://user-images.githubusercontent.com/61362146/222204720-0d11e088-db56-4d9a-86a2-4d16b687d053.png)

We search of main function in the functions tab and we find 2 functions:

![image](https://user-images.githubusercontent.com/61362146/222204911-fb3e7498-0650-402e-bfad-69bc1f2dd93e.png)

Dissassembly of the <stong> main </strong>function gives us the following:
  
![image](https://user-images.githubusercontent.com/61362146/222205250-05e49c94-3071-489d-bfe6-6b29bb34d473.png)

I decided then to look at the functions that are being called and found this:

![image](https://user-images.githubusercontent.com/61362146/222206249-e2c21f56-8519-4d4f-b7a2-c5d72c71f24a.png)

After analysing this function, it seems that it calls a bunch of split functions and index functions on the string that we saw earlier, so I figured that this function must be our trail toward the flag.

And because I was too lazy to try to reverse what the functions does, I launched the binary in GDB to make it make the flag for me.

We breakpoint main and run the binary:

![image](https://user-images.githubusercontent.com/61362146/222207547-193faa25-5051-4783-81c2-8632dcb59659.png)




