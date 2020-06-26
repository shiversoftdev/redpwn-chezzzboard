# redpwn-chezzzboard
This challenge involves a chess board where you can play against yourself.\
Popping the executable into IDA and xref'ing the flag.txt brings us here (my idb is already labeled, sorry)\
![cida1](/img/cida1.png)\
\
It is clear that the objective of the game is to obtain a specific checksum on the board\
\
Entering the subroutine shows a double for loop and a switch, with some subroutines identifying the piece type and calculating various scores for each piece\
![cida2](/img/cida2.png)\
\
In that image I already have the enum labeled, but during the challenge I had to search for the draw function and label based on the piece strings\
![cida3](/img/cida3.png)\
\
With this information, we are able to calculate an array of winning boards using this python script
```python
for e in range(10,25):
    for a in range(1,9):
        for b in range(9, 17):
            for c in range(1, 9):
                for d in range(9, 17):
                    if ((a * b + c * d + (256 - e)) == 467):
                        print(a, b, c, d, e)
```
\
From there, its just about choosing an arbitrary valid board and writing a script to move the pieces to their spots (or you can do it by hand if you like)
```python
from pwn import *

conn = remote('2020.redpwnc.tf', 31611)

def CMove(col, _f, _t):
    print('cmd_mov(' + col + ', ' + _f + ', ' + _t + ');')
    print(conn.recvuntil(col + ': Move from').decode())
    conn.send(_f + '\n')
    print(conn.recvuntil('To').decode())
    conn.send(_t + '\n')
    return _t

def Prep():
    iswhite = True
    _from =   ['2E', '7E', '3E', '6E', '2D', '7D', '3D', '6D', '4D', '5D', '1D', '8D', '4D', '5D']
    _to =     ['3E', '6E', '4E', '5E', '3D', '6D', '4D', '5D', '5E', '4E', '4D', '5D', '4E', '5E']
    i = 0
    while i < len(_to):
        col = 'Black'
        if iswhite:
            col = 'White'
        
        CMove(col, _from[i], _to[i])
        print(col + ': ' + _from[i] + ' => ' + _to[i])
        iswhite = not iswhite
        i += 1

#8 16 8 12 13
def Slay():
    blackpos = '5E'
    whitepos = '4E'
    iswhite = True

    _wto = ['7H', '7G', '7F', '7C', '7B', '7A', '8B', '8C', '8E', '8F', '8G', '8H', '3H', '4H', '5H', '8H']
    _bto = ['2H', '2G', '2F', '2C', '2B', '2A', '1A', '1B', '1C', '1E', '1F', '1G', '1H', '1D', '8D']

    i = 0

    while i < len(_wto):

        if iswhite:
            whitepos = CMove('White', whitepos, _wto[i])
        else:
            blackpos = CMove('Black', blackpos, _bto[i])
        
        iswhite = not iswhite

        if iswhite or len(_bto) <= i:
            i += 1

def Finish():
    CMove('Black', '8A', '4A')
    return
    
Prep()
Slay()
Finish()

conn.interactive()
```
