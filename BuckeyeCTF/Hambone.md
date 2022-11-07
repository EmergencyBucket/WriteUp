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
The path's binary length is 384 and since the binary is being compared, the first step is to use binary then convert to hex befor testing the string.  
The python operator ``^`` compares the binary of two integers.  
This is the exploit we can use to guess and check bits in our path.  
Here is an example:
```
11000
01011
----- Caret Operator
10011
```
