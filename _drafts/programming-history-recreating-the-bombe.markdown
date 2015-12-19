---
layout: post
title: Programming History - Recreating the Enigma
---

Alan Turing was an amazing man. His vision for computation and cryptography have led to his continual place in our minds as a fore-runner to modern day techniques and technology.

When working through WWII as a code breaker his main aim was to break the enigma. He did this through the use of the bombe machines. These rudimentary computers were the a large part of what enabled the allies to take victory.

Today, we're going to create a simulation of these computers and attempt to break the enigma ourselves.

##The design of the Enigma
The enigma machine was a rather large keyboard and light box. It would allow the user to encrypt and decrypt information. To encrypt the user would set the dials and plugboard to a known position and then press the key they were sending. A light would shine indicating the letter to be used as the encrypted value. 

The person receiving this message would then put their board into the same configuration and enter the letter received. Their light board would light up showing the original key that was pressed, and thus secure communication was achieved.
![enigma]