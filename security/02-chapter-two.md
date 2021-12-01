# General Notes From Chapter Two


### We can characterize cryptographic system by:
* **Types of encryption operations used:**
	1. substitution
	2. transposition
	3. product (substitution and transposition)
* **Number of keys used:**
	1. single-key or private
	2. two-key o/r public
* **Way in which plaintext is processed:**
	1. block
	2. stream

## Cryptanalysis
objective to recover key not just message
* **general approaches:**
	1. **cryptanalytic attack** => relies on the nature of the algorithm plus perhaps some knowledge of the general characteristics of the plaintext or even some sample plaintext- ciphertext pairs. This type of attack exploits the characteristics of the algorithm to attempt to deduce a specific plaintext or to deduce the key being used.
	2. **brute-force attack** => try every possible key on a piece of ciphertext until an intelligible translation into plaintext is obtained. On average,half of all possible keys must be tried to achieve success. 
If either type of attack succeeds in deducing the key, the effect is catastrophic: All future and past messages encrypted with that key are compromised. 	

### Cryptanalytic Attacks
1. **ciphertext only** *{This is the most difficult problem}*
	* only know algorithm & ciphertext, is statistical, know or can identify plaintext 
2. **known plaintext** *{Generally, an encryption algorithm is designed to withstand a known-plaintext attack}*
	* Know both plaintext & ciphertext of some random message
3. **chosen plaintext**
	* select plaintext and obtain ciphertext
	* usually done by hacking the server that create the ciphertexts and injects the chosen plaintext and get the ciphertext from it
4. **chosen ciphertext**
	* select ciphertext and obtain plaintext
	* like the **chosen plaintext** but even easier
5. **chosen text** *{This is the least difficult problem}*
	* select plaintext or ciphertext to encrypt/decrypt

**unconditional security**
	* no matter how much computer power or time is available, the cipher cannot be broken since the ciphertext provides insufficient information to uniquely determine the corresponding plaintext 

**computational security**
	* given limited computing resources (e.g. time needed for calculations is greater than age of universe), the cipher cannot be broken 


### Brute-force Attack
<img src="https://github.com/ahmadateya/learning-notes/blob/main/images/Screenshot%20from%202021-11-16%2003-43-26.png" width="500" height="300">

### Additional Definitions
* **unconditional security**
	* no matter how much computer power or time is available, the cipher cannot be broken since the ciphertext provides insufficient information to uniquely determine the corresponding plaintext 
* **computational security**
	* given limited computing resources (e.g. time needed for calculations is greater than age of universe), the cipher cannot be broken 

------------------------------------------------------------------------------
### Classical Ciphers
* where letters of plaintext are replaced by other letters or by numbers or symbols
* or if plaintext is viewed as a sequence of bits, then substitution involves replacing plaintext bit patterns with ciphertext bit patterns

* **substitution Ciphers**
1. **Caesar Cipher** *{This is the simplest monoalphabetic cipher}*
	* <img src="https://github.com/ahmadateya/learning-notes/blob/main/images/Screenshot%20from%202021-11-16%2003-56-26.png" width="600" height="200">
	* **Cryptanalysis of Caesar Cipher**
		* only have 26 possible ciphers 
		* could simply try each in turn 
		* a brute force search 
		* given ciphertext, just try all shifts of letters {all values of k}

**Polyalphabetic Ciphers**
One way to improve on the simple monoalphabetic technique is to use different monoalphabetic substitutions as one proceeds through the plaintext message. The general name for this approach is polyalphabetic substitution cipher.

2. **Vigenère Cipher** *{a simple example for polyalphabetic ciphers}*
	* This scheme depends on using a set of Caesar ciphers each with a different shift value from 0 to 25.
	* To encrypt a message, a key is needed that has the same number of characters as the message.
	* Usually, the key is a repeating keyword.
		* For example, if the keyword is deceptive, the message “we are discovered save yourself” is encrypted as: 
			* key:             deceptivedeceptivedeceptive
			* plaintext:       wearediscoveredsaveyourself
			* ciphertext:      ZICVTWQNGRZGVTWAVZHCQYGLMGJ
	* **Vigenère Autokey System**
		* A keyword is concatenated with the plaintext itself to provide a running key
		* Example:
			* key: 	        **deceptive**wearediscoveredsav
			* plaintext:    wearediscoveredsaveyourself
			* ciphertext:   ZICVTWQNGKZEIIGASXSTSLVVWLA
		* Although its more powerful than the regular **Vigenère Cipher**, but even this scheme is vulnerable to cryptanalysis because the key and the plaintext share the same frequency distribution of letters, a statistical technique can be applied.
3. **Monoalphabetic Cipher**
	* rather than just shifting the alphabet, could shuffle (jumble) the letters arbitrarily 
	* each plaintext letter maps to a different random ciphertext letter , hence key is 26 letters long; a key example is shown below
		* Plain:  abcdefghijklmnopqrstuvwxyz
		* Cipher: DKVQFIBJWPESCXHTMYAUOLRGZN
	* Example
		* Plaintext:  ifwewishtoreplaceletters
		* Ciphertext: WIRFRWAJUHYFTSDVFSFUUFYA 
 
* Monoalphabetic Cipher Security
	* now have a total of 26! = 4 x 1026 keys, with so many keys, might think is secure, but the problem is language characteristics
		* human languages are redundant *{we don't actually need all the letters in order to understand written English text}*
		* some letters are not equally commonly used and some are used frequently, in English E is by far the most common letter, followed by T,R,N,I,O,A,S 

4. **One-Time Pad**
	* if a truly random key having the same length as the message is used, the cipher will be secure 
	* is unbreakable since ciphertext bears no statistical relationship to the plaintext since for any plaintext & any ciphertext there exists a key mapping one to other
	* can only use the key once though there are problems in generation & safe distribution of key
	* is useful primarily for low-bandwidth channels requiring very high security.
	* Provably secure
		* Ciphertext provides no information about plaintext
		* All plaintexts are equally likely
	* BUT only when be used correctly
		* Pad must be random, used only once
		* Pad is known only to sender and receiver
	* Note: pad (key) is same size as message
	* So, why not distribute message instead of pad? *{Answer: key may be distributed before message becomes ready}*

* **Transposition or Permutation Ciphers**
	* these hide the message by rearranging the letter order without altering the actual letters used
	* can recognise these since have the same frequency distribution as the original text 

5. **Row Transposition Ciphers**
	* Write letters of message out in rows over a specified number of columns
	* Then reorder the columns according to some key before reading off the rows
	* Example: consider a 7-digit key (digits from 1 to 7 are arranged in any required order) as follows
		* Key: 4312567
		* Column Out     4   3   1  2  5  6  7
		* Plaintext:     a   t   t  a  c  k  p
			         o   s   t  p  o  n  e
			         d   u   n  t  i  l  t
			         w   o   a  m  x  y  z
		* Ciphertext: TTNAAPTMTSUOAODWCOIXKNLYPETZ
	* If the block of the words is not finished you can pad it with *XYZ* or any dummy letters


* **Product Ciphers** A substitution followed by a transposition
	* ciphers using substitutions or transpositions are not secure because of language characteristics.
	* hence consider using several ciphers in succession to make harder, but:
		* two substitutions make a more complex substitution 
		* two transpositions make more complex transposition 
		* but a substitution followed by a transposition makes a new much harder cipher 
		* this is bridge from classical to modern ciphers,
-------------------------------------------------------------------------
### Steganography
* an alternative to encryption, hides existence of message
* using only a subset of letters/words in a longer message marked in some way using invisible ink
* hiding in LSB in graphic image or sound file
* has drawbacks
	* high overhead to hide relatively few information bits

* advantage:
	* it can obscure encryption
	* use messages can be exchanged without being noticed
* a message can be first encrypted and then hidden using steganography.
