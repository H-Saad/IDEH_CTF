# WRITEUP FOR RISSTER REVERSE CHALLENGE IDEH V4 CTF

<h3> Challenge name: Rister </h3>
<h3> Category: Reverse </h3>
<h3> Points: 100 </h3>

## DESCRIPTION

<i>To be added later</i>


## INITIAL ANALYSIS

After downloading the challenge file, we get a 64bit ELF binary that contains debug info and not stripped

![image](https://user-images.githubusercontent.com/61362146/222203357-4c881206-f7fa-4068-a529-90d3d6a3836c.png)

Executing the binary leaves us with the following results:

![image](https://user-images.githubusercontent.com/61362146/222203566-a2394010-8930-46ed-a870-44fc04bb9c04.png)

and executing the binary with arguments doesn't seem to change anything

Runing : <pre><code> strings ./Risster | grep -i "ideh" </pre></code> gives the following results:

![image](https://user-images.githubusercontent.com/61362146/222203970-5db8faa7-7da2-4fbf-b31f-5c15dcfc854f.png)

We can see a mangled flag and the goodluck text shown after execution.

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

And because I was too lazy to try to reverse what the functions does, I launched the binary in GDB so it can make the flag for me.

We breakpoint main and run the binary:

![image](https://user-images.githubusercontent.com/61362146/222207547-193faa25-5051-4783-81c2-8632dcb59659.png)

We set disassembly-flavor to intel for better readability and we dissassemble current location:

<pre><code> disass </code></pre>

![image](https://user-images.githubusercontent.com/61362146/222512945-fcf72e83-39b1-485a-a09b-7035c8d3f0bb.png)

We breakpoint this function and continue

<pre> <code> b* 0x55555555e560 </code> </pre>
<pre> <code> continue </code> </pre>
<pre> <code> disass </code> </pre>

![image](https://user-images.githubusercontent.com/61362146/222513331-0ccc68b9-bdb8-4cb1-8453-bf0df72037e9.png)

We get a lot of assembly code but we already have a decent idea of what the function does due to IDA decompilation

So I decided to breakpoint the last function that is going to be called in this context and take a look if the flag was gonna get prepared at the end in some registers

<pre> <code> b* 0x000055555555ece6 </code> </pre>

![image](https://user-images.githubusercontent.com/61362146/222513914-de76afdf-5590-4af8-87ff-c0b6fcdd5922.png)

<pre> <code> continue </code> </pre>

![image](https://user-images.githubusercontent.com/61362146/222514141-041668c7-2f58-482c-bd3a-edf027a6f830.png)

Now that we stepped into the function we can see a lot of strings that have been pushed to stack

So lets try and go down in the function step by step and see if anything interesting pops up

<pre> <code> step </code> </pre>

After some steps in the function we can see the actual flag popping up

![image](https://user-images.githubusercontent.com/61362146/222516802-e5c9b033-0f5c-4fa9-bcc7-d88ba5a9c03c.png)

And we get our flag:

<pre><code> flag: IDEH{Br4v0_3l1K_4lb4t4l_Kt4CH3F71_l0W_l3V3l_D_rUst} </code> </pre>

This was a very lazy solution as we in time of ctf didn't have enough time to go through the disassembly and understand what the code actually does, especially since it's a rust binary and takes a lot of head scratching to get through it, I will try to do a more detailed walkthrough of the binary when I get the time.

Generally it's much faster to use debugging in these kind of challenges. <br>
If you suspect that the binary will form the flag on its own in one point at it's runtime, it's usually worth it to run it with gdb and try to read what the registers have in different steps of execution.

That's it and I hope it was helpful





