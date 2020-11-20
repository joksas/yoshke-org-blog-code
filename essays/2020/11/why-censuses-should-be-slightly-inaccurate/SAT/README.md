# About

This directory contains scripts that I used in the data reconstruction problem in my essay. I used the principles explored in [1], including [the code](https://queue.acm.org/appendices/Garfinkel_SugarInput.txt).

# Workflow

Here I describe how I obtained the solutions to the example problem in my essay. I came up with the following workflow using a lot of trial and error. I am no expert on SAT solvers, so there may be a more straightforward way of doing things, but this process worked for me and I have thus stuck with it. 

To reproduce my results, you will need to download [Sugar SAT Solver](http://bach.istc.kobe-u.ac.jp/sugar/) and [PicoSAT solver](http://fmv.jku.at/picosat/). The file [`input.txt`](input.txt) is supplied to Perl script `sugar` to convert problem specification to a set of boolean expressions in a form of a CNF file:
```text
sugar -n -map my.map input.txt
```

This will produce two files:

* CNF file which will contain a number of boolean variables and constraints. If you are on Linux, you should be able to find this file in `/tmp` directory. I will move it to my working directory and rename it to `my.cnf`. I think that for some reason Sugar solver produces more boolean variables than required, which results in a lot of duplicate solutions (involving variables in `input.txt`, not the boolean variables) in later stages. I will discuss how I dealt with that.
* `my.map` file which will contain the description of how the boolean variables in the CNF file are related to the variables (like `A_1`, `A_2`, etc.) that we defined in `input.txt`.

Then we use picoSAT solver to find the solutions. I use this solver because it has the ability to enumerate all solutions:
```text
picosat --all my.cnf > boolean-solutions.txt
```

`boolean-solutions.txt` will contain boolean solutions to the problem. We are going to use Sugar Java decoder to convert those boolean solutions to solutions involving the variables that we defined in `input.txt`. However, it seems that the decoder accepts only one boolean solution at a time, while `boolean-solutions.txt` contains all of them. To address this issue, I wrote a Bash script that splits `boolean-solutions.txt` into files involving individual solutions. It is embarrassingly inefficient, so I decided to not commit that script to this repository... I am sure you will come up with something better!

Anyway, with all those individual solution files generated, you can start to decode them  using the following command:
```text
java -jar sugar-v2-1-1.jar -decode boolean-solution-i.txt my.map > solution-i.txt
```
Of course, you should cycle through all `i`s using a script.

Finally, I mentioned how the produced CNF file (I think) contained more boolean variables than it should. Although all the boolean solutions were unique, once decoded back to the variables used in `input.txt`, some of the solutions were duplicate. As a result, I needed to find duplicate `solution-i.txt` files and delete them. For that I used a program called `fdupes`.

# References

[1] Garfinkel, S., Abowd, J. M., & Martindale, C. (2018). *Understanding database reconstruction attacks on public data*. Queue, 16(5), 28-53.
