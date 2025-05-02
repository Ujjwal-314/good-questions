If the markdown rendering of the question is not correct, [click here]()

I was learning about path traversal vulnerability, and i got reference to this [webpage](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Directory%20Traversal/README.md) 
. In the [overlong encoding section](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Directory%20Traversal/README.md#overlong-utf-8-unicode-encoding) , i got this table, 
[![overlong encoding on PayloadAllTheThings][1]][1]


  [1]: https://i.sstatic.net/6HqGTu0B.png

The first 2 encoding of . and / seems correct to me, they are doing overlong encoding paired with bits sequence change (learnt from this [answer](https://security.stackexchange.com/questions/48879/why-does-directory-traversal-attack-c0af-work)). 

I created my own table to understand this, 

| character               | binary representation           | hexadecimal rep | Description                                                  |
| ----------------------- | ------------------------------- | --------------- | ------------------------------------------------------------ |
| \ 1-byte-UTF-8 encoding | 0101 1100                       | 5C              |                                                              |
| \ 2-byte-encoding       | 1100 0001  1001 1100            | C1 9C           | creating overlong-encoding, it is invalid but used to bypass |
| \ 2-byte-encoding       | 1100 0001  0101 1100            | C1 5C           | changing bits sequence, invalid but used to bypass           |
| \ 2-byte-encoding       | 1100 0001  0001 1100            | C1 1C           | again changing bits sequence                                 |
| \ 2-byte-encoding       | 1100 0001  1101 1100            | C1 DC           | again changing bits sequence                                 |
|                         |                                 |                 |                                                              |
| \ 3-byte-encoding       | 1110 0000  1000 0001  1001 1100 | E0 81 9C        | overlong-encoding of \ with 3 byte                           |

We can further change the first 2 bits sequence, but it will become very large, In PayloadAllTheThing's page, we had C0 80 5C, but ours is E0 81 9C, both are not same. Giving them benefit of doubt, they maybe changing the bits sequence, but even the first byte is not matching, which seems wrong at this point, even if they were changing the bits-sequence, they should have changed the first 2 bits of 2nd or 3rd byte, 
it would then looked like


| 1110 0000  1000 0001  1001 1100         | E0 81 9C |
| --------------------------------------- | -------- |
| 1110 0000  **10**00 0001  **01**01 1100 | E0 81 5C |
| 1110 0000  **10**00 0001  **00**01 1100 | E0 81 1C |
| 1110 0000  **10**00 0001  **11**01 1100 | E0 81 DC |
| 1110 0000  **01**00 0001  **10**01 1100 | E0 41 9C |
| 1110 0000  **01**00 0001  **01**01 1100 | E0 41 5C |
| 1110 0000  **01**00 0001  **00**01 1100 | E0 41 1C |
| 1110 0000  **01**00 0001  **11**01 1100 | E0 41 DC |
| 1110 0000  **00**00 0001  **10**01 1100 | E0 01 9C |
| 1110 0000  **00**00 0001  **01**01 1100 | E0 01 5C |
| 1110 0000  **00**00 0001  **00**01 1100 | E0 01 1C |
| 1110 0000  **00**00 0001  **11**01 1100 | E0 01 DC |
| 1110 0000  **11**00 0001  **10**01 1100 | E0 C1 9C |
| 1110 0000  **11**00 0001  **01**01 1100 | E0 C1 5C |
| 1110 0000  **11**00 0001  **00**01 1100 | E0 C1 1C |
| 1110 0000  **11**00 0001  **11**01 1100 | E0 C1 DC |
Visually, it is very clear that none of our values are matching with theirs.
I understand, all of this wasn't necessary, but just to give you visual idea, i did this hardwork.

QUESTION: what is the logic behind PayloadAllTheThings encoding of backslash(`\`), mine didn't matched with his. Or am i wrong somewhere.