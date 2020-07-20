---
layout:     post
title:      "RACTF Writeup | Snakes and Ladders"
date:       2020-07-19 10:07:00
categories: RACTF
---

## RACTF Reversing | Snakes and Ladders


They give us an encrypted flag: `fqtbjfub4uj_0_d00151a52523e510f3e50521814141c` and the function they used to encrypt it. We must reverse engineer the function and build another function that decrypts the flag they gave us so we can obtain the real flag. 

Studying the encryption function, we see that the flag is split into two parts: `end_text` and `hex_xored`. 

| fqtbjfub4uj_0_d | 00151a52523e510f3e50521814141c |
| :-------------: | :----------------------------: |
|    end_text     |           hex_xored            |

The algorithm they use for the encryption separates the unencrypted flag into the previous two parts by skipping every other letter. Essentially, `end_text` is a list of all the even characters, and `hex_xored` is a list of all the odd characters. 

`hex_xored` is obtained after running every odd character through the function below with the argument a string of `a`'s. Then the result is turned into a list of hex numbers (which doubles its length).

```python
def xor (s1, s2):
    return ''.join(chr(ord(a) ^ ord(b)) for a, b in zip(s1, s2))
```

If something is encrypted with xor, like so: `c = xor(a, b)`, then we can retrieve `b` with: `b = xor(a, c)`. 

Since `hex_xored` is a list of hex numbers, we have to convert that to a list of characters to re-input into `xor()`. (`hex_xored` is our `c` in this case).

```python
hex_xored = "00151a52523e510f3e50521814141c"
hex_xored = [hex_xored[i: i+2] for i in range(0, len(hex_xored), 2)] # group into pairs 
chars_xored = [chr(int(ch, base=16)) for ch in hex_xored] # convert to chars
xor("aaaaaaaaaaaaaaaa", chars_xored)
```

The result of the last expression is list of every odd character in the decrypted flag. 

Now, to decrypt `end_text`.

Looking at the algorithm, it seems to be a simple shifting cipher with the shift key a predefined random number. Luckily we have that number.

```python
for ch in end_text:
    if ch >= 'a' and c <= 'z':
        shifted_ch = chr(ord(ch) - randnum)
        if shifted_ch < 'a':
            shifted_ch = chr(ord(shifted_ch) + 26)
    result += shifted_ch
```

The above code will decrypt `end_text` into a list of the even characters of the decrypted flag.

```python
for a, b in zip(end_text_decrpyted, hex_xored_decrypted):
    print(f'{a}{b}', end='')
```

Finally, this code above will combine both texts to give us the flag.

Combining all of the previous code snippets together results in the following decryption function:

```python

def xor (s1, s2):
    return ''.join(chr(ord(a) ^ ord(b)) for a, b in zip(s1, s2))

def shift (msg, key):
    result = ""

    for ch in msg:
        if ch >= 'a' and ch <= 'z':
            shifted_ch = chr(ord(ch) - key)
            if shifted_ch < 'a':
                shifted_ch = chr(ord(shifted_ch) + 26)
        result += shifted_ch

    return result

def decrypt (msg):
    length = len(msg)
    split = int(length / 3)
        
    part1 = msg[:split]
    part2 = msg[split:]
    
    hex_xored = [part2[i:i+2] for i in range(0, len(part2), 2)]
    chars_xored = [chr(int(ch, base=16)) for ch in hex_xored]
    
    part2_decrypted = xor("a" * length, chars_xored)
    
    randnum = 14
    part1_decrypted = shift(part1, randnum)
    
    for a, b, in zip(part1_decrypted, part2_decrypted):
        print(f'{a}{b}', end='')
    print()
    
decrypt("fqtbjfub4uj_0_d00151a52523e510f3e50521814141c")
```

