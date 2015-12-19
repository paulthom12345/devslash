---
layout: post
title: Diffie-Hellman key exchange
---

<link rel='stylesheet' href='https://www.devslash.net/assets/dh.css'/>
The Diffie-Hellman was one of the largest changes in cryptography over the past few decades. It suddenly allowed for people to perform a key exchange over an unsecured line. This instantly had dramatic effects to furthering the cause of cryptography.

One was suddenly able to create a secure connection with a person that they had never met before. Previously keys had to be exchanged in some already secure manner such as an in-person exchange. Often this was done in person by exchanging books of keys for use in cyphers. This has obvious downsides, the largest of which being a captured key list will cause all others using the same key list to have their communications suddenly become completely compromised.

This lack of ability to exchange keys upon starting communication meant that in situations such as WWII submarines would often have large amounts of keys stored. That way on their long trips without docking they were able to keep up continuous conversation. A well known example of this is [U-559](http://en.wikipedia.org/wiki/German_submarine_U-559), a German submarine that was forced to surface after the Allies uncovered her location. Upon doing so the Allies instantly attempted to recover the code books. They were successful the code breakers at Bletchley Park were able to break the Naval enigma for several weeks.

Though this is an example which one can definitely make the point that it's a good thing that we didn't have D-H key exchange it does point out the obvious problems that cryptanalisys has been having since rather recently.

##The general idea
An often used analogy of the key exchange is colours. 

* User A selects a secret colour <span id='firstClass'></span>
* User B selects a secret colour <span id='secondClass'></span>
* User A and B agree on a shared colour <span id='shareClass'></span>
* Users A and B mix their colour and the shared colour
* Users A and B swap their mixed colour
* Users A and B add their own colour to the colour that the other person gave them

After this they will both end up with the same colour. All we have to do is find a mathematical basis that will work the same.

## The Security
The security of the above exchange hinges on anything that was made public. In our example it was the two mixed colours that were made public. If an eavesdropper was able to reverse the mixing process they would suddenly be able to figure out each persons' original colour. In our example one can imagine it's rather difficult to remove a colour from a mix. So we just need to make sure that in the mathematics that is used the *mixing* step will also create a result that makes it difficult to reverse.

##The Algorithm

We'll start off with the two traditional characters **Alice** and **Bob**. They want to decide on a key that will be used for their encryption.

Diffie-Hellman is *not* involved in the actual encryption of data. It is only a key exchange algorithm and as such this algorithm only needs to resolve to a shared key that **Alice** and **Bob** both can work out.

To start off with they will decide on the a *public* prime number. So lets say they decided on **29** (`p = 29`). As well as that a [Primitive Root](http://mathworld.wolfram.com/PrimitiveRoot.html) must be decided on. This depends on the prime number, and in our case will be 18 (`g = 8`).

Both **Alice** and **Bob** then will choose their own secret number. This does not have to be prime.

* Alice chooses 8
* Bob chooses 5

They then perform the equation <code>Share = g<sup>a</sup> mod p</code> where `a` is their secret number.

* **Alice** - <code>8<sup>8</sup> mod(29) = 20</code>
* **Bob** - <code>8<sup>5</sup> mod(29) = 27</code>

They now share that number with each other. So **Alice** sends the number 20 to **Bob** and **Bob** replies with 27.

They will then perform the following on the number received. 
<pre>S = r<sup>a</sup> mod(p)</pre>
where r is the number they received.

* **Alice** - <code>27<sup>8</sup> mod(29) = 24</code>
* **Bob**   - <code>20<sup>5</sup> mod(29) = 24</code>

Notice that they both arrived at the result of 24? That will now be the agreed value for their further encryption that takes place. Of course this is just an example and small values were used, in actual implementations they will instead use very large primes and integers.

##Security and Validity
Generally maths people will only be keen on using something if they have a proper proof that it works. So lets start by ensuring that we didn't just happen to be lucky when creating our example above. To do this we need to start by defining general numbers.

<pre>
p = prime decided
g = primitive root

A = Alice's secret number
B = Bob's secret number

A<sub>i</sub> = Alice's intemediatry result
   = g<sup>A</sup> mod(p)
   
B<sub>i</sub> = Bob's intemediatry result
   = g<sup>B</sup> mod(p)

A<sub>R</sub> = B<sub>i</sub><sup>A</sup> mod(p)
B<sub>R</sub> = A<sub>i</sub><sup>B</sup> mod(p)
</pre>
