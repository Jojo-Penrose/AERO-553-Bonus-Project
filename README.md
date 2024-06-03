# AERO-553-Bonus-Project
This file contains the implementation and demonstration code for the AERO 553 bonus project. The assignment was to implement the exponent operator, X^Y, for all real numbers in a compiled language (C, in this case).

To view the full code manual and marked up source, visit the GitHub Pages site at: https://jojo-penrose.github.io/AERO-553-Bonus-Project/html/index.html

See below for solution examples and the algorithm's full derivation.

# Solution Examples (w/ WolframAlpha Comparison)
![image](https://github.com/Jojo-Penrose/AERO-553-Bonus-Project/assets/44389628/58db1ec1-1ef4-41b9-890f-563b671236d0)

![image](https://github.com/Jojo-Penrose/AERO-553-Bonus-Project/assets/44389628/f4b4e2ae-f737-41fc-a20d-16324e45dee2)

![image](https://github.com/Jojo-Penrose/AERO-553-Bonus-Project/assets/44389628/dde95355-2b35-4992-a5ed-ab44b63577a8)

![image](https://github.com/Jojo-Penrose/AERO-553-Bonus-Project/assets/44389628/83610cd9-5e5c-49d2-8df2-4026a0fb36dc)

![image](https://github.com/Jojo-Penrose/AERO-553-Bonus-Project/assets/44389628/287be551-31e1-487a-a5ab-6195601391f4)

![image](https://github.com/Jojo-Penrose/AERO-553-Bonus-Project/assets/44389628/e1f1c184-546a-43f8-80a2-dcb8b2f4a956)

![image](https://github.com/Jojo-Penrose/AERO-553-Bonus-Project/assets/44389628/ed8ce198-4daf-44c9-90d5-acec7573cc9d)

Note that they disagree on the sign of the imaginary part. However, this is only a consequence of *which* root the calculator chooses to report -- in this example, there are five distinct roots:

![image](https://github.com/Jojo-Penrose/AERO-553-Bonus-Project/assets/44389628/8fdd1059-c551-4fe6-b00d-262f4969487d)

![image](https://github.com/Jojo-Penrose/AERO-553-Bonus-Project/assets/44389628/cef187fd-cf41-44e5-aeda-8d5968a01f9c)

Again, the calculators disagree on the sign (and it would take far too many clicks on the "More Roots" button to reach this particular one). However, recall that *all complex roots must come as complex conjugates* -- in other words, these two results are efffectively equivalent.

![image](https://github.com/Jojo-Penrose/AERO-553-Bonus-Project/assets/44389628/ae9ba288-5a1f-4a3e-bbfc-dfaeffc0aa2b)

# Solution Derivation
The challenge in this project was to write a function in a compiled language to compute the exponentiation function for all real numbers:

$$ X^Y, \text{ where } \lbrace X,Y \in \mathbb{R} \rbrace $$

The cases that make this a challenge typically look like, for example:

$$ (-1.1)^{0.7} $$

where some root of a negative number is requested, and the answers are potentially non-real. Typically, on handheld calculators, logarithm-based methods are used to compute non-integer powers, and the domain is heavily restricted as a result. With the available power on modern computers (and the examples set by applications like MATLAB and WolframAlpha), a more robust system is surely possible.

To come up with a solution to this problem, one critical assumption was made that makes the entire algorithm possible. In mathematics, the domain of all real numbers includes irrational numbers. On a computer, however, irrational numbers cannot be truly represented, only approximated. The simplest way to perform floating-point computation on a computer is to represent any decimal number as a ratio of integers. For example:

$$ \pi = 3.141592653589793... \approx 3.14159 \approx \frac{314159}{100000} $$

In assembly language programming for embedded systems, it is common to find workarounds and simplifications for specific computations whenever floating-point math is needed, because floating-point math requires a separate processing unit (FPU) and adds significant overhead in real-time systems.

Therefore, in the context of this project, **we assume that all real numbers can be approximated as a ratio of integers.** In other words, **we restrict the domain to any non-irrational real numbers.** This assumption is validated by the fact that a computer cannot resolve irrational numbers anyway, and in fact represents a very real limitation of the hardware.

In base C, without any additional libraries, we limit the solution to the following operators:

- Addition & subtraction
- Multiplication & division
- Modulo
- Logical comparison

We can also implement the integer exponentiation function very simply, using recursion:

```
double WholeExp(double X, int n)
{
    if (n == 0) {
        return 1;
    } else if(n > 0) {
        return X*WholeExp(X, n-1);
    } else {
        return 1/(WholeExp(X, -n));
    }
}
```

We will also need the factorial, which is another simple recursion problem:

```
unsigned long long int IntFactorial(unsigned long long int F)
{
    if(F <= 1) {
        return 1;
    } else {
        return F*IntFactorial(F-1);
    }
}

```

These two round out the basic toolset.

To derive the generalized exponentiation algorithm, begin by applying the computer-compatible non-irrational real number assumption that:

$$ X^Y = \bigl( \frac{a}{b} \bigr) ^{\frac{c}{d}}, \text{ where } X = \frac{a}{b} \text{ and } Y = \frac{c}{d} $$

where $a$, $b$, $c$, and $d$ are **integers.** This is easily done in C by multiplying $X$ and $Y$ by increasing powers of $10$ until the floating point passes the last digit. The algorithm also uses a list of the first 26 prime numbers to factor the ratio as much as possible. In order to handle negative numbers, we further describe the operation by:

$$ X^Y = \lgroup (-1)^m\frac{a}{b} \rgroup ^{ \lgroup (-1)^n\frac{c}{d} \rgroup }, \text{ where } X = (-1)^m\frac{a}{b} \text{ and } Y = (-1)^n\frac{c}{d} $$

where $m$ and $n$ can be 0 or 1. If the base or exponent are positive, then $m = 0$ or $n = 0$, respectively. If the base or exponent are negative, then $m = 1$ or $n = 1$, respectively. This guarantees that $a$, $b$, $c$, and $d$ are **positive integers.**

Use the rules of exponents to manipulate the operation and split the base into three terms $T_1$, $T_2$, and $T_3$:

$$ X^Y = \bigl( (-1)^m \bigr) ^{ \bigl( (-1)^n\frac{c}{d} \bigr) } 
         \bigl( a \bigr) ^{ \bigl( (-1)^n\frac{c}{d} \bigr) } 
         \bigl( \frac{1}{b} \bigr) ^{ \bigl( (-1)^n\frac{c}{d} \bigr) } 
         = (T_1)(T_2)(T_3) $$
         
Where:

$$ T_1 = \bigl( (-1)^m \bigr) ^{ \bigl( (-1)^n\frac{c}{d} \bigr) }
       = \Bigl( \bigl( (-1)^{m(-1)^n} \bigr) ^{c} \Bigr) ^{1/d} $$

$$ T_2 = \bigl( a \bigr) ^{ \bigl( (-1)^n\frac{c}{d} \bigr) }
       = \Bigl( \bigl( (a)^{(-1)^n} \bigr) ^{c} \Bigr) ^{1/d} $$
       
$$ T_3 = \bigl( \frac{1}{b} \bigr) ^{ \bigl( (-1)^n\frac{c}{d} \bigr) } 
       = \Bigl( \bigl( (\frac{1}{b})^{(-1)^n} \bigr) ^{c} \Bigr) ^{1/d} $$

Grouping the exponent subterms in this manner shows that, under the rules of exponents, each subpart of the exponentiation operation can be performed one at a time. Further, some of these terms will have simplifications that make the overall computation easier.

Starting with $T_1$, there are four possible combinations of $m$ and $n$ that are determined by the sign of the base and exponent. They are:

$$ m = 0, n = 0: (-1)^{0(1)} = (-1)^0 = 1 $$

$$ m = 0, n = 1: (-1)^{0(-1)} = (-1)^0 = 1 $$

$$ m = 1, n = 0: (-1)^{1(1)} = (-1)^1 = -1 $$

$$ m = 1, n = 1: (-1)^{1(-1)} = (-1)^{-1} = \frac{1}{-1} = -1 $$

In the trivial case that $m = 0$ (i.e., $X$ is positive), the entire first term $T_1$ works out to 1 and the result is guaranteed to be real. If $X$ is negative, however, certain conditions lead to non-real answers.

Following the case where $m = 1$, we find:

$$ T_1 = \bigl( (-1) ^{c} \bigr) ^{1/d} $$ 

Because $c$ and $d$ are guaranteed to be positive integers by design, it follows that:

$$ c \text{ is odd: } T_1 = (-1) ^{1/d} $$

$$ c \text{ is even: } T_1 = (1) ^{1/d} = 1 $$

If $c$ is even, then the $-1$ flips sign and $T_1 = 1$. Knowing this, we can case-check $m$, $n$ (using sign), and $c$ (using modulo 2) at the start and immediately determine whether the answer is real or not. If it is guaranteed to be real, then $T_1$ is simply $1$, and the algorithm can continue without dealing with complex numbers at all. When $c$ is odd as in this case, $T_1$ reduces to an integer root of $-1$. In order to resolve the $d^{\text{th}}$ root of $-1$, we first need to represent $-1$ as a complex number in polar form:

$$ Z = \rho \bigl( \cos{\phi} + i\sin{\phi} \bigr) $$ 

$$ -1 + i0 = (1)\bigl( \cos{\pi} + i\sin{\pi} \bigr) $$

$$ -1 + i0 = (1)(-1 + i0) $$

$$ -1 = -1 $$

Then we can represent $-1$ in imaginary polar form using $\rho = 1$, $\phi = \pi$. Then, we can employ de Moivre's formula, the precursor to Euler's formula:

$$ Z^n = \Bigl[ \rho \bigl( \cos{\phi} + i\sin{\phi} \bigr) \Bigr] ^{n} = \rho^n \Bigl[ \cos{n\phi} + i\sin{n\phi} \Bigr] $$

Euler would come up with the modern form of this theorem 41 years after de Moivre, but this particular form is very useful because its inversion asserts that:

$$ Z^{1/n} = \Bigl[ \rho \bigl( \cos{\phi} + i\sin{\phi} \bigr) \Bigr] ^{1/n} = \text{ } \rho^{1/n} \Bigl[ \cos{\frac{\phi + 2\pi k}{n}} + i\sin{\frac{\phi + 2\pi k}{n}} \Bigr] , \text{ } k = 0,1,2,... $$

The power of de Moivre's formula is that it was the first to connect imaginary numbers to complex trigonometry. All imaginary numbers have $k$ $k^{\text{th}}$ roots, and the $k$ term adds rotations to the sinusoid terms. After $k$ roots, they start overlapping in the complex plane. This is why calculators will sometimes differ on imaginary results for certain root calculations of negative numbers. Some solutions might even have purely real roots, and some calculators are designed to report purely real results before reporting complex ones. Others, like WolframAlpha, will make all solutions available. For simplicty, this algorithm will only pull the first root that is the simplest to calculate ($k=0$), although it could be easily improved by recording all roots as WolframAlpha does.

Then, substituting the polar form of $-1$ and our own terminology, de Moivre's formula for $T_1$ simplifies to:

$$ T_1 = (-1)^{1/d} = (1)^{1/d} \Bigl[ \cos{\frac{\pi + 2\pi (0)}{d}} + i\sin{\frac{\pi + 2\pi (0)}{d}} \Bigr] $$

$$ T_1 = (-1)^{1/d} = \cos{\frac{\pi}{d}} + i\sin{\frac{\pi}{d}} $$

From here, we only need to resolve the cosine and sine terms once each to compute the root and entirely handle any complex number math, if necessary. Luckily, sine and cosine have well-defined Maclaurin series expansions, which converge very quickly. They are:

$$ \sin{x} = \sum_{n=0}^{\infty} \frac{(-1)^n}{(2n+1)!}x^{2n+1} = x - \frac{x^3}{3!} + \frac{x^5}{5!} - ... $$

$$ \cos{x} = \sum_{n=0}^{\infty} \frac{(-1)^n}{(2n)!}x^{2n} = 1 - \frac{x^2}{2!} + \frac{x^4}{4!} - ... $$

One interesting limitation of the modern computer is that most microprocessors use 64-bit registers, which puts a ceiling on the size of resolvable numbers. While that number is very, very large, the integer factorial actually caps at $20!$, which limits the sine and cosine Maclaurin series to ten terms each. In order to minimize error and cycle time, it becomes critical to limit the number of terms required for the Maclaurin series to converge. One small trick used by the algorithm is to detect $\frac{\pi}{d}$ arguments that are larger than $2\pi$ and subtract rotations until the argument sent to cosine and sine is small. This tends to save a handful of iterations from the number required for the series to converge.

At this point, all cases of $T_1$ are handled, including non-real ones. If $T_1$ is non-real, the algorithm will simply scale the real and imaginary parts of the result by the results of $T_2$ and $T_3$ at the very end.

To compute $T_2$ and $T_3$, recall:

$$ T_2 = \bigl( (a)^{(-1)^n} \bigr) ^{\frac{c}{d}} $$
       
$$ T_3 = \bigl( (\frac{1}{b})^{(-1)^n} \bigr) ^{\frac{c}{d}} $$

$$ \text{If } n=0 \text{: } (a)^{\frac{c}{d}} \text{, } (\frac{1}{b})^{\frac{c}{d}} $$

$$ \text{If } n=1 \text{: } (\frac{1}{a})^{\frac{c}{d}} \text{, } (b)^{\frac{c}{d}} $$

Therefore, $n$ simply determines which term, $a$ or $b$, is flipped. We may also show that, using the power of a quotient rule: 

$$ (\frac{1}{b}) ^{\frac{c}{d}} = \frac{1}{b^{\frac{c}{d}}} $$

Thus, we only need to calculate $a^{\frac{c}{d}}$ and $b^{\frac{c}{d}}$ and use $n$ to decide which one should divide $1$ as a final step; in other words, we may limit our discussion to $T_2$, because the calculation of $T_3$ is essentially identical.

The full decomposition (ignoring the $-1$ exponent, as it has been shown to be a trivial final step) is:

$$ T_2 = a^{\frac{c}{d}} $$

$$ \ln{T_2} = \ln{a^{\frac{c}{d}}} $$

$$ \ln{T_2} = \frac{c}{d}\ln{a} $$

$$ e^{\ln{T_2}} = T_2 = e^{\frac{c}{d}\ln{a}} $$

It is important to note that, because of the constraints put on $a$, $b$, $c$, and $d$ (that is, that they are guaranteed to be positive integers), the use of the natural logarithm is valid under its particular domain.

To resolve the new expression for $T_2$, we first need the natural logarithm. There are many techniques available for computing the natural logarithm -- this algorithm uses Halley's method, which is an augmented form of Newton's method that leverages the unique derivatives of natural log and the natural exponent:

$$ y = \ln{x} $$

$$ y_{n+1} = y_n + 2 \bigl( \frac{x - e^{y_n}}{x + e^{y_n}} \bigr) $$

Halley's method is known to converge relatively quickly with high precision, but testing showed that it tended to fail in practice when the initial guess value was below the final result for large numbers. Another small trick must be employed here to guarantee quick convergence. An odd fact of the natural logarithm is:

$$ lim_{x\to\infty} \bigl( \ln{x} \bigr) = \infty $$

$$ lim_{x\to\infty} \bigl( \frac{d}{dx}\ln{x} \bigr) = lim_{x\to\infty} \bigl( \frac{1}{x} \bigr) = 0 $$

The limit of the natural logarithm tending towards infinity is infinite, but its derivative (the slope of the curve) tends to zero -- which suggests that the natural logarithm is asymptotic, but how can this be? Essentially, what this is saying is that, as the input $x$ to the natural logarithm grows, the change in $x$'s natural logarithm is insignificant in comparison.

Consider this: the algorithm is designed to use 64-bit integers everywhere (unsigned long long int) to fully leverage modern computers and online C compilers, and the largest unsigned 64-bit integer is:

$$ 18446744073709551615 $$

And:

$$ \ln{18446744073709551615} = 44.36 $$

Then, the initial guess value in the Halley's method function was hard-coded to 45. This effectively guarantees convergence for any possible input.

For this and for the second step in the expression for $T_2$, the natural exponent is also needed. Thankfully, it also has a convenient Maclaurin series expansion:

$$ e^x = \sum_{n=0}^{\infty} \frac{x^n}{n!} = 1 + x + \frac{x^2}{2!} + \frac{x^3}{3!} + ... $$

This series is known to mathematicians to converge relatively quickly (in particular, for $0 \leq x \leq 1$), but again, the use of the factorial limits this series to 20 terms. One last trick leveraging the rules of exponentiation was used to speed up convergence:

$$ e^x = e^{x\frac{N}{N}} = \bigl( e ^{\frac{x}{N}} \bigr) ^{N}  $$

At the start of the function, a factor $N$ is used to scale down the exponent to $0 < x/N \leq 1$. Then, the result of the exponential is raised to the integer $N$ power using the recursive method.

Using these techniques, $T_2$ and $T_3$ can be calculated, and $n$ determines which one gets flipped. Finally, the conclusive result is computed by combining all three terms:

$$ X^Y = (T_1)(T_2)(T_3) $$

In review, the workflow for computing $X^Y$ is:

0. Catch and return trivial cases ($0^Y=0$, $X^0=1$, $1^Y=1$, $X^1=X$)
1. Factorize $X$ to $\frac{a}{b}$ and $Y$ to $\frac{c}{d}$, and record the sign variables $m$ and $n$
2. Use $m$ and $c$ to determine if $T_1$ (and therefore, the result) is potentially non-real
   - If it is, use de Moivre's formula to compute the real and imaginary components
       - Employ the cosine and sine Maclaurin series
   - If not, $T_1$ = 1
3. Calculate $a^{c/d}$ and $b^{c/d}$
   - Approximate $\ln{a}$ and $\ln{b}$ using Halley's method
   - Raise $e$ to $((c/d)\ln{a})$ and $((c/d)\ln{b})$
4. Use $n$ to determine which of $T_2$ or $T_3$ must be flipped
5. Combine terms: $X^Y = (T_1)(T_2)(T_3)$


