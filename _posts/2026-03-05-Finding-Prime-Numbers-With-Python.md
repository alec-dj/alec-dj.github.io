---
layout: post
title: Finding Prime Numbers with Python
image: "/posts/primes_image.jpeg"
tags: [Python, Primes]
---

In this post I lay out a function in Python that can quickly find all the Prime numbers below a given value.  For example, if the function is passed a value of 100, it would find all the prime numbers below 100 and detail what is the highest of those prime numbers.

For clarity, a prime number is a number that can only be divided into another integer by itself and 1. I.e.,  7 is a prime number as no other numbers apart from 7 or 1 divide cleanly into it; 8 is not a prime number, as while 8 and 1 divide into it, so do 2 and 4.

---

We start by defining a variable that will act as the upper limit of numbers through which we want to search. We'll start with 20, therefore for now we are hoping to find all prime numbers that are equal to or smaller than 20:

```ruby
n = 20
```

The smallest true prime number is 2, so we start by creating a set of numbers over which our function will run, checking whether each is or isn't a prime. Therefore we want every integer between 2 and what we set above as the upper bound, which in this case was 20. We use n+1 as the range logic in python is not inclusive of the upper limit we set.

Instead of using a list, we're going to use a set.  The reason for this is that sets have some special functions that will allow us to eliminate non-primes during our search (more on this below!):

```ruby
number_range = set(range(2, n+1))
```

Next we must create a repository in which we can store any primes we discover.  Here, a list works well:

```ruby
primes_list = []
```

Whilst the final product will use a while loop to iterate through our list and check for primes, before throwing everything into that at once, I find it useful (and often time-saving) to write up the logic individually and iterate manually first.  This means I can check that it is working correctly before it is set off to run through everything on its own.

Now we have our set of numbers (called number_range) to check all integers between 2 and 20. Let's extract the first number from that set, to check whether or not it is a prime. When we check the value we're going to check if it is a prime; if it is, we will add it to our list called primes_list, and if it isn't a prime, we will discard it.

There is a method which will remove an element from a list or set and provide that value to us, and that method is called *pop*.

```ruby
print(number_range)
>>> {2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20}
```
If we use pop, and assign this to the object called **prime** it will *pop* the first element from the set from **number_range** into **prime**:

```ruby
prime = number_range.pop()
print(prime)
>>> 2
print(number_range)
>>> {3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20}
```

Now we know that the first value in our range must be a prime, as there is nothing smaller than it, so nothing else can divide evenly into it.  As we know it's a prime, let's add it to our list of primes:

```ruby
primes_list.append(prime)
print(primes_list)
>>> [2]
```

Here we use a little maths trick to speed up the check of our remaining number_range for non-primes. For the prime number we just checked (in this first case, 2), we want to generate all the multiples of that up to our upper range (in this case 20), as none of these can be primes.

Again, we will use a set rather than a list, because it allows us some special functionality that will be detailed below, which is the crux of this approach:

```ruby
multiples = set(range(prime*2, n+1, prime))
```

When a range is created the syntax is: range(start, stop, step). For the starting point, we don't need the first number in the number_range set as that has already been added as a prime, so we start our range of multiples at 2 * this number (this is the first possible multiple); in our case, the starting number is 2 so the first multiple will be 4.

For the end point of our range, we specified that we want our range to go up to n (in this case n=20), so we use n+1 to specify that we want n (20) to be included.

The **step** is key here: we want multiples of our number, so we want to increment in steps *of our* number. Therefore we put in **prime** here.

Lets have a look at our list of multiples:

```ruby
print(multiples)
>>> {4, 6, 8, 10, 12, 14, 16, 18, 20}
```

The next part is the set functionality referenced above: **difference_update**. This removes any values from number_range that are multiples of the number we just checked. Again, the reason we do this is because if a number is a multiple of anything other than 1 or itself then it is **not a prime number** and it can be removed.

Before we apply **difference_update**, let's look at our two sets:

```ruby
print(number_range)
>>> {3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20}

print(multiples)
>>> {4, 6, 8, 10, 12, 14, 16, 18, 20}
```

**difference_update** works in a way that will update one set to only include the values that are *different* from those in a second set.

To use this, we put our initial set and then apply difference update with our multiples:

```ruby
number_range.difference_update(multiples)
print(number_range)
>>> {3, 5, 7, 9, 11, 13, 15, 17, 19}
```

When we look at our number range now, all values that were also present in the multiples set have been removed as we *know* they are not primes.

Using this method, we have already made a massive reduction to the pool of numbers that need to be tested, so this is really efficient. It also means the smallest number in our range *is a prime number* as we know nothing smaller than it divides into it; therefore, we can run all that logic again from the top!

Whenever something can be run over and over again, a while loop is often a good solution.

Below is the above code, with a while loop doing the hard work of updating the number list and extracting primes until the list is empty.

Let's run it for any primes below 1000:

```ruby
n = 1000  # upper bound
number_range = set(range(2, n+1))  # number range to be checked
primes_list = []  # empty list to append primes to

# iterate until list is empty
while number_range:
    prime = number_range.pop()  # remove prime (lowest number in set) from number_range
    primes_list.append(prime)   # know it's prime - add it to primes_list               
    multiples = set(range(prime*2, n+1, prime))  # create set of multiples of prime
    number_range.difference_update(multiples)  # update number_range to remove all multiples
```

Let's print the primes_list to have a look at what we found!

```ruby
print(primes_list)
>>> [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97, 101, 103, 107, 109, 113, 127, 131, 137, 139, 149, 151, 157, 163, 167, 173, 179, 181, 191, 193, 197, 199, 211, 223, 227, 229, 233, 239, 241, 251, 257, 263, 269, 271, 277, 281, 283, 293, 307, 311, 313, 317, 331, 337, 347, 349, 353, 359, 367, 373, 379, 383, 389, 397, 401, 409, 419, 421, 431, 433, 439, 443, 449, 457, 461, 463, 467, 479, 487, 491, 499, 503, 509, 521, 523, 541, 547, 557, 563, 569, 571, 577, 587, 593, 599, 601, 607, 613, 617, 619, 631, 641, 643, 647, 653, 659, 661, 673, 677, 683, 691, 701, 709, 719, 727, 733, 739, 743, 751, 757, 761, 769, 773, 787, 797, 809, 811, 821, 823, 827, 829, 839, 853, 857, 859, 863, 877, 881, 883, 887, 907, 911, 919, 929, 937, 941, 947, 953, 967, 971, 977, 983, 991, 997]
```

Let's now produce some interesting stats from our list which we can use to summarise our findings: the number of primes that were found, and the largest prime in the list.

```ruby
prime_count = len(primes_list)
largest_prime = max(primes_list)
print(f"There are {prime_count} prime numbers between 1 and {n}, the largest of which is {largest_prime}")
>>> There are 168 prime numbers between 1 and 1000, the largest of which is 997
```

Amazing!

To tidy this up, we can collapse it into function, which you can see below:

```ruby
def primes_finder(n):  # n (the upper bound of the search) is the only variable we will need to pass to the function when we call it
    
    number_range = set(range(2, n+1))

    primes_list = []

    while number_range:
        prime = number_range.pop()
        primes_list.append(prime)
        multiples = set(range(prime*2, n+1, prime))
        number_range.difference_update(multiples)
        
    prime_count = len(primes_list)
    largest_prime = max(primes_list)
    print(f"There are {prime_count} prime numbers between 1 and {n}, the largest of which is {largest_prime}")
```

Now we can just pass the function the upper bound of our search and it will do the rest!

Let's go for something large, say a million:

```ruby
primes_finder(1000000)
>>> There are 78498 prime numbers between 1 and 1000000, the largest of which is 999983
```

I hoped you enjoyed learning about primes, and one way to search for them using Python.

---

###### Important Note: Using pop() on a Set in Python

In the real world, we would need to make a consideration around the pop() method when used on a set as in some cases it can be inconsistent.

The pop() method will usually extract the lowest element of a set. Sets however are, by definition, unordered. The items are stored internally with some order, but this internal order is determined by the hash code of the key (which is what allows retrieval to be so fast). 

This hashing method means that we can't 100% rely on it successfully getting the lowest value. In very rare cases, the hash provides a value that is not the lowest.

Even though here we're just coding up something fun, it is most definitely a useful thing to note when using Sets and pop() in Python in the future.

The simplest solution to force the minimum value to be used is to replace the line:

```ruby
prime = number_range.pop()
```

...with the lines:

```ruby
prime = min(sorted(number_range))
number_range.remove(prime)
```

Where we firstly force the identification of the lowest number in the number_range into our prime variable, and following that we remove it.

However, because we have to sort the list for each iteration of the loop in order to get the minimum value, it's slightly slower than what we saw with pop()!


