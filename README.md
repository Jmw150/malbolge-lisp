
# MalbolgeLisp v1.1
Made by Palaiologos, 2020 - 2021. Released to the public domain.

## What is MalbolgeLisp?

**MalbolgeLisp** is a LISP interpreter written in Malbolge. It's as of 2020 and 2021, [the most advanced, usable Malbolge program ever created](https://en.wikipedia.org/wiki/Malbolge#Programming_in_Malbolge). It supports everything LISPs generally tend to support (like `cond`, `let`, `lambda`, etc...). The v1.1 release greatly improved the performance and reduced the code size, while adding a few features.

## What is Malbolge? Why is it difficult?

**Malbolge** is a public domain esoteric programming language. It was specifically designed to be almost impossible to use, via a counter-intuitive 'crazy operation', trinary arithmetic, and self-modifying code. It builds on the difficulty of earlier, challenging esoteric languages like Brainfuck, but takes this aspect to the extreme. Despite this design, it is possible to write useful Malbolge programs (as this project proves).

What Malbolge instructions do depends on their position in the source code. After being ran, they are encrypted (so to make a loop, one has to decrypt it after each iteration - sounds hard already?). This is how so-called instruction cycles have been invented - it has been observed that some instructions on certain locations form looping cycles, which is the basis of Malbolge programming.

The most complex programs made in Malbolge, to date, include an adder, a 99 bottles of beer program, and a "Hello, world!" program (originally generated by a Lisp program utilizing the genetic algorithm).

MalbolgeLisp uses a special variant of Malbolge called **Malbolge Unshackled**. It's considerably harder to program for multiple reasons:

1) The rotation width is chosen randomly by the interpreter
2) Malbolge Unshackled lets the width of rotation be variable, which grows with the values in the D register, and since the initial rotation width is unknown, you have to probe it (because otherwise `*` returns unpredictable results)
3) Malbolge Unshackled's print instruction requires unicode codepoints
4) if the rotation width is unknown then you can't load values larger than 3^4-1, except values starting with a `1` trit
5) to overcome this you need a loop that probes the rotation width which is probably beyond most people's comprehension
6) the specification says that the value `0t21` should be used to print a newline, but this value is _theoretically_ impossible to obtain without having read an end of line or end of file from I/O before.
7) Malbolge Unshackled is actually usable because it's (as this project proves) Turing complete. The default Malbolge rotation width (10) constrains the addressable memory enough to make something cool with it.

## What is inside the zip file?

The release bundle includes interpreter binaries for Windows (x64, one is optimized for memory consumption and one for speed), and a few interpreters for Linux (the names should be self explanatory, also x64). If you're not running a x64 machine, you'll have to compile `fast20.c` yourself. From my observations, the best results were yielded by GCC and the code has been tuned to perform well when compiled with it. `malbolgelisp-v1.1.mb` is the source code for the interpreter.

## How to use MalbolgeLisp?

Pass the Lisp interpreter source code (`malbolgelisp-v1.1.mb`) as an argument to `fast20`.

## Using the REPL

After you start the interpreter, you should be greeted with a banner (containing the version, dot commands, etc..). The following dot commands are currently available:
  - `.F` - display all the features recognized by the interpreter
  - `.A` - display the information about the interpreter's author
  - `.R` - reset the memory
  - `.M` - display the information about memory consumption
  -  Any other command (and sending EOF - ^Z on Windows, ^D on UNIXes
     will terminate the interpreter).

After entering an expression (for example - `(+ 2 2)`), the interpreter should print something akin to this:
```
   % (+ 2 2)
   ..............|.....
   4
   %
```
The dots are used to signal that the code is being parsed (before the pipe character) or evaluated (after the pipe character). The amount of dots doesn't correspond to any consistent amount of time.

## The language
The interpreter supports the following features that will be discussed in this README:
```
   MALBOLGELISP V1.1 (2020-2021, PALAIOLOGOS)
   DOT COMMANDS:  .F(eatures)  .A(uthor)  .R(eset)  .M(emory)
   % .F
   ' + - < > = ! & | * / define defun lambda
   cond print atom cons car cdr if let
   iota size nth
```

## Arithmetics on numbers

A few examples of performing arithmetic in MalbolgeLisp are presented below. Note: Signed integers aren't supported.
```
   % (+ 2 -1)
   ..............|.....
   1
   % (* 3 3)
   ..............|.....
   9
   % (% 5 2)
   ..............|.....
   1
   % (> 5 6)
   ..............|.....
   0
   % (< 5 6)
   ..............|.....
   1
   % (= 6 6)
   ..............|......
   1
   % (! (= 6 7))
   ......................|.........
   1
   % (/ 25 5)
   ..............|.....
   5
   % (- 4 3)
   ..............|.....
   1
   % (& (= 2 2) (= 3 3))
   ....................................|...............
   1
```
There is no `>=` or `<=`, it has to be implemented by negating respectively `<` and `>`. A single `&` and `|` are respectively logical AND and OR.

## `define`, `defun` and `lambda`.
In MalbolgeLISP, you can introduce constants using the `define` function. For example:
```
   % (define x 5)
   ................|..
   5
   % (+ x 1)
   ................|.....
   6
```
MalbolgeLISP also supports lexically scoped (a function sees it's lexical ancestor's environment, not it's callers) lambdas. For instance:
```
   % ((lambda (n) (* n n)) 3)
   ........................................|.........
   9
   % ((lambda (m) ((lambda (n) (+ m n)) 2)) 2)
   ........................................................|.............
   4
```
How to define functions then? Well, a logical conclusion can be drawn from the examples above:
```
   % (define succ (lambda (x) (+ x 1)))
   .............................................|..
   (lambda (x) (+ x 1))
   % (succ 2)
   ...........|.........
   3
```
Because `define lambda` is a bit annoying to type each time and doesn't look all that well, we can use a nicer syntax for that - `defun`:
```
   % (defun succ (x) (+ x 1))
   .....................................|.
   (lambda (x) (+ x 1))
   % (succ 2)
   ...........|.........
   3
```
## Operations on lists and atoms
The equals function (`=') also works on atoms and lists. Although, you can't just introduce a list in your code, since an attempt of evaluating it would be made by an interpreter. For this reason, you have to use quoting, like so:
```
   % (= test test)
   ..................|......
   1
   % (= '(1 2 3) '(1 2 3))
   ..................................|.........
   1
   %
```
The `iota` function generates natural numbers from `0` to `N-1` (inclusive). For instance:
```
   % (iota 5)
   ...........|....
   (0 1 2 3 4)
```
`size` yields the size of a list passed as an argument. Using `size` and `iota`, we can make a really inefficient identity function for numbers:
```
   % (defun id (x) (size (iota x)))
   ..........................................|.
   (lambda (x) (size (iota x)))
   % (id 6)
   ...........|...........
   6
   % .M
   233B USED
   % # That's a lot of memory wasted!
```
`nth` yields the N-th element of a list. It uses the same indexing as `iota`, as demonstrated below:
```
   % (nth 3 (iota 5))
   ......................|........
   3
```
Let's try using atoms now. We can print an atom or a list using `print`, and we can check if something is an atom using the `atom` function. Note that `print` is an identity function with a side effect of printing an expression's value:
```
   % (atom null)
   ...........|....
   1
   % (atom '(1 2 3))
   .....................|....
   0
   % (print hello)
   .............|....
   hello

   hello
   % (print '(1 2 3))
   .....................|....
   (1 2 3)

   (1 2 3)
   %
```
Let's look at how `car` and `cdr` functions work. `car` takes the head (first element) of a list, and `cdr` takes the tail (everything _except_ the first element) of a list:
```
   % (define list '(1 2 3))
   ..........................|..
   (1 2 3)
   % (car list)
   .............|....
   1
   % (cdr list)
   .............|....
   (2 3)
```
Finally, `cons` can be used to prepend someting to a list. Like so:
```
   % (cons 3 '(1 2))
   .....................|.....
   (3 1 2)
```
## `if`, `cond` and `let`

The `if` function requires exactly 3 parameters - the condition, expression to evaluate if the condition is true, and the expression to evaluate if the condition is false. A small example:
```
   % (defun gt10 (x) (if (> x 10) (print yes) (print no)))
   .......................................................................|.
   (lambda (x) (if (> x 10) (print yes) (print no)))
   % (gt10 5)
   ...........|...............
   no

   no
   % (gt10 16)
   ...........|...............
   yes

   yes
```
If you need a lot of `if`s in one place, you can consider using `cond`. It takes a list of lists `(condition result)`, optionally including a `(result)` list. Using `cond`, we can make a three-way-comparison function (that returns `2 1 0` for `> = <`):
```
   % (defun <=> (x y) (cond ((> x y) 2) ((< x y) 0) (1)))
   ....................................................................................|.
   (lambda (x y) (cond ((> x y) 2) ((< x y) 0) (1)))
   % (<=> 5 6)
   ..............|.....................
   0
```
The final builtin function is `let'. It allows us to bind names to expressions (so that they're not re-evaluated at a later time). For example:
```
   % (let (x 5 y 6) (+ x y))
   .............................................|.............
   11
```
## Example programs

Quicksort (@La Condizione)
```
(defun filter (f l) (cond ((= l null) null) ((f (car l)) (cons (car l) (filter f (cdr l)))) (1 (filter f (cdr l)))))
(defun append (a b) (if (= null a) b (cons (car a) (append (cdr a) b) )))
(defun append3 (a b c) (append a (append b c)))
(defun list>= (m list) (filter (lambda (n) (! (< n m))) list))
(defun list< (m list) (filter (lambda (n) (< n m)) list))
(defun qsort (l) (if (= null l) null (append3 (qsort (list< (car l) (cdr l))) (cons (car l) null) (qsort (list>= (car l) (cdr l))))))
(qsort '(6 8 1 0 6 8 2))
```
"Dumb" sort (@umnikos)
```
(defun insert (i l) (if (= null l) (cons i l) (if (> i (car l)) (cons (car l) (insert i (cdr l))) (cons i l))))
(defun insertsort (l) (if (= null l) null (insert (car l) (insertsort (cdr l)))))
(insertsort '(6 8 1 0 6 8 2))
```
And a bunch of programs I wrote myself.
Counter
```
(defun counter (x) (cond ((< x 10) (counter (print (+ 1 x)))) (print ok)))
(counter 1)
```
Power
```
(defun power (x y) (if (= y 0) 1 (* x (power x (- y 1)))))
(power 3 5)
```
Factorials
```
(defun fac (n) (if (= n 0) 1 (* n (fac (- n 1)))))
(fac 6)
```
Maximum of a list
```
(defun max (l) (if (= null l) 0 (let (m (max (cdr l))) (if (> (car l) m) (car l) m))))
(max '(2 5 1 9 10))
```

Do you want _your_ code featured? Please open a pull request.
