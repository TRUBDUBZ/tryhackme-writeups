[[CTF Notes]]

---
## Task 1: "What does the base said?"

- The challenge asks us to decode `VEhNe2p1NTdfZDNjMGQzXzdoM19iNDUzfQ==`

- `echo 'VEhNe2p1NTdfZDNjMGQzXzdoM19iNDUzfQ==' | base64 -d`

## Task 2: "Meta Meta"

- The challenge gives us a jpeg to download `findme.jpg`

- `exiftool findme.jpg` returns a list of metadata for the image. 

- Under the `Owner Name` data we find the flag

## Task 3: "Mon, are we going to be okay?"

- Challenge description states: "Something is hiding. That's all you need to know"

- We are given another jpeg to download: `Extinction.jpg`

- `steghide extract -sf Extinction.jpg` and no passphrase, gives us a textfile `Final_message.txt`

- `cat Final_message.txt` gives us the flag

## Task 4: "Erm......Magick"

- This one is super easy. In the challenge description 'Huh, where is the flag? THM{wh173_fl46}' but the flag is covered in white. 

- Just highlight the white block and you can read the flag, copy and paste and we are all done with this one!

## Task 5: "QRrrrr"

- Another pretty simple one, we are given a QR code to download `QR.png`

- `zbarimg QR.png` returns the flag

## Task 6: "Reverse it or read it?"

- We are given a file to download `hello_1577977122465.hello`

- `strings hello_1577977122465.hello | less` gives us the flag

## Task 7: "Another decoding stuff"

- The challenge gives us a string to decode `3agrSy1CewF9v8ukcSkPSYm3oKUoByUpKG4L`

- After a bit of research, we find that this is a `base58` string

- `echo '3agrSy1CewF9v8ukcSkPSYm3oKUoByUpKG4L' | base58 -d` returns the flag

## Task 8: "Left or right"

- The challenge gives us a clue "Left, right, left, right... Rot 13 is too mainstream. Solve this"

- `MAF{atbe_max_vtxltk}`

- Throw this into cyberchef and use the `ROT13` module

- Use the default settings and lower the amount down from 13, when you reach amount: 7 you will get the flag

## Task 9: "Make a comment"

- "No downloadable file, no ciphered or encoded text. Huh ......."

- The hint for this one says to check the HTML

- After some searching we find a `<p style="display:none;">`

- Opening this gives us our flag

## Task 10: "Can you fix it?"

- "I accidentaly messed up with this PNG file. Can you help me fix it? Thanks, ^^"

- Download the png file and dump it as a hex `xxd -p spoil.png > spoil_hex_data`

- Fix the corrupted header: `2333445f0d0a1a0a0000000d4948445200000320000003200806000000db` -> `89504E470D0A1A0A0000000d4948445200000320000003200806000000db`

- Now that the png is restored with the correct header we can rebuild the image `xxd -r -p spoil_hex_data > fixed.png`

- Finally, open the png to get the flag `xdg-open fixed.png`

## Task 11: "Read it"

- "Some hidden flag inside Tryhackme social account."

- The only hint here says "reddit"

- Using the CTF authors username and a bit of google dorking we find the flag

- `site:reddit.com intext:DesKel intext:THM{`

## Task 13: "Spin my head"

- "What is this?

++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>>++++++++++++++.------------.+++++.>+++++++++++++++++++++++.<<++++++++++++++++++.>>-------------------.---------.++++++++++++++.++++++++++++.<++++++++++++++++++.+++++++++.<+++.+.>----.>++++."

- ChatGPT tell us this is from an esoteric programming language known as 'Brainfuck' 

- Using the site `https://copy.sh/brainfuck` we can copy and paste the code and run it to get the flag

## Task 14: "An exclusive!"

- "Exclusive strings for everyone!"

    - `S1: 44585d6b2368737c65252166234f20626d`

    - `S2: 1010101010101010101010101010101010`

- Looks to be an XOR challenge. 

```python
s1 = bytes.fromhex("44585d6b2368737c65252166234f20626d")
s2 = bytes.fromhex("1010101010101010101010101010101010")

result = bytes(a ^ b for a, b in zip (s1, s2))

print(result.decode('utf-8', errors='ignore'))
``` 
- Running this gives us our flag

## Task 15: "Binary walk"

- "Please exfiltrate my file :)"

- We are given a file to download: `hell.jpg`

- Since the task is named 'Binary walk' we will use `binwalk hell.jpg`

- We are shown a few embedded files. Extract them with `binwalk -e hell.jpg`

- Binwalk spotted a ZIP archive embedded inside the JPG starting at offset `0x40E75`, containing a file named `hello_there.txt` 

- Next we can extract the ZIP archive using `dd if=hell.jpg of=embedded.zip bs=1 skip=$((0x40E75))`

- Unzip the extracted file `unzip embedded.zip`

- This gives us a textfile `hello_there.txt` 

- `cat hello_there.txt` gives us the flag

## Task 16: "Darkness"

- "There is something lurking in the dark."

- We are given a PNG 'dark.png'

- Using an online stegsolver `https://georgeom.net/StegOnline/image` and selecting the `LSB half` setting allows us to just barely see the flag

## Task 17: "A sounding QR"

- "How good is your listening skill?"

- We are given another QR code to download `QRCTF.png`

- Toss this into an online QR code reader `zxing.org` returns a soundcloud link 

- Listening to the audio on soundcloud gives us the flag

## Task 18: "Dig up the past"

- "Sometimes we need a 'machine' to dig the past"

    - Targetted website: https://www.embeddedhacker.com/
        
    - Targetted time: 2 January 2020

- Using the webpage `waybackmachine.org` and going to the snapshot from the given date `2 January 2020`, scroll down a bit and find the flag

## Task 19: "Uncrackable!"

- "Can you solve the following? By the way, I lost the key. Sorry >.<"

    - MYKAHODTQ{RVG_YVGGK_FAL_WXF}

- The hint states "find the key of the vignere cypher"

- Toss the string into `cyberchef` and use the `Vignere Decode` module

- We guessed the key `THM` and we were able to decode the flag

## Task 20: "Small bases"

- "Decode the following text:"

    - 581695969015253365094191591547859387620042736036246486373595515576333693

- Hint: 'dec -> hex -> ascii'

- Write a simple python script to decode

```python
# Step 1: Big decimal number
num = 581695969015253365094191591547859387620042736036246486373595515576333693

# Step 2: Convert decimal to hex (remove the '0x' prefix)
hex_str = hex(num)[2:]

# Step 3: Convert hex string to bytes
byte_data = bytes.fromhex(hex_str)

# Step 4: Decode bytes to ASCII
decoded = byte_data.decode('utf-8')

# Print the result
print(f"Flag: THM{{{decoded}}}")
``` 
- Running the script returns our flag


## Task 21: "Read the packet"

- "I just hacked my neighbor's WiFi and try to capture some packet. He must be up to no good. Help me find it."

- The hint for this one says "Put it into stream. It will be much easier."

- The task gives us a file `flag.pcapng` which means it can be read using wireshark

- `wireshark flag.pcagng` opens the file in wireshark

- Using the search filter at the top we can narrow the results down using a query for `http`

- `1827 52.509987109    192.168.247.140 192.168.247.130 HTTP    455     HTTP/1.1 200 OK  (text/plain)`

- Open this up and do some searching, under the `Line-based text data: text/plain (3 lines)` drop down we find the flag

