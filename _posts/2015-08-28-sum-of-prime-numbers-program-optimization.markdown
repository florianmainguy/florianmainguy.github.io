---
layout: post
title:  "Sum of prime numbers - Program optimization"
date:   2015-08-28 20:30:00
categories: jekyll update
---
When solving a problem on <a href="https://projecteuler.net/">Project Euler</a>, you can use whatever language you fancy. The only suggestion is to keep an execution time of less than a minute.

The problem n°10 is the first one I encountered where my solution exceeded this time. So I decided to do some algorithm optimizations. This process was really interesting and I feel so much more knowledgeable now that I want to share how I did it.

##Problem statement

>The sum of the primes below 10 is 2 + 3 + 5 + 7 = 17.
>Find the sum of all the primes below two million.

A number is a prime number if and only if it is divisible by 1 and itself.

As I am currently learning ruby and I enjoy it, I choose this language.

##First Solution - Brut Force

<div class="highlight">

</div>

In this first solution, I use the brut force method. I check all numbers from 2 to 2million by dividing them by all numbers less than themselves. When I get a prime number,  I add it to 'sum_primes'. The execution time is 18min 30sec. Not very efficient !

##Second Solution - First Optimization

We know that any number that is not a prime can be expressed by a product of primes. So instead of checking if a number is divisible by all numbers less than itself, we check if it is divisible by the primes less than itself. If not, it's a prime.

*** pb10-2 ***

I first create a prime array including only ***2***, and I push into it all the found primes.

An optimization I found is that any non prime numbers can be expressed by a product a primes, with the smallest prime being less than the square root of this number.

Time execution: 5.380sec. Not too bad ! At that point, I'm quite happy with that, and I go on the Project Euler forum to see other user solutions. And I found something unexpected..

##Third Solution - Cheat !

*** pb10-3 ***

There is a <cod>Prime</cod>Prime*** class in ruby ! What a surprise. And the execution time: 0.383sec. Oh god, this is quick !

I need to know what's behind this magical class. I need to beat this time !

##Eratosthenes Sieve

The ***Prime*** class is really big, so we will focus only on the internal subclass ***EratosthenesSieve*** that generates the prime numbers.

*** pb10-4 ***

What I learn first with this code, is that there is something called the Sieve of Eratosthenes and it is apparently important in this algorithm. 
A quick look on <a href="https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes">wikipedia</a> and I discover that Eratosthenes is a Greek mathematician, and that he created a simple algorithm to find all prime numbers to any given limit.

So, how does it work ?

***cit***
To find all the prime numbers less than or equal to a given integer n by Eratosthenes' method:
Create a list of consecutive integers from 2 through n: (2, 3, 4, ..., n).
Initially, let p equal 2, the first prime number.
Starting from p, enumerate its multiples by counting to n in increments of p, and mark them in the list (these will be 2p, 3p, 4p, ... ; the p itself should not be marked).
Find the first number greater than p in the list that is not marked. If there was no such number, stop. Otherwise, let p now equal this new number (which is the next prime), and repeat from step 3.
When the algorithm terminates, the numbers remaining not marked in the list are all the primes below n.
***cit***

Oh I see, so actually I was wrong, instead of taking a number and checking its divisors, I should take a prime and mark all his multiples. Great idea ! As there are much less primes than numbers, there will be much less instructions. Let's try to do it myself, by helping me with the ***EratosthenesSieve*** subclass.

##Fourth Solution - Greek Tribute

*** pb 10-5 ***

There is two optimizations in this algorithm. The first one is to look for multiples of a prime p beginning at ***p square***, and the second is to stop the process when we arrive at a prime p where ***p>square root number***, where N is the limit. But is it enough ?

Time executed: 0.443sec. Ahhhhh I'm so close !

Wait, I got an idea..

##Fifth Solution - Revelation

Multiples of any primes are half even, half odd. That means that half of the time, we eliminate even multiples, that have already been eliminated with the prime ***2***. Can we avoid doing that ?

*** pb 10-6 ***

This new optimization allows to eliminate first all even numbers, then to eliminate odd multiples of primes. And the execution time.. 0.348sec.

Yeah ! I did it !

![Usain Bolt]({{site.baseurl}}/assets/usain_bolt.jpg)