---
layout: post
title:  "Sum of prime numbers - Program optimization"
date:   2015-08-28 20:30:00
categories: jekyll update
---
When solving a problem on <a href="https://projecteuler.net/">Project Euler</a>, you can use whatever language you fancy. The only suggestion is to keep an execution time of less than a minute.

The problem n°10 is the first one I encountered where my solution exceeded this time. So I decided to do some algorithm optimizations. This process was really interesting and I feel so much more knowledgeable now that I want to share what I did.


##Problem statement

<div class="blockquote">
<p>The sum of the primes below 10 is 2 + 3 + 5 + 7 = 17.<br>
Find the sum of all the primes below two million.</p>
</div>

A number is a prime number if and only if it is divisible by 1 and itself.

As I am currently learning ruby and I enjoy it, I choose this programming language to solve the problem.


##First Solution - Brut Force

{% highlight ruby linenos%}
# SOLUTION 1. Executed Time: 18m31.577s

# Brut force method. 
# For each number n from 0 to 2million, check if it is divisible by any number 
# from 2 to n. If not, it's a prime, add it to "sum_primes".

nb_max = 2_000_000
arr_nb = (2..nb_max).to_a
sum_primes = 0

until arr_nb.count == 0
  divisor = arr_nb.first
  break if arr_nb.last / divisor < divisor
  arr_nb.delete_if { | number | number % divisor == 0 && number != divisor }
  sum_primes += divisor
  arr_nb.delete(divisor)
end

sum_primes += arr_nb.inject { |sum, n| sum + n }
puts "The sum of the primes below two million: #{sum_primes}"
{% endhighlight %}

In this first solution, I use the brut force method. I check all numbers from 2 to 2million by dividing them by all numbers less than themselves. When I get a prime number,  I add it to <code class="highlight">sum_primes</code>. The execution time is 18min 30sec. Not very efficient !


##Second Solution - First Optimization

We know that any number that is not a prime can be expressed by a product of primes. So instead of checking if a number is divisible by all numbers less than itself, we check if it is divisible by the primes less than itself. If not, it's a prime.

{% highlight ruby linenos%}
# SOLUTION 2. Executed Time: 5.380s

# For each number n from 0 to 2million, check if it is divisible by 
# the known prime numbers. If not, it is a prime, add it to "primes"

nb_max = 2_000_000
arr_nb = (2..nb_max).to_a
primes = [2]
sum_primes = 0
i=0

arr_nb.each do |number|
  primes.each do |prime|
  	break if number % prime == 0
  	if (prime == primes.last) || (number < prime**2)  # The smallest prime p multiple
  	  primes << number                                # of a number N: p**2 < N
  	  break
  	end
  end
end

sum_primes = primes.inject { |sum, n| sum + n }
puts "The sum of the primes below two million: #{sum_primes}"
{% endhighlight %}

I first create a prime array including only <code class="highlight">2</code>, and I push into it all the found primes.

An optimization I found is that any non prime numbers can be expressed by a product a primes, with the smallest prime being less than the square root of this number.

Time execution: 5.380sec. Not too bad ! At that point, I'm quite happy with that, and I go on the Project Euler forum to see other user solutions. And I find something unexpected..


##Third Solution - Cheat !

{% highlight ruby linenos%}
# SOLUTION 3. Executed Time: 0.383s

# Too easy..

require 'prime'

sum_primes = Prime.each(2_000_000).inject(:+)

puts "The sum of the primes below nb_max: #{sum_primes}"
{% endhighlight %}

There is a <code class="highlight">Prime</code> class in ruby ! What a surprise. And the execution time: 0.383sec. Oh god, this is quick !

I need to know what's behind this magical class. I need to beat this time !


##Eratosthenes Sieve

The <code class="highlight">Prime</code> class is really big, so we will focus only on the internal subclass <code class="highlight">EratosthenesSieve</code> that generates the prime numbers.

{% highlight ruby linenos%}
# Internal use. An implementation of eratosthenes' sieve
  class EratosthenesSieve
    include Singleton

    def initialize
      @primes = [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59,
 61, 67, 71, 73, 79, 83, 89, 97, 101]
      # @max_checked must be an even number
      @max_checked = @primes.last + 1
    end

    def get_nth_prime(n)
      compute_primes while @primes.size <= n
      @primes[n]
    end

    private
    def compute_primes
      # max_segment_size must be an even number
      max_segment_size = 1e6.to_i
      max_cached_prime = @primes.last
      # do not double count primes if #compute_primes is interrupted
      # by Timeout.timeout
      @max_checked = max_cached_prime + 1 if max_cached_prime > @max_checked

      segment_min = @max_checked
      segment_max = [segment_min + max_segment_size, max_cached_prime * 2].min
      root = Integer(Math.sqrt(segment_max).floor)

      sieving_primes = @primes[1 .. -1].take_while { |prime| prime <= root }
      offsets = Array.new(sieving_primes.size) do |i|
        (-(segment_min + 1 + sieving_primes[i]) / 2) % sieving_primes[i]
      end

      segment = ((segment_min + 1) .. segment_max).step(2).to_a
      sieving_primes.each_with_index do |prime, index|
        composite_index = offsets[index]
        while composite_index < segment.size do
          segment[composite_index] = nil
          composite_index += prime
        end
      end

      segment.each do |prime|
        @primes.push prime unless prime.nil?
      end
      @max_checked = segment_max
    end
  end

{% endhighlight %}

What I learn first with this code, is that there is something called the Sieve of Eratosthenes and it is apparently important in this algorithm. 
A quick look on <a href="https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes">wikipedia</a> and I discover that Eratosthenes is a Greek mathematician, and that he created a simple algorithm to find all prime numbers to any given limit.

So, how does it work ?

<div class="blockquote">
<p>To find all the prime numbers less than or equal to a given integer n by Eratosthenes' method:
<ol>
<li>Create a list of consecutive integers from 2 through n: (2, 3, 4, ..., n).
Initially, let p equal 2, the first prime number.</li>
<li>Starting from p, enumerate its multiples by counting to n in increments of p, and mark them in the list (these will be 2p, 3p, 4p, ... ; the p itself should not be marked).</li>
<li>Find the first number greater than p in the list that is not marked. If there was no such number, stop. Otherwise, let p now equal this new number (which is the next prime), and repeat from step 3.</li>
<li>When the algorithm terminates, the numbers remaining not marked in the list are all the primes below n.</li>
</ol></p>
</div>

Oh I see, so actually I was wrong, instead of taking a number and checking its divisors, I should take a prime and mark all his multiples. Great idea ! As there are much less primes than numbers, there will be much less instructions. Let's try to do it myself, by helping me with the <code class="highlight">EratosthenesSieve</code> subclass.


##Fourth Solution - Greek Tribute

{% highlight ruby linenos%}
# SOLUTION 4. Executed Time: 0.443s

# Implementation of eratosthenes' sieve
#
# Instead of taking a number and checking if it's a prime, we take a prime and 
# eliminate all his multiples. The next number following the prime that has not been
# eliminating is the next prime. We can then loop this two operations till we get
# all the primes of the number N.
#
# Optimization 1: stop eliminating multiples when we arrive at a prime p where
#                 p**2 > N because following non primes have already been eliminated.
# Optimization 2: eliminate multiples of a prime p beginning at p**2 because 
#                 previous non primes have already been eliminated.

nb_max = 2_000_000

# create array of numbers
arr_nb = [*0..nb_max]
arr_nb[0] = nil
arr_nb[1] = nil  

# if number is not a prime, replace value by "nil"
(2..Math.sqrt(nb_max)).each do |nb|            # optimization 1
  if arr_nb[nb] 								   
    (nb**2..nb_max).step(2*nb) do |not_prime|  # optimization 2
      arr_nb[not_prime]=nil
    end
  end
end

# sum the primes
sum_primes = arr_nb.compact.inject {|sum, n| sum + n}
puts "The sum of the primes below nb_max: #{sum_primes}"
{% endhighlight %}

There are two improvements here compared to the eratosthenes' sieve. The first one is to look for multiples of a prime p beginning at p<sup>2</sup>, and the second is to stop the process when we arrive at a prime p where p<sup>2</sup> > N, with N the limit. But is it enough ?

Time executed: 0.443sec. Ahhhhh I'm so close !

Wait, I got an idea..


##Fifth Solution - Revelation

Multiples of any primes are half even, half odd. That means that half of the time, we eliminate even multiples, that have already been eliminated with the prime <code class="highlight">2</code>. Can we avoid doing that ?

{% highlight ruby linenos%}
# SOLUTION 5 Executed Time: 0.348s

# Implementation of eratosthenes' sieve
#
# Instead of taking a number and checking if it's a prime, we take a prime and 
# eliminate all his multiples. The next number following the prime that has not been
# eliminating is the next prime. We can then loop this two operations till we get
# all the primes of the number N.
#
# Optimization 3: eliminate first all even numbers (except 2).
#                 Then eliminate only multiples of primes that are odd.

nb_max = 2_000_000

# create array of numbers
arr_nb = [*0..nb_max]
arr_nb[0] = nil
arr_nb[1] = nil  

# even numbers
(2**2..nb_max).step(2) {|even_nb| arr_nb[even_nb] = nil}

# odd numbers
(3..Math.sqrt(nb_max)).each do |number|   
  if arr_nb[number]
    (number**2..nb_max).step(2 *number) do |not_prime|   # optimization 3
      arr_nb[not_prime]=nil
    end
  end
end

# sum the primes
sum_primes = arr_nb.compact.inject {|sum, n| sum + n}
puts "The sum of the primes below nb_max: #{sum_primes}"
{% endhighlight %}

This new optimization eliminates first all even numbers, then odd multiples of primes. And the execution time.. **0.348sec**.

Yesss ! I did it !

![Usain Bolt]({{site.baseurl}}/assets/usain_bolt.jpg)