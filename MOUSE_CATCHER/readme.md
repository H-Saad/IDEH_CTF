# WRITEUP FOR MOUSE CATCHER CHALLENGE IDEH CTF

## DESCRIPTION
Mouse Catcher
Created By m4rc0s

Category: Forensics <br>
Difficulty: Medium <br>
Points: 50 <br>

The identity of the hacker is found, unfortunately my keyboard was not working, so I wrote his name using my mouse.
Wrap the flag in CRISIS{..}

<a href="https://s3.eu-west-3.amazonaws.com/crisis-assets/crisis_attachements/RmigPbf1mZOAMFJix7wResmGoye7rws4cjPZcZUJ.zip">Download attachement</a>

## DETAILED SOLUTION

We have a .pcap file and from the challenge title we will probably need to reconstruct mouse movement captured to draw the flag.

After opening the attachement with Wireshark we find a bunch of DESCRIPTION/CONFIGURATION requests followed by URB_INTERRUPT packets.

The later contains the usb data that we want to extract
![capt1](https://user-images.githubusercontent.com/61362146/159765745-a73a45ac-e809-4a00-97f6-9b6e4a07d5de.PNG)

Before we do that lets find out more about the type of input device used

To do that we open one of the URB_INTERRUPT packet and look for device address
![capt2](https://user-images.githubusercontent.com/61362146/159765869-6eaa1605-bb45-44cb-9593-8fe44b92d99a.PNG)

Then we filter data to keep only packets related to the device id 5 by applying the filter:

<pre><code> usb.device_address==5  </code></pre>

We open the GET DESCRIPTOR RESPONSE DEVICE packet and look for idVendor and idProduct fields

![capt3](https://user-images.githubusercontent.com/61362146/159767060-b3dc1e02-5285-4630-aa78-5b145eb3cb53.PNG)

We look up the ids in <a href="https://devicehunt.com/">Device hunt website</a> and find out that the device is a <b>Logitech Unifying Reciever</b>

So we know that we are looking at wireless logitech mouse data.

Lets extract the HID data with tshark:

<pre><code>thshark -2 -r mouse.pcap -R 'usb.src=="1.5.2" and usb.device_address==5 and !(usbhid.data=="0000000000000000")' -T fields -e usbhid.data > data.txt
</code></pre>

We find out that the data is coded in 8 bytes which is a bit odd since standard mouse movement packets are normally coded in 4 bytes 

<a href="https://wiki.osdev.org/Mouse_Input#Format_of_First_3_Packet_Bytes">Article</a>

Now lets examine data and look for bytes that are constantly changing, those bytes will probably contain our X and Y coordinates and mouse button clicks

<pre><code>0201000000000000
02010000f0ff0000
020100ff0f000000
020100ff0f000000
02010000f0ff0000
02010000f0ff0000</code></pre>

We can see that only the 2nd 4th 5th and 6th bytes are changing

The 2nd bytes takes either 00 or 01 so let's assume that it has something to do with left mouse button being clicked or not

Now we are left with 3 more bytes to look at which is weird since we only need 1 byte for X coordinates and another for Y

We can also see that the 6th byte takes either 00 or ff

So after long and painful googling i finally found this <a href="https://ubisec.cse.buffalo.edu/files/07061471.pdf">research paper</a> that talks about <b>Password Extraction via Reconstructed Wireless Mouse Trajectory</b>

This <a href="https://www.epanorama.net/documents/pc/mouse.html">article</a> will also help you understand the logitech protocol

This paragraph has really interesting infos about the logitech wireless mouse raw data packet

![atric](https://user-images.githubusercontent.com/61362146/159770549-66c35bbc-38a7-4f64-ad4e-93add4c59805.PNG)

We find out that logitech uses a unique algorithm to process X and Y coordinates as seen in the paper

Algorithm:

<pre><code>Require: HASH = ( F → 16, E → 32, D → 48, C → 64, B → 80, A
→ 96);
1: if (XO >= 127 in decimal) then #Left movement
2: X = HASH[first digit of XO] - second digit of XO;
3: else #right movement
4: X = XO;
5: end if
6: if (first digit of YO,2 == F) then #Up movement
7: Y = HASH[second digit of YO,2] - first digit of YO,1;
8: else #Down movement
9: if (YO,2 == 00) then
10: Y = first digit of YO,1;
11: else
12: Y = result of concatenating second digit of YO,2 with
first digit of YO,1;
13: end if
14: end if
</code></pre>

From the <a href="https://wiki.osdev.org/Mouse_Input#Format_of_First_3_Packet_Bytes">article</a> cited above we take the following paragraph

<pre><code> The second byte is the "delta X" value -- that is, it measures horizontal mouse movement, with left being negative. The third byte is "delta Y", with down (toward the user) being negative</code></pre>

Now we also know that we need to multiply our return values in case of left or down movement by -1 to obtain accurate results

Lets code the solution and extract the X and Y coordinates then redraw the mouse movement using gnuplot

My python code:

<pre><code>def hash_l(c): #hash function defined in the algorithm
	c = c.lower()
	return {
		'a': 96,
		'b': 80,
		'c': 64,
		'd': 48,
		'e': 32,
		'f': 16
	}[c]

def get_x(x0):
	if(int(x0,16)>=127):
		return -(hash_l(x0[0:1]) - int(x0[1:2],16))
	else:
		return int(x0,16)

def get_y(y01,y02):
	y01 = y01.lower()
	y02 = y02.lower()
	if(y02[0:1]=='f'):
		return (hash_l(y02[1:2]) - int(y01[0:1],16))
	else:
		if(y02=="00"):
			return -int(y01[0:1],16)
		else:
			temp = str(y02[1:2]) + str(y01[0:1])
			return -int(temp,16)

o = open("cord.txt","a")
y=0
x=0
with open("data.txt") as d:
	for i in d.read().splitlines():
		x += get_x(i[6:8])
		y += get_y(i[8:10],i[10:12])
		if(i[2:4]!="00"): #only write to file if the left mouse button is clicked
			o.write(str(x) + " " + str(y) + "\n")
o.close()</code></pre>

Now use gnuplot to draw the coordinates

![gnu](https://user-images.githubusercontent.com/61362146/159772445-123cc03f-b236-40ce-8e37-a3271ec5ff60.PNG)

And finally we obtain the following image

![flag](https://user-images.githubusercontent.com/61362146/159772497-dc2c0759-b3dd-4133-b0f9-545926dc1cf0.png)

<pre><code>M4df63e</code></pre>

We wrap this in CRISIS{} and we obtain our flag

<pre><code>CRISIS{M4df63e}</code></pre>

Thats it and i hope this was helpful
