---
layout: post
title:  "Failing To Crack A Cipher With Genetic Algorithms"
description: How I tried to crack substitution ciphers using genetic algorithms, and why it failed miserably.
date:   2025-03-01 00:19:13 +0100
categories: Python Ciphers Genetics
---

# The Problem
I decided that I wanted to see how successful automated cracking of substitution ciphers was.
I tried websites that used frequency analysis and they returned garbage (xjq evmdl pkzrw nzc ovgih zfqk xjq tbua szy) when using wikipedia's example (THE QUICK BROWN FOX JUMPS OVER THE LAZY DOG -> QEB NRFZH YOLTK CLU GRJMP LSBO QEB IXWV ALD).

# My Approach
Unsatisfied I decided that I would try and improve this frequency analysis using genetic algorithms, by first constructing the frequency mappings and then improving them using
a genetic algorithm that works on how many real words are present in the decoded cipher.

(Forgive the code quality here, as it didn't work and I bodged it in an evening I didn't make it greatly readable...)

## Rating Sentences
I first downloaded a large dictionary and wrote some code to rate sentences:
```python
def rate_sentence(sentence):
    global dictionary
    sentence = sentence.lower()
    words = sentence.split()
    
    word_count = len(words)
    real_word_count = 0
    
    for word in words:
        if word in dictionary:
            real_word_count+=1
            
    return real_word_count/word_count # Ranges from 0 to 1, as a percentage, of how many in dict
```

This code just performs a frequency analysis of how many of the words are real (in the dictionary file), and returns it as a percentage from 0 to 1.

## Finding Frequencies
Using a string, which contains characters in order of their frequency, I constructed a subtitution function:
```python
letter_freq = "etaoinshrdlcumwfgypbvkjxqz" # In order how common each is
cipher = "QEB NRFZH YOLTK CLU GRJMP LSBO QEB IXWV ALD"

def check_freq(x):
    freq = {}
    for c in set(x):
       freq[c] = x.count(c)
    return freq

def sort_dict(x):
    return {k: v for k, v in sorted(x.items(), key=lambda item: item[1])}

def map_cipher_freq(cipher, letter_freq):
    cipher = cipher.strip().lower().replace(" ","_")
    cipher_freq = check_freq(cipher)
    cipher_freq = sort_dict(cipher_freq)
    cipher_freq = list(cipher_freq.keys())
    
    mapped_cipher = ""
    for ch in cipher:
        if ch == "_":
            mapped_cipher += " "
        else:
            index = cipher_freq.index(ch)
            mapped_cipher += letter_freq[index]
        
    return mapped_cipher
```

## Doing Evolution
Finally I borrowed some old genetic algorithm code I wrote following the book [Genetic Algorithms With Python](https://www.amazon.co.uk/Genetic-Algorithms-Python-Clinton-Sheppard/dp/1540324001) by Clinton Sheppard. I adapted it and got a working program:
```python
# Start with the default mapping then mutate
parent = letter_freq

# Having generated our initial guess we then do a genetic algorithm loop to try and improve it
import random
import datetime

# Calculate how closely a string (guess) matches the target string.
def get_fitness(letter_freq):
    global cipher
    return rate_sentence(map_cipher_freq(cipher, letter_freq))

# Purpose: Generate a new string (child) by slightly modifying the parent.
# Steps:
#     Pick two random indexes in the string
#     Swap them
def mutate(parent):
    iter_count = random.randint(1, len(parent))
    
    for i in range(iter_count):
        parent_len = len(parent)
        
        parent = list(parent)
        index1 = random.randint(0, parent_len-1)
        index2 = random.randint(0, parent_len-1)
        
        char1 = parent[index1]
        char2 = parent[index2]
        
        parent[index1] = char2
        parent[index2] = char1
        
        parent = "".join(parent)
    
    return parent
    
def display(letter_freq):
    print("New best:", map_cipher_freq(cipher, letter_freq), get_fitness(letter_freq))

random.seed()
startTime = datetime.datetime.now()
bestParent = parent
bestFitness = get_fitness(parent)
display(bestParent)

while True:
    child = mutate(bestParent)
    childFitness = get_fitness(child)

    if bestFitness >= childFitness:
        continue
        
    display(child)
    bestFitness = childFitness
    bestParent = child
```

## Where It All Goes Wrong
You might be wondering now how it could go wrong, when surely the algorithm must improve upon a crude guess?

The answer is found by examining the output of the script:
```text
New best: xjq evmdl pkzrw nzc ovgih zfqk xjq tbua szy 0.0
New best: mka lvgcb pthnq ihj ovzrx heat mka wfus dhy 0.1111111111111111
New best: ida lxgsb ythnz mhw oxqce hvat ida jfur khp 0.2222222222222222
New best: ida lntsw qfexz meb onych evaf ida jgur kep 0.3333333333333333
New best: ida opvjw tsemz neb gpych eqas ida flur kex 0.4444444444444444
New best: ida lpvjw tsemz neb gpych eqas ida four kex 0.5555555555555556
New best: ida kpvyz qsemw neb gpjch etas ida four lex 0.6666666666666666
```

We can see the issues clearly:
- The program has a bad starting point, so struggles to make the leaps needed to mutate correctly
- It has found a local maximum where some words are found, and so it only changes letters to keep the words, never straying far from its current output
- It has not concept of likely words or sentence structure, so the actual meaning doesn't matter

## How Could It Be Fixed?
There are a few ways this could maybe be fixed:
- A better ranking algorithm, taking into account how common the words are and how 'real' the sentence is
- A better genetic algorithm approach, with more groups and a higher mutation rate
- Faster code, using GPU acceleration and libraries such as numpy

## Conclusions
I think this experience was a good lesson in the limits of genetic algorithms, and the problems they can face regarding local maximums and mutation.
It's a shame the code didn't work but I'm not done on my journey for computer cracking of ciphers, so it's not the end of the world, just a failed concept.