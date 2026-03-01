---
layout: post
title:  "Implementing A Genetic Algorithm In C"
description: A guide to writing a simple genetic algorithm for finding a matching string
date:   2024-12-15 00:19:13 +0100
categories: Genetic C
---

# What Is A Genetic Algorithm?
A genetic algorithm is an algorithm that can be used to solve problems meeting a certain criteria:
- Rating the quality of a solution is fast
- Generating new solution data is fast (that is, the data is small and can be made without lengthy transformations, such as a compilation process)
- Solutions can be meaningfully compared
- The result can be continously approached (similar inputs give similar outputs)

A genetic algorithm repeatedly generates new candidates, measures them and then repeats until an optimal candidate has been found.

At a high level:
```
generate a parent

until the candidate is at a target fitness:
    generate a new candidate, by mutating the parent slightly
    if it's better make it the new candidate
```

It's very simple, an effectively an optimised brute-force but can be used to generate much more complex data, such as sudoku puzzles, or solve continuous problems such as the n-queens problem. Furthermore, improvements such as multiple parallel generations that can find different solutions can be implemented to make it effective for a wider set of problems.

# Implementing One In C
I have been working through the book ['Genetic Algorithms With Python'](https://www.amazon.co.uk/Genetic-Algorithms-Python-Clinton-Sheppard/dp/1540324001), but found that using Python abstracted so much that I wanted to see how practical the algorithm was at the lowest possible level.

The algorithm in question is from the first chapter of the book and simply generates new strings to target a specific string, in our case 'Hello World!'. 

First we need a way to generate the parent, which acts as an initial candidate to evaluate new children against.

As we are attempting to find a matching string, we simply generate a random string as a starting parent:
```c
const char* geneSet = " abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!";

char* generate_parent(uint64_t length) {
    char* parent = malloc(sizeof(char) * (length + 1)); // One extra char for null terminator

    // Generate a random string made of characters from GeneSet of the target length
    for (size_t i = 0; i < length; i++) {
        size_t array_index = rand() % strlen(geneSet);
        parent[i] =  geneSet[array_index];
    }

    parent[length] = '\0'; // Add null terminator to the string

    return parent; 
}
```

Then we need a function to mutate the parent, this creates children, which can be compared against the current parent.

When a child is better than the parent they become the new one, and future children are compared against them:
```c
void mutate(char* guess) {
    size_t guess_index = rand() % strlen(guess); // Pick an index
    size_t gene_index = rand() % strlen(geneSet); // Pick a character to set it to from GeneSet

    guess[guess_index] = geneSet[gene_index];
}
```

Finally we need a way to judge how fit the strings are relative to the target, to help decide if a child is more fit than the current parent, so we use a simple character match count:
```c
const char* target = "Hello World!";

int get_fitness(char* guess) {
    int fitness = 0;

    // Count how many characters match
    for (size_t i = 0; i < strlen(target); i++) {
        if (guess[i] == target[i]) {
            fitness += 1;
        }
    }
    return fitness;
}
```

With this we have all the core elements needed to run the algorithm and we just need a driving loop.

This loop will create a parent, then keep creating and comparing children. When a child is better then parent new children are mutated from it, as it becomes the parent. This means each parent gradually drifts closer to the answer, as successive parents are based off an improved model. When the fitness is perfect, the strings match and we have found a perfect solution:
```c
int main() {
    srand((unsigned int)time(NULL));

    // Generate and rate the parent
    char* best_parent = generate_parent(strlen(target));
    int best_fitness = get_fitness(best_parent);

    while (true) {
        // Generate the child from a mutated parent
        char* child = malloc(sizeof(char) * (strlen(best_parent) + 1));
        strcpy(child, best_parent);
        mutate(child);

        // If the child is more fit make it the new parent
        int child_fitness = get_fitness(child);

        if (child_fitness > best_fitness) {
            free(best_parent);
            best_parent = child;
            best_fitness = child_fitness;
            printf("New Parent: %s Fitness: %d\n", best_parent, best_fitness);
        }
        else {
            free(child);
        }

        // If the child has reached peak fitness we have succeeded
        if (child_fitness >= strlen(target)) {
            break;
        }
    }

    printf("%s\n", best_parent);
}
```

Finally we can run it and see the results:
```
New Parent: EKlsUZD!mlgc Fitness: 2
New Parent: EKlsUZDomlgc Fitness: 3
New Parent: EKlsU Domlgc Fitness: 4
New Parent: EKllU Domlgc Fitness: 5
New Parent: EKllU Domldc Fitness: 6
New Parent: EKllo Domldc Fitness: 7
New Parent: EKllo Dorldc Fitness: 8
New Parent: EKllo Dorld! Fitness: 9
New Parent: HKllo Dorld! Fitness: 10
New Parent: Hello Dorld! Fitness: 11
New Parent: Hello World! Fitness: 12
Hello World!

```

The algorithm flies through hundreds of generations, but finishes on my laptop near-instantly. It's a surprisingly fast and effective algorithm and I'm consistently impressed by how much it can do as I learn more.

# Improvements To Be Made
While the algorithm works surprisingly well there are many techniques to make genetic algorithms better at solving more complex optimisation problems:
- **Populations**: Instead of evolving a single parent, maintain a population of candidates and select the fittest individuals each generation. This improves exploration and reduces the risk of getting stuck in locally optimal solution.

![](/images/genetic-optimum.png)

- **Crossover**: Combine parts of two strong candidates to produce offspring. This allows partial solutions to be merged to produce more whole solutions, which mutation alone cannot efficiently achieve.

- **Parallel Evaluation**: Fitness evaluation is easily parallelised, meaning it is possible to run thousands of populations and optimisations at once, greatly improving the time to find a solution.

- **Generalised Representations**: Abstracting genes away from strings allows the same algorithm to evolve numbers, bitfields, or structured data.

# Real-world Use Cases
Genetic algorithms have been used in a number of real-world scenarios, particularly in finding the optimal solution to physics problems. NASA notably used genetic algorithms to create the '[evolved antenna](https://en.wikipedia.org/wiki/Evolved_antenna)' they use on missions, which are adapted to find the best fitness for a number of factors such as background radiation patterns. The result looks odd, but is a well-designed and functional solution to the problem space:
![](/images/genetic-evolved-antenna.jpg)

# Conclusion
Genetic algorithms are a great example of how simple rules can produce surprisingly effective behaviour. Even with a minimal implementation, they can solve complex search spaces to find optimal solutions.

While this example is deliberately small, the same principles scale to much harder problems with only a few additional techniques. That flexibility is what makes genetic algorithms such a useful and enduring approach to optimisation.