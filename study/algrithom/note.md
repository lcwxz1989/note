# Alg

## Defination

Problem: A
Variable: i, j …. n
Subproblem: A1

## Tips

### Less Argument

Every argument is the signature for a function, this is very important, the more dimension, the smaller granularity question, the more complex, the more loop, the more …. . So make as less argument as possible.
Conclusion: So A1( i, j ) is better than A1( i, j, z )

### Argument Not Include Variable Out Of The Subproblem

If i of A1( i, j, z ) is out of the subproblem, this is not fine. Because it may be not controlled, it not belong to the subproblem.
Example:
Problem: 0-n
Variable: i, j , z
Subproblem: subproblem is from j to z
                      i is out of j…z, belong to 0….j-1

### Loop is better than iteration

Iteration is also loop by system, but it will cost a lot of space for every subproblem

### Independent variable vs dependent variable

To make the least variable is important, so to make sure which one is variable
Conclusion: To be sure which dimension is independent variable

