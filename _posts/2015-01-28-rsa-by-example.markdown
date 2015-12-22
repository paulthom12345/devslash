---
layout: post
title: RSA Explained using Examples
date: '2015-01-28 09:08:04'
comments: true
---

<link rel='stylesheet' href='https://www.devslash.net/assets/css/extras.css'/>
RSA provides a fantastic method for allowing public key cryptography. For many years it was a debated topic whether it was possible at *all* to create a scheme for public cryptography. But in the year 1977 Ron Rivest, Adi Shamir, and Leonard Adleman published a paper on RSA, so named for the first letter of each of their last names. 

<div class='sidenote'>
<div class='exclaim'>
</div>
<p>
A curious side-note comes from the fact that Rivest, Shamir and Adleman were not actually the first people to have uncovered the algorithm. English intelligence had created a similar algorithm as early as 1973. The story goes that a new hire to the agency was introduced around the office. His name was Clifford Cocks. On the tour he met James H. Ellis where he learned that James had been working on the problem of public-private key systems for a long while. Clifford Cocks must have missed the part about the difficulty of the problem as he went to his office and decided to spend the day seeing if <i>he</i> could manage to solve this difficult problem. 
</p>
<p>
Later in the day he comes back to talk to Mr Ellis mentioning that he believes he'd solved the problem. A fresh set of eyes to the problem appeared to be all that it needed as it solved the problem that Mr Ellis had been working on for years.</p>
</div>

## [L](https://www.youtube.com/watch?v=ZSS5dEeMX64)ets get down to business
Lets have a look at an example of RSA before we get into how it works.

In each of these examples we have the following 'actors'. 

 * **Alice** - Good guy, friends with Bob
 * **Bob** - Good guy, friends with Alice
 * **Eve** - An eavesdropper. Pretty keen to hear what Alice and Bob are up to.
 
So to send a message between Alice and Bob we're first going to have to generate our set of public-private keys. The steps for that are below. We'll go through it in more detail in a moment.
 
 

 - **Alice** chooses  some *prime numbers* <br/>
**p** = 17 <br/>
**q** = 19
 - **Alice** Computes n `n = p * q`<br/> 
**n** = 17 * 19 = 323 <br/>
**Alice can share n with anyone. It's not secret.**
 - Compute `φ(n) = (p-1)(q-1) = 16 * 18 = 288`
 - Choose an number `e` such that `1 < e < φ(n)`
  -  `gcd(e, φ(n) = 1` therefore `e` and `φ(n)` are coprime 
  - Our example `e = 11`<br/>
 __Alice can share e with anyone. It's not secret.__
 - Determine <code>d = e<sup>-1</sup> mod φ(n)</code>
  - <code>d = 11<sup>-1</sup> mod(288)</code>
  - Our example makes `d = 131`
 
With these numbers we can now make our set of public/private keys. 

####Public Key
A public key is made up of **n** and **e**. **n** being the multiplication of the two large prime numbers and **e** being a number between 1 and 288 that had a greatest common divisor with 288 as 1. Often you're fine to just choose a random prime, but do test that `gcd(e, φ(n)) = 1` is true.  

<span class='emphasize'>**Alice's** public key is (323, 11)

####Private Key
Alice's private key is first of all made up with the same **n** that her public key was made from. The difference is that the other number used for the key is **d**. This number was the multiplicative inverse of `e (modulo φ(n))`. We'll go into why this works a bit later but for now you can just solve the equation <code>d = e<sup>-1</sup> mod(288)</code>. This is done through the Extended Euclid's Algorithm (see below). 

<span class='emphasize'>**Alice's** private key is (323, 131)

####How to use them

This is where Bob comes in. Bob wants to send Alice the message: `you should not trust eve`. First Bob knows that any message that he sends must be of an integer value less than `n`. In this case any message must be less than `228`. This counts as `11100100` in binary. So therefore we can set an easy upper bound on only transmitting 7 bits at a time. So lets make our string!

~~~
>>> ' '.join(format(ord(x), 'b') for x in "you should not trust eve")
'1111001 1101111 1110101 100000 1110011 1101000 1101111 1110101 1101100 1100100 100000 1101110 1101111 1110100 100000 1110100 1110010 1110101 1110011 1110100 100000 1100101 1110110 1100101'
~~~

###Encoding using the Public Key
Lets take our first message to send `1111001` and convert it to decimal. Once a decimal we will be able to encode it using the following equation.

<span class='emphasize'>**Encrypt** as follows: <code>CypherText of Message M = M<sup>e</sup> log(n)</code>

So our binary data can be converted to decimal and will come out as the number *121*. We then need to encode this data so that only Alice will be able to read it. Once we do this Bob will not be able to decrypt it again. It's a one way step.

In our example we end up with 
<pre><code>c(M) = M<sup>e</sup> mod(n)
     = 121<sup>11</sup> mod(323)
     = 379,749,833,583,241 mod(323)
     = 144
</code></pre>
Our first letter is now encoded as 144 or binary 10010000. This can then be sent across the wire to Alice.

###Lets Decrypt - Using the Private Key

Alice and only Alice will be able to decrypt the data (assuming that good values were used for the primes originally). So to do that she'll need to perform the following

<span class='emphasize'>**Decrypt** as <code>Plain Text from Message C = C<sup>d</sup> mod(n) </code>
<pre><code>m(C) = C<sup>d</sup> mod(n)
     = 144<sup>131</sup> mod(323)
     = 121
</code></pre>
We have just managed to encrypt what is the first letter of our message. We can set this as binary again and convert it back again.

```
>>> str(unichr(int('01111001', 2)))
'y'
```

Thus we've managed to send our first letter of our string to Alice. The rest can of course be completed in much the same way. 

###What makes this strong?
It's all well and good to show that we can go encrypt and decrypt a number. But to prove that it's a good idea we've got to make sure that the public key does not leak any required information. There's a few things that we need to make sure that we can ensure.

 * That Bob was *definitely* using Alice's public key when encrypting
 * That Alice was *definitely* receiving a message from Bob
 * That Eve was unable to infer the private key from listening to all public communication to Bob.
 
Lets start with the last one.

###Stopping the Cracker
In our above case there wasn't much that was transmitted publicly. The only information that is available is the public key, and anyone at all can get this. The idea behind a public key is to not keep it safe, it should be able to stand by itself. 

With that in mind lets take a look at the information provided in the public key. 

 * **n** - The multiplication of the two original primes
 * **e** - A number less than n that is relatively prime number to n 
 
So to get the private key Eve will need to get the factors of **n** and the number **d** where d was the multiplicative inverse of e mod n. 

####N - Why it's hard
So within N are two pieces of information that would unravel the whole thing. If you look at the original process the only numbers that are needed to work out the private key are **p**, **q** (the primes used in the original n equation) and **e**. Seeing we already have **e** we had better hope that finding out **p** and **q** is difficult. 

Thankfully. It turns out that it is. It's really, *really* difficult. 

When creating your p and q values each of them is most likely a prime number with a bit length of ~1024. So our number n is going to be incredibly large. This is of prime security concern as we need to make it as difficult as possible to factorise n. If n is ever factorised then suddenly we've lost all of our security as the private key is trivial to figure out. 

The problem of [Integer Factorisation](http://en.wikipedia.org/wiki/Integer_factorization) is a difficult problem. Without the use of Quantum computer (and Shor's algorithm) we are unable to currently solve this in a respectable time. Most of the methods that do work are based around trying a heap of values. For this reason we are able to be fairly sure that if we choose strong primes in p and q that the key will not be cracked (at least for a few thousand millennia).

###Interactive Example!

<select id='psel'>
  <option selected='selected'>P - The first prime</option>
  <option>4001</option>
  <option>4003</option>
  <option>4007</option>
  <option>4013</option>
  <option>4019</option>
  <option>4021</option>
  <option>4027</option>
  <option>4049</option>
  <option>4051</option>
  <option>4057</option>
  <option>4073</option>
  <option>4079</option>
  <option>4091</option>
  <option>4093</option>
  <option>4099</option>
  <option>4111</option>
  <option>4127</option>
  <option>4129</option>
  <option>4133</option>
  <option>4139</option>
  <option>4153</option>
  <option>4157</option>
  <option>4159</option>
  <option>4177</option>
  <option>4201</option>
  <option>4211</option>
  <option>4217</option>
  <option>4219</option>
  <option>4229</option>
  <option>4231</option>
  <option>4241</option>
  <option>4243</option>
  <option>4253</option>
  <option>4259</option>
  <option>4261</option>
  <option>4271</option>
  <option>4273</option>
  <option>4283</option>
  <option>4289</option>
  <option>4297</option>
  <option>4327</option>
  <option>4337</option>
  <option>4339</option>
  <option>4349</option>
  <option>4357</option>
  <option>4363</option>
  <option>4373</option>
  <option>4391</option>
  <option>4397</option>
  <option>4409</option>
  <option>4421</option>
  <option>4423</option>
  <option>4441</option>
  <option>4447</option>
  <option>4451</option>
  <option>4457</option>
  <option>4463</option>
  <option>4481</option>
  <option>4483</option>
  <option>4493</option>
  <option>4507</option>
  <option>4513</option>
  <option>4517</option>
  <option>4519</option>
  <option>4523</option>
  <option>4547</option>
  <option>4549</option>
  <option>4561</option>
  <option>4567</option>
  <option>4583</option>
  <option>4591</option>
  <option>4597</option>
  <option>4603</option>
  <option>4621</option>
  <option>4637</option>
  <option>4639</option>
  <option>4643</option>
  <option>4649</option>
  <option>4651</option>
  <option>4657</option>
  <option>4663</option>
  <option>4673</option>
  <option>4679</option>
  <option>4691</option>
  <option>4703</option>
  <option>4721</option>
  <option>4723</option>
  <option>4729</option>
  <option>4733</option>
  <option>4751</option>
  <option>4759</option>
  <option>4783</option>
  <option>4787</option>
  <option>4789</option>
  <option>4793</option>
  <option>4799</option>
  <option>4801</option>
  <option>4813</option>
  <option>4817</option>
  <option>4831</option>
  <option>4861</option>
  <option>4871</option>
  <option>4877</option>
  <option>4889</option>
  <option>4903</option>
  <option>4909</option>
  <option>4919</option>
  <option>4931</option>
  <option>4933</option>
  <option>4937</option>
  <option>4943</option>
  <option>4951</option>
  <option>4957</option>
  <option>4967</option>
  <option>4969</option>
  <option>4973</option>
  <option>4987</option>
  <option>4993</option>
  <option>4999</option>
</select>

<select id='qsel'>
<option selected='selected'>Q - The first prime</option>
  <option>4001</option>
  <option>4003</option>
  <option>4007</option>
  <option>4013</option>
  <option>4019</option>
  <option>4021</option>
  <option>4027</option>
  <option>4049</option>
  <option>4051</option>
  <option>4057</option>
  <option>4073</option>
  <option>4079</option>
  <option>4091</option>
  <option>4093</option>
  <option>4099</option>
  <option>4111</option>
  <option>4127</option>
  <option>4129</option>
  <option>4133</option>
  <option>4139</option>
  <option>4153</option>
  <option>4157</option>
  <option>4159</option>
  <option>4177</option>
  <option>4201</option>
  <option>4211</option>
  <option>4217</option>
  <option>4219</option>
  <option>4229</option>
  <option>4231</option>
  <option>4241</option>
  <option>4243</option>
  <option>4253</option>
  <option>4259</option>
  <option>4261</option>
  <option>4271</option>
  <option>4273</option>
  <option>4283</option>
  <option>4289</option>
  <option>4297</option>
  <option>4327</option>
  <option>4337</option>
  <option>4339</option>
  <option>4349</option>
  <option>4357</option>
  <option>4363</option>
  <option>4373</option>
  <option>4391</option>
  <option>4397</option>
  <option>4409</option>
  <option>4421</option>
  <option>4423</option>
  <option>4441</option>
  <option>4447</option>
  <option>4451</option>
  <option>4457</option>
  <option>4463</option>
  <option>4481</option>
  <option>4483</option>
  <option>4493</option>
  <option>4507</option>
  <option>4513</option>
  <option>4517</option>
  <option>4519</option>
  <option>4523</option>
  <option>4547</option>
  <option>4549</option>
  <option>4561</option>
  <option>4567</option>
  <option>4583</option>
  <option>4591</option>
  <option>4597</option>
  <option>4603</option>
  <option>4621</option>
  <option>4637</option>
  <option>4639</option>
  <option>4643</option>
  <option>4649</option>
  <option>4651</option>
  <option>4657</option>
  <option>4663</option>
  <option>4673</option>
  <option>4679</option>
  <option>4691</option>
  <option>4703</option>
  <option>4721</option>
  <option>4723</option>
  <option>4729</option>
  <option>4733</option>
  <option>4751</option>
  <option>4759</option>
  <option>4783</option>
  <option>4787</option>
  <option>4789</option>
  <option>4793</option>
  <option>4799</option>
  <option>4801</option>
  <option>4813</option>
  <option>4817</option>
  <option>4831</option>
  <option>4861</option>
  <option>4871</option>
  <option>4877</option>
  <option>4889</option>
  <option>4903</option>
  <option>4909</option>
  <option>4919</option>
  <option>4931</option>
  <option>4933</option>
  <option>4937</option>
  <option>4943</option>
  <option>4951</option>
  <option>4957</option>
  <option>4967</option>
  <option>4969</option>
  <option>4973</option>
  <option>4987</option>
  <option>4993</option>
  <option>4999</option>
</select>

n = <span id='p'></span> x <span id='q'></span> = <span id='ans'></span>

φ(n) = (<span id='pine'></span>-1)(<span id='qine'></span>-1) = <span id='secresult'></span>

Now we need to choose 1 < **e** < φ(n) and gcd(e, φ(n)) = 1;
We'll choose a common e that's used. That being  65,537 which is 2<sup>16</sup>+1



<b>Values</b><br/>
<b>N:</b><span id='ansn'></span><br/>
<b>φ(n):</b><span id='ansg'></span><br/>
<b>E:</b>65537<br/>
<b>D:</b><span id='ansd'></span>


<h4>Public Key</h4>

(e,n) = (<span id='pe'></span>, <span id='pn'></span>)


<h4>Private Key</h4>

(d,n) = (<span id='rd'></span>, <span id='rn'></span>)


<script src="https://ajax.googleapis.com/ajax/libs/jquery/2.1.3/jquery.min.js"></script>
<script>
//Populate the primes
$("#psel").change(function(){$("#p").text(this.value);checkN()})
$("#qsel").change(function(){$("#q").text(this.value);checkN()})

function checkN(){
	var p = +$("#psel").val()
	var q = +$("#qsel").val()

	if(isNaN(p) || isNaN(q))
		return;

	var n = p*q;
	var t = (p-1)*(q-1)
	$("#ans").text(n)
	$("#ansn").text(n)
	$("#pn").text(n)
	$("#rn").text(n)
	$("#ansg").text(t)

	$("#pine").text(p)
	$("#qine").text(q)
	$("#secresult").text(t)

	res = xgcd(65537, t)[0]
	if(res < 0)
		res = res + t;

	$("#ansd").text(res);
	$("#pe").text('65537')
	$("#rd").text(res)

}


function xgcd(a, b) { 

	if (b == 0) {
		return [1, 0, a];
	}

	temp = xgcd(b, a % b);
	x = temp[0];
	y = temp[1];
	d = temp[2];
	return [y, x-y*Math.floor(a/b), d];
}
</script>