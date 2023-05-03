Download Link: https://assignmentchef.com/product/solved-cs344-program-4-programs-that-encrypt-and-decrypt-information
<br>
In this assignment, you will be creating five small programs that encrypt and decrypt information using a one-time pad-like system. I believe that you will find the topic quite fascinating: one of your challenges will be to pull yourself away from the stories of real-world espionage and tradecraft that have used the techniques you will be implementing.

These programs serve as a capstone to what you have been learning in this course, and will combine the multi-processing code you have been learning with socket-based inter-process communication. Your programs will also accessible from the command line using standard UNIX features like input/output redirection, and job control. Finally, you will write a short compilation script.

<h1>Specifications</h1>

All execution, compiling, and testing of this program should ONLY be done in the bash prompt on our class server!

Use the following link as your primary reference on One-Time Pads (OTP):

<a href="https://en.wikipedia.org/wiki/One-time_pad">http://en.wikipedia.org/wiki/One-time_pad</a><u>​</u><a href="https://en.wikipedia.org/wiki/One-time_pad"> (Links to an external site.)Links to an external</a> <a href="https://en.wikipedia.org/wiki/One-time_pad">site.</a>

The following definitions will be important:

<strong>Plaintext</strong>​ is the term for the information that you wish to encrypt and protect. It is human readable.

<strong>Ciphertext</strong>​ is the term for the plaintext after it has been encrypted by your programs. Ciphertext is not human-readable, and in fact cannot be cracked, if the OTP system is used correctly.

A ​<strong>Key</strong>​ is the random sequence of characters that will be used to convert Plaintext to Ciphertext, and back again. It must not be re-used, or else the encryption is in danger of being broken.

The following excerpt from this Wikipedia article was captured on 2/21/2015:

“Suppose Alice wishes to send the message “HELLO” to Bob. Assume two pads of paper containing identical random sequences of letters were somehow previously produced and securely issued to both. Alice chooses the appropriate unused page from the pad. The way to do this is normally arranged for in advance, as for instance ‘use the 12th sheet on 1 May’, or ‘use the next available sheet for the next message’.

The material on the selected sheet is the key for this message. Each letter from the pad will be combined in a predetermined way with one letter of the message. (It is common, but not required, to assign each letter a numerical value, e.g., “A” is 0, “B” is 1, and so on.)

In this example, the technique is to combine the key and the message using modular addition. The numerical values of corresponding message and key letters are ​<em>added</em> together, modulo 26. So, if key material begins with “XMCKL” and the message is “HELLO”, then the coding would be done as follows:

H     E     L     L       O  message

7(H)  4(E) 11(L) 11(L)  14 (O) message

+ 23(X) 12(M)  2(C) 10(K)  11 (L) key

= 30    16    13 21   25     message + key

=  4(E) 16(Q) 13(N) 21(V)  25 (Z) message + key (mod 26)

E     Q     N     V       Z  → ciphertext

If a number is larger than 26, then the remainder, after ​<em>subtraction</em>​ of 26, is taken [as the result]. This simply means that if the computations “go past” Z, the sequence starts again at A.

The ciphertext to be sent to Bob is thus “EQNVZ”. Bob uses the matching key page and the same process, but in reverse, to obtain the plaintext. Here the key is ​<em>subtracted</em> from the ciphertext, again using modular arithmetic:

E      Q      N   V     Z  ciphertext     4(E)  16(Q)  13(N)  21(V)  25 (Z) ciphertext

-23(X)  12(M)   2(C)  10(K)  11 (L) key

= -19         4   11     11  14    ciphertext – key

=   7(H)   4(E)  11(L)  11(L)  14 (O) ciphertext – key (mod 26)

H      E      L      L       O  → message

Similar to the above, if a number is negative then 26 is ​<em>added</em>​ to make the number zero or higher.

Thus Bob recovers Alice’s plaintext, the message “HELLO”. Both Alice and Bob destroy the key sheet immediately after use, thus preventing reuse and an attack against the cipher.”

Your program will encrypt and decrypt plaintext into ciphertext, using a key, in exactly the same fashion as above, except it will be using modulo 27 operations: your 27 characters are the 26 capital letters, and the space character ( ). All 27 characters will be encrypted and decrypted as above.

To do this, you will be creating five small programs in C. Two of these will function like “daemons” (but aren’t actually daemons), and will be accessed using network sockets. Two will use the daemons to perform work, and the last is a standalone utility.

Your programs must use the network calls we’ve discussed in class (​send()​, ​recv()​, socket()​, ​bind()​, ​listen()​, &amp; ​accept()​) to send and receive sequences of bytes for the purposes of encryption and decryption by the appropriate daemons. The whole point is to use the network, even though for testing purposes we’re using the same machine: if you just ​open()​ the datafiles from the server without using the network calls, you’ll receive 0 points on the assignment. Here are the specifications of the five programs:

<strong>otp_enc_d:</strong>​ This program will run in the background as a daemon. Upon execution, otp_enc_d​ must output an error if it cannot be run due to a network error, such as the ports being unavailable. Its function is to perform the actual encoding, as described above in the Wikipedia quote. This program will listen on a particular port/socket, assigned when it is first ran (see syntax below). When a connection is made, otp_enc_d​must call ​accept()​ to generate the socket used for actual communication, and then use a separate process to handle the rest of the transaction (see below), which will occur on the newly accepted socket.

This child process of ​otp_enc_d must first check to make sure it is communicating with​ otp_enc​ (see ​otp_enc​, below). After verifying that the connection to ​otp_enc_d​ is coming from ​otp_enc​, then this child receives from ​otp_enc​ plaintext and a key via the communication socket (not the original listen socket). The ​otp_enc_d​ child will then write back the ciphertext to the ​otp_enc​ process that it is connected to via the same communication socket. Note that the key passed in must be at least as big as the plaintext.

Your version of ​otp_enc_d must support up to five ​    ​<em>concurrent</em>​ socket connections running at the same time; this is different than the number of processes that could queue up on your listening socket (which is specified in the second parameter of the listen()​ call). Again, only in the child process will the actual encryption take place, and the ciphertext be written back: the original server daemon process continues listening for new connections, not encrypting data.

In terms of creating that child process as described above, you may either create with fork()​ a ​<em>new</em>​ process every time a connection is made, ​<em>or</em>​ set up a pool of five processes at the beginning of the program, before connections are allowed, to handle your encryption tasks. As above, your system must be able to do five separate encryptions at once, using either method you choose.

Use this syntax for ​otp_enc_d​: otp_enc_d ​<em>listening_port</em>

<em>listening_port</em>​ is the port that ​otp_enc_d​ should listen on. You will always start otp_enc_d​ in the background, as follows (the port 57171 is just an example; yours should be able to use any port):

$ otp_enc_d 57171 &amp;

In all error situations, this program must output errors to stderr as appropriate (see grading script below for details), but should not crash or otherwise exit, unless the errors happen when the program is starting up (i.e. are part of the networking start up protocols like bind()). If given bad input, once running, ​otp_enc_d​ should recognize the bad input, report an error to stderr, and continue to run. Generally speaking, though, this daemon shouldn’t receive bad input, since that should be discovered and handled in the client first. All error text must be output to ​<em>stderr</em>​.

This program, and the other 3 network programs, should use “localhost” as the target IP address/host. This makes them use the actual computer they all share as the target for the networking connections.

<strong>otp_enc:</strong>​ This program connects to ​otp_enc_d​, and asks it to perform a one-time pad style encryption as detailed above. By itself, ​otp_enc​ doesn’t do the encryption – otp_end_d​ does. The syntax of ​otp_enc​is as follows:

otp_enc ​<em>plaintext key port</em>

In this syntax, ​<em>plaintext</em>​ is the name of a file in the current directory that contains the plaintext you wish to encrypt. Similarly, ​<em>key</em>​ contains the encryption key you wish to use to encrypt the text. Finally, ​<em>port</em>​ is the port that ​otp_enc​ should attempt to connect to otp_enc_d​ on.

When ​otp_enc​ receives the ciphertext back from otp_enc_d​         ​, it should output it to <em>stdout</em>​. Thus, ​otp_enc​ can be launched in any of the following methods, and should send its output appropriately:

$ otp_enc myplaintext mykey 57171

$ otp_enc myplaintext mykey 57171 &gt; myciphertext

$ otp_enc myplaintext mykey 57171 &gt; myciphertext &amp;

If ​otp_enc​ receives key or plaintext files with ANY bad characters in them, or the key file is shorter than the plaintext, then it should terminate, send appropriate error text to stderr, and set the exit value to 1.

otp_enc​ should NOT be able to connect to ​otp_dec_d​, even if it tries to connect on the correct port – you’ll need to have the programs reject each other. If this happens, otp_enc​ should report the rejection to stderr and then terminate itself. In more detail: if otp_enc​ cannot connect to the ​otp_enc_d​ server, for any reason (including that it has accidentally tried to connect to the ​otp_dec_d​ server), it should report this error to stderr with the attempted port, and set the exit value to 2. Otherwise, upon successfully running and terminating, ​otp_enc​ should set the exit value to 0.

Again, any and all error text must be output to stderr (not into the plaintext or ciphertext files).

<strong>otp_dec_d:</strong>​ This program performs exactly like ​otp_enc_d​, in syntax and usage. In this case, however, ​otp_dec_d​ will decrypt ciphertext it is given, using the passed-in ciphertext and key. Thus, it returns plaintext again to​ otp_dec​.

<strong>otp_dec:</strong>​ Similarly, this program will connect to ​otp_dec_d​ and will ask it to decrypt ciphertext using a passed-in ciphertext and key, and otherwise performs exactly like otp_enc​, and must be runnable in the same three ways. ​otp_dec​ should NOT be able to connect to ​otp_enc_d​, even if it tries to connect on the correct port – you’ll need to have the programs reject each other, as described in ​otp_enc​.

<strong>keygen:</strong>​ This program creates a key file of specified length. The characters in the file generated will be any of the 27 allowed characters, generated using the standard UNIX randomization methods. Do not create spaces every five characters, as has been historically done. Note that you specifically do not have to do any fancy random number generation: we’re not looking for cryptographically secure random number generation! rand()​ is just fine. The last character keygen outputs should be a newline. All error text must be output to ​<em>stderr</em>​, if any.

The syntax for ​keygen​ is as follows:

keygen ​<em>keylength</em>

Where ​<em>keylength</em>​ is the length of the key file in characters. ​keygen​ outputs to stdout. Here is an example run, which redirects stdout to a key file of 256 characters called “mykey” (note that mykey is 257 characters long because of the newline):

$ keygen 256 &gt; mykey

<h2>Files and Scripts</h2>

You are provided with 5 plaintext files to use (​<a href="https://oregonstate.instructure.com/courses/1662153/files/69465561/download?verifier=eYH4iTHzS5B0IvZsaCCptTYRTkajhGmCepQ2v2La&amp;wrap=1">one</a><u>​</u><a href="https://oregonstate.instructure.com/courses/1662153/files/69465561/download?verifier=eYH4iTHzS5B0IvZsaCCptTYRTkajhGmCepQ2v2La&amp;wrap=1">,</a> <u>​</u><a href="https://oregonstate.instructure.com/courses/1662153/files/69465558/download?verifier=QzTCuDEMY5zLOl2o3t1Yo5lNODVizND4xg2Me6ba&amp;wrap=1">two</a>​, <u>​</u><a href="https://oregonstate.instructure.com/courses/1662153/files/69465560/download?verifier=YjA7cJ81gwA1jLsquNJrk5YEExH0BUdQx4L2MtRT&amp;wrap=1">three</a><u>​</u><a href="https://oregonstate.instructure.com/courses/1662153/files/69465560/download?verifier=YjA7cJ81gwA1jLsquNJrk5YEExH0BUdQx4L2MtRT&amp;wrap=1">,</a> ​<a href="https://oregonstate.instructure.com/courses/1662153/files/69465532/download?verifier=ZjPDR9yxlUTRZ5ik3CZP8OXuBJeCZ6Sgc3tX7IUc&amp;wrap=1">four</a>​, <u>​</u><a href="https://oregonstate.instructure.com/courses/1662153/files/69465530/download?verifier=izEHHMFNsiZwxDOgFSCzw9q0YoV2dgnJvnqBNjFj&amp;wrap=1">five</a><u>​</u><a href="https://oregonstate.instructure.com/courses/1662153/files/69465530/download?verifier=izEHHMFNsiZwxDOgFSCzw9q0YoV2dgnJvnqBNjFj&amp;wrap=1">)</a>. The grading will use these specific files; do not feel like you have to create others.

You are also provided with a grading script (“​<a href="https://oregonstate.instructure.com/courses/1662153/files/69465562/download?verifier=lkvtv1coXtxWK0AHnaDTz4Yj5mUKgRCXeuXTuIRo&amp;wrap=1">p4gradingscript</a>​”) that you can run to test your software. If it passes the tests in the script, and has sufficient commenting, it will receive full points (see below). EVERY TIME you run this script, change the port numbers you use! Otherwise, because UNIX may not let go of your ports immediately, your successive runs may fail!

Finally, you will be required to write a compilation script (see below) that compiles all five of your programs, allowing you to use whatever C code and methods you desire. This will ease grading. Note that only C will be allowed, no C++ or any other language (Python, Perl, awk, etc.).

<h2>Example</h2>

Here is an example of usage, if you were testing your code from the command line:

$ cat plaintext1

THE RED GOOSE FLIES AT MIDNIGHT

$ otp_enc_d 57171 &amp;

$ otp_dec_d 57172 &amp;

$ keygen 10 &gt; myshortkey

$ otp_enc plaintext1 myshortkey 57171 &gt; ciphertext1

Error: key ‘myshortkey’ is too short $ echo $?

1

$ keygen 1024 &gt; mykey

$ otp_enc plaintext1 mykey 57171 &gt; ciphertext1

$ cat ciphertext1

GU WIRGEWOMGRIFOENBYIWUG T WOFL

$ keygen 1024 &gt; mykey2

$ otp_dec ciphertext1 mykey 57172 &gt; plaintext1_a

$ otp_dec ciphertext1 mykey2 57172 &gt; plaintext1_b

$ cat plaintext1_a

THE RED GOOSE FLIES AT MIDNIGHT $ cat plaintext1_b

WVIOWBTUEIOBC  FVTROIROUXA JBWE

$ cmp plaintext1 plaintext1_a

$ echo $?

0

$ cmp plaintext1 plaintext1_b plaintext1 plaintext1_b differ: byte 1, line 1 $ echo $?

1

$ otp_enc plaintext5 mykey 57171 otp_enc error: input contains bad characters $ echo $?

1

$ otp_enc plaintext3 mykey 57172

Error: could not contact otp_enc_d on port 57172

$ echo $?

2

$

<h2>Compilation Script</h2>

You must also write a short bash shell script called “compileall” that merely compiles your five programs. For example, the first two lines might be:

#!/bin/bash

gcc -o otp_enc_d otp_enc_d.c

…

This script will be used to compile your software, and must successfully run on our class server. The compilation must create all five programs, in the same directory as “compileall”, for immediate use by the grading script, which is named “p4gradingscript”.

<h1>Hints</h1>

<h2>Where to Start</h2>

First, write ​keygen​ – it’s simple and fun! Then, use our sample network programs <a href="https://oregonstate.instructure.com/courses/1662153/files/69465547/download?verifier=6kB1aA63zXb0URA9ScH1VqOm92FFaITqkRfS4zE7&amp;wrap=1">client.c</a> and <u>​</u><a href="https://oregonstate.instructure.com/courses/1662153/files/69465521/download?verifier=PLXfqZqBLM2InsyqzXoBp1pjjLcKECyKO5LcsBBU&amp;wrap=1">server.</a><a href="https://oregonstate.instructure.com/courses/1662153/files/69465521/download?verifier=PLXfqZqBLM2InsyqzXoBp1pjjLcKECyKO5LcsBBU&amp;wrap=1">c</a> (you don’t have to cite your use of them) to implement otp_enc​ and ​otp_enc_d​. Once they are functional, copy them and begin work on otp_dec​ and ​otp_dec_d​.

If you have questions about what your programs needs to be able to do, just examine the grading script. Your programs have to deal with exactly what’s in there: no more, no less. &#x1f642;

<h2>Sending Data</h2>

Recall that when sending data, not all of the data may be written with just one call to send() or write(). This occurs because of network interruptions, server load, and other factors. You’ll need to carefully watch the number of characters read and/or written, as appropriate. If the number returned is less than what you intended, you’ll need to restart the process from where it stopped. This means you’ll need to wrap a loop around the send/receive routines to ensure they finish their job before continuing.

If you try to send too much data at once, the server will likely break the transmission, as in the previous paragraph. Consider setting a maximum send size, breaking the transmission yourself every 1000 characters, say.

There are a few ways to handle knowing how much data you need to send in a given transmission. One way is to send an integer from client to server (or vice versa) first, informing the other side how much is coming. This relatively small integer is unlikely to be split and interrupted. Another way is to have the listening side looking for a termination character that it recognizes as the end of the transmission string. It could loop, for example, until it has seen that termination character.

<h2>Concurrency Implications</h2>

Remember that only one socket can be bound to a port at a time. Multiple incoming connections all queue up on the socket that has had ​listen()​ called on it for that port. After each ​accept()​ call is made, a new socket file descriptor is returned which is your server’s handle to that TCP connection. The server can accept multiple incoming streams, and communicate with all of them, by continuing to call ​accept()​, generating a new socket file descriptor each time.

<h2>About Newlines</h2>

You are only supposed to accept the 26 letters of alphabet and the “space” character as valid for encrypting/decrypting. However, all of the plaintext input files end with a newline character. ​<em>Text files need to end in a newline character for various reasons. </em>

When one of your programs reads in an input file, strip off the newline. Then encrypt and decrypt the text string, again with no newline character. ​<em>When you send the result to stdout, or save results into a file, tack a newline to the end, or your length will be off in the grading script.</em>​ Note that the newline character affects the length of files as reported by the wc command! Try it!

<h2>About Reusing Sockets</h2>

In the p4gradingscript, you can select which ports to use: I recommend ports in the 50000+ range. However, UNIX doesn’t immediately let go of the ports you use after your program finishes! I highly recommend that you frequently change and randomize the sockets you’re using, to make sure you’re not using sockets that someone else is playing with. In addition, to allow your program to continue to use the same port (your mileage may vary), read this:

<a href="https://beej.us/guide/bgnet/output/html/singlepage/bgnet.html#setsockoptman">http://beej.us/guide/bgnet/output/html/singlepage/bgnet.html#setsockoptman (Links to</a> <a href="https://beej.us/guide/bgnet/output/html/singlepage/bgnet.html#setsockoptman">an external site.)Links to an external site.</a>

…and then play around with this command:

setsockopt(sock_fd, SOL_SOCKET, SO_REUSEADDR, &amp;yes, sizeof(int));

<h2>Where to Develop</h2>

I ​<strong><em>HIGHLY</em></strong>​ recommend that you develop this program directly on our class server! Doing so will prevent you from having problems transferring the program back and forth, which can cause compatibility issues. Do not use any other non-class server to develop these programs.

If you do see ^M characters all over your files, which come from copying a Windows-saved file onto a UNIX file system, try this command:

$ dos2unix bustedFile