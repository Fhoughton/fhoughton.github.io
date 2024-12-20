---
layout: post
title:  "APAW Week 23: Implementing A Genetic Algorithm In C"
date:   2024-12-15 00:19:13 +0100
categories: genetic c
---

# What Is A Genetic Algorithm?
A genetic algorithm is an algorithm that can be used to solve problems meeting a certain criteria:
- Checking if a solution is correct is fast
- Generating new data is fast
- It is possible to rank a solution against another
- The size of the data is relatively small
- The data can be continously approached (similar inputs give similar outputs)

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

First we need a way to generate the parent:
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

Then we need a function to mutate the parent:
```c
void mutate(char* guess) {
    size_t guess_index = rand() % strlen(guess); // Pick an index
    size_t gene_index = rand() % strlen(geneSet); // Pick a character to set it to from GeneSet

    guess[guess_index] = geneSet[gene_index];
}
```

Finally we need a way to judge how fit the strings are relative to the target:
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

With this we have all the core elements needed to run the algorithm and we just need a driving loop:
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

# Conclusion
I'm a big fan of genetic algorithms, and implementing one in C wasn't too much of an issue compared to Python. Getting memory safety and management right was a bit of an issue but I didn't have too many issues otherwise. I'm excited to learn more and I'd be interested in seeing if I could strike a careful balance between performance and abstraction in another language such as Golang.