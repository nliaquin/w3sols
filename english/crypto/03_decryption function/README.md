# Challenge
You are provided with the following encryption function: *f(x)=21x+11 [26]*
First, you have to encrypt the word *GOOGLE* with this function.
Then, your mission is to find the decryption function **g**.
Finally, you have to decrypt the word *GELKT*.

Details:
1. [26] means "modulo 26".
2. The decryption function will be of the following form: g(y)=ay+b[26], with a and b natural integers lower than 26, and y=f(x).
3. You will enter the answer lowercase in the form
(crypted word)_(decryption function)_(decrypted word)
4. Example:
If the encryption of GOOGLE was BTEVER, that the function was g(y)=15y+3[26], and that the decryption of GELKT was SUPER, then the answer would be: btever_15y+3[26]_super
5. If you don't do it with maths, make sure that your function works for any input


# Guide
We're going to use Python to solve this challenge.

Let's create a map to convert letters to numbers, where A=0 ... Z=25
```python
import string

alphabet = string.ascii_uppercase
letter_to_index = {letter: index for index, letter in enumerate(alphabet)}
index_to_letter = {index: letter for index, letter in enumerate(alphabet)}

# Test with the following print statement:
print(letter_to_index)
```

Now we need to encrypt the word "GOOGLE"
We're going to add some code below the last codeblock to do the following:
1. Convert the letters in *GOOGLE* to their correspending indices
2. Apply the function *f(x) = (21x + 11) mod 26* to each index
3. Convert the result back to letters
```python
word = "GOOGLE"
encrypted_indices = [encrypt_char(letter_to_index[char]) for char in word]
encrypted_word = ''.join(index_to_letter[i] for i in encrypted_indices)
# This should result in [7, 19, 19, 7, 8, 17] printing out
print(encrypted_indices)
# This should result in HTTHIR printing out
print(encrypted_word)
```

We must now find the modular inverse of *21 mod 26*. We could simply import mod_inverse from sympy, or even write a function.
```python
from sympy import mod_inverse
a_inv = mod_inverse(21, 26)
# Should result in printing 5
print(a_inv)
```
If you want to know what the mod_inverse function basically looks like:
```python
def mod_inverse(a, mod):
    for i in range(1, mod):
        if (a * i) % mod == 1:
            return i
    return None  # No inverse exists
```

This means g(y) = ***5***y + 23 mod 26. We can now write a decryption function in Python:
```python
def decrypt_char(y):
    return (5 * y + 23) % 26
```

Using this function, we're going to convert each letter in *GELKT* to its alphabetical index, then apply the decryption function to each index, and finally convert the results back into alphabetical characters.
```python
cipher_text = "GELKT"
cipher_indices = [letter_to_index[char] for char in cipher_text]
print(cipher_indices)
# Prints [6, 4, 11, 10, 19]

decrypted_indices = [decrypt_char(i) for i in cipher_indices]
print(decrypted_indices)
# Prints [1, 17, 0, 21, 14]

decrypted_word = ''.join(index_to_letter[i] for i in decrypted_indices)
print(decrypted_word)
# Prints BRAVO
```

#### Solution File: function.txt
