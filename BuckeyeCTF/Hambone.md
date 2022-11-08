# Hambone

## Description
I hid the flag somewhere on the website as a 48 byte hex value, so I know you'll never find it. Just, don't check out how the background is calculated.

https://hambone.chall.pwnoh.io

Note: the flag will be in lowercase, not uppercase

File: ``dist.py``

```py
def get_distances(padded_url : str, flag_path : str):
    distances = []
    for i in range(3):
        # calculate hamming distance on 16 byte subgroups
        flag_subgroup = flag_path[i*32:i*32+32]
                
        z = int(padded_url[i*32:i*32+32], 16)^int(flag_subgroup, 16)
        distances.append(bin(z).count('1'))  
        
    return distances
```

## Solution

Here is an annotated version of the code.

```py
# Calculates the distance for the website background
# @param padded_url: The path in the request url; 96 length hexadecimal string
# @param flag_path: The path to get the flag; 96 length hexadecimal string
def get_distances(padded_url : str, flag_path : str):
    distances = [] # The distances for the background
    for i in range(3): # RGB Triplet
        # calculate hamming distance on 16 byte subgroups
        flag_subgroup = flag_path[i*32:i*32+32] # Gets 32 chars of the string based on which iteration it is on
                
        z = int(padded_url[i*32:i*32+32], 16)^int(flag_subgroup, 16) # Gets the bit difference between the current 32 hex section and the flag 32 hex section
        distances.append(bin(z).count('1')) # Adds the number of 1's to the distance
        
    return distances # Returns the distance array
```

The path for the flag url and the solution url should be in this format
```
             32 chars                         32 chars                         32 chars           
╭──────────────────────────────╮ ╭──────────────────────────────╮ ╭──────────────────────────────╮
ac72c3ecbd95984a48a1890735da8c10 b7dd222b9addf2ab7b17778c6b8fc353 7852861c969f6738865996481438b29d
```
The path's binary length is 384 and since the binary is being compared, the first step is to use binary then convert to hex before testing the string.  
The python operator ``^`` compares the binary of two integers.  
This is the exploit we can use to guess and check bits in our path.  
Here is an example:
```
11000
01011
----- XOR Operator
10011
```
If the two bits are the same the binary of the background will have a 0 in that place and if they are different the binary will have a 1.  
Since the background is calculated based off of how many 1's are in the binary difference, we can first check the background for all 0s then change each bit one by one and observing the change that occurs in the background. If the background hex increased, that means the bit we changed was wrong, if the background hex value decreases that means that the bit we changed was correct.

Example:
```
Zero path: 000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
Correct path: ac72c3ecbd95984a48a1890735da8c10b7dd222b9addf2ab7b17778c6b8fc3537852861c969f6738865996481438b29d
```
Binary:
```
000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
101011000111001011000011111011001011110110010101100110000100101001001000101000011000100100000111001101011101101010001100000100001011011111011101001000100010101110011010110111011111001010101011011110110001011101110111100011000110101110001111110000110101001101111000010100101000011000011100100101101001111101100111001110001000011001011001100101100100100000010100001110001011001010011101
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
101011000111001011000011111011001011110110010101100110000100101001001000101000011000100100000111001101011101101010001100000100001011011111011101001000100010101110011010110111011111001010101011011110110001011101110111100011000110101110001111110000110101001101111000010100101000011000011100100101101001111101100111001110001000011001011001100101100100100000010100001110001011001010011101
```
Number of 1s: 190
```
000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001
101011000111001011000011111011001011110110010101100110000100101001001000101000011000100100000111001101011101101010001100000100001011011111011101001000100010101110011010110111011111001010101011011110110001011101110111100011000110101110001111110000110101001101111000010100101000011000011100100101101001111101100111001110001000011001011001100101100100100000010100001110001011001010011101
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
101011000111001011000011111011001011110110010101100110000100101001001000101000011000100100000111001101011101101010001100000100001011011111011101001000100010101110011010110111011111001010101011011110110001011101110111100011000110101110001111110000110101001101111000010100101000011000011100100101101001111101100111001110001000011001011001100101100100100000010100001110001011001010011100
```
Number of 1s: 189  
By testing each of the 384 bits one by one, we can find the value of each bit by checking if the number of 0s decreases or increases.  
Here is a small helper function to test our solutions using the python requests package:
```py
def get_background(path : str):
    req = requests.get(f"https://hambone.chall.pwnoh.io/{path}")
    background = req.text.split("background: #")[1].split("\"")[0]
    return int(background, 16)
```
Here is the final solution:
```python
import requests

# Calculates the distance for the website background
# @param padded_url: The path in the request url; 96 length hexadecimal string
# @param flag_path: The path to get the flag; 96 length hexadecimal string
def get_distances(padded_url : str, flag_path : str):
    distances = [] # The distances for the background
    for i in range(3): # RGB Triplet
        # calculate hamming distance on 16 byte subgroups
        flag_subgroup = flag_path[i*32:i*32+32] # Gets 32 chars of the string based on which iteration it is on
                
        z = int(padded_url[i*32:i*32+32], 16)^int(flag_subgroup, 16) # Gets the bit difference between the current 32 hex section and the flag 32 hex section
        distances.append(bin(z).count('1')) # Adds the number of 1's to the distance
        
    return distances # Returns the distance array
    
def get_background(path : str):
    req = requests.get(f"https://hambone.chall.pwnoh.io/{path}")
    background = req.text.split("background: #")[1].split("\"")[0]
    return int(background, 16)

# The background when the value of the path is 0
zero_bg = "c6b4c5"

curr_path = ""

for i in range(384):
    path = "0"*(383-i)+"1"+"0"*i
    dis = hex(int(path, 2)).split("0x")[1].zfill(96)
    print(f"Currently testing: {dis}")
    result = get_background(dis)
    zero = int(zero_bg, 16)
    current = result

    if current > zero:
        curr_path = "1"+curr_path
    else:
        curr_path = "0"+curr_path


print(curr_path)
```
Output:
![](https://i.mrxbox98.me/file/2022/11/WindowsTerminal_pGQ4KOPO6G.gif)
At the end the program will output ``101011000111001011000011111011001011110110010101100110000100101001001000101000011000100100000111001101011101101010001100000100001011011111011101001000100010101110011010110111011111001010101011011110110001011101110111100011000110101110001111110000110101001101111000010100101000011000011100100101101001111101100111001110001000011001011001100101100100100000010100001110001011001010011101`` which converted into hex is ``ac72c3ecbd95984a48a1890735da8c10b7dd222b9addf2ab7b17778c6b8fc3537852861c969f6738865996481438b29d`` which means the final path is https://hambone.chall.pwnoh.io/ac72c3ecbd95984a48a1890735da8c10b7dd222b9addf2ab7b17778c6b8fc3537852861c969f6738865996481438b29d  
Chrome and some other browsers sometimes autocapitalize the path, but running 
```sh
wget https://hambone.chall.pwnoh.io/ac72c3ecbd95984a48a1890735da8c10b7dd222b9addf2ab7b17778c6b8fc3537852861c969f6738865996481438b29d
```
creates a file with the contents:
```html
<!doctype html>
<html>
  <head>

    <meta charset="utf-8">
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.2.2/dist/js/bootstrap.bundle.min.js" integrity="sha384-OERcA2EqjJCMA+/3y+gxIOqMEjwtxJY7qPCqsdltbNJuaOe923+mo//f6V8Qbsw3" crossorigin="anonymous"></script>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.2.2/dist/css/bootstrap.min.css" integrity="sha384-Zenh87qX5JnK2Jl0vWa8Ck2rdkQ2Bzep5IDxbcnCeuOxjzrPF/et3URy9Bv1WTRi" crossorigin="anonymous">
    <title></title>

  </head>
  <body style="background: #ffffff">
    <div id="content">
        <div class='container-fluid my-5'>
            <p style="color:#000000"> Whoa! How&#39;d you find this? Guess I owe you the flag: buckeye{th3_b4ckgr0und_i5_n0t_4_l13} </p>
        </div>
    </div>
    <div>
  </body>
</html>
```
so the flag is ``buckeye{th3_b4ckgr0und_i5_n0t_4_l13}``
