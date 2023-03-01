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

