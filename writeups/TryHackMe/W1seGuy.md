W1seGuy — My Writeup
Spun up the machine, hit it with netcat on port 1337.

nc (target ip) and port 1337
Got back a hex string and a prompt asking for the key. Looked at the source code they provided
basically the server picks a random 5-character key (letters + digits), XORs the flag with it,
and sends the hex-encoded result. If you guess the key right, it gives you a second flag.

I stared at the hex for a bit. The flag format is always THM{ — that's 4 bytes I already know.
Opened up CyberChef.
Pasted THM{ as the input. Used the XOR recipe. For the key, I put in the first 4 bytes of the ciphertext (first 8 hex characters). 
That gave me the first 4 characters of the key. I think it was something like KDf9.

Now I just needed the 5th character. Went back to CyberChef, put in the full ciphertext,
set the key format to KDf9 plus one unknown character, and just started guessing. 
Letters, digits — tried them one by one until the output actually looked like a flag. Took maybe a minute.

Found it — the key was KDf98. The decrypted text was the first flag.

Sent the key back through netcat and got the second flag.

Clean and easy. No code, just netcat and a browser tab.
