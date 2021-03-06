# Calculating entropy

Summary: This article looks into how to calculate entropy. The article
includes example C code.

I was reading something and it mentioned in passing that you can
convert from base 10 to base 2 by simply dividing by the base to
convert to, and then collecting the remainder digits. So for base 10
to base 2 conversion you simply keep dividing by two and collecting
the remainders. I'd never heard of that before, so I had to try it out
in code immediately, as it seemed like magic. I spent a few minutes
coming up with some code to test it out and yes, it works! I then went
off on a bit of a tangent...

It seems like the sort of thing that should be doable in 3 lines or
so, but my version is a bit long-winded. It also seems like it should
be amenable to recursion too. Anyway, here's some noddy code:

```C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// size hardcoded at 8 bits

void to_binary (int *bin_array, unsigned int num)
{
    unsigned int base = 2;
    unsigned int res, rem;

    int i=7;
    while (num > 0)
    {
        res = num / base;
        rem = num % base;

        bin_array[i] = rem;
        i--;
        num = res;
    }
    
}

void clear_array (int *array)
{
    for (int i = 0; i < 8; i++)
    {
        array[i] = 0;
    }

}

void print_array (int *array)
{
    for (int i = 0; i < 8; i++)
    {
        printf("%d", array[i]);
    }
    printf("\n");    
}

int main (int argc, char **argv)
{
    int bin_array[8];

    clear_array(bin_array);
    to_binary(bin_array, 19);
    print_array(bin_array);
    
    return 0;
}
```

It's then possible to take this silliness a step further and add a
little tweak to tell us how many bits are required to code a
number. This is the entropy of a number:

```C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define BITS 8

void to_binary (int *bin_array, unsigned int num)
{
    unsigned int base = 2;
    unsigned int res, rem;

    int i=BITS-1;
    int entropy_bits = 0;
    while (num > 0)
    {
        res = num / base;
        rem = num % base;

        bin_array[i] = rem;
        i--;
        num = res;
        entropy_bits++;
    }
    printf("Entropy Bits: %d\n", entropy_bits);
}

void clear_array (int *array)
{
    for (int i = 0; i < BITS; i++)
    {
        array[i] = 0;
    }

}

void print_array (int *array)
{
    for (int i = 0; i < BITS; i++)
    {
        printf("%d", array[i]);
    }
    printf("\n");    
}

int main (int argc, char **argv)
{
    int bin_array[BITS];

    if (argc == 1)
    {
        printf("Example usage: to_binary 123\n");
        exit(-1);
    }

    clear_array(bin_array);
    to_binary(bin_array, atoi(argv[1]));
    print_array(bin_array);
    
    return 0;
}
```

A few runs of the program are shown here:

``` shell
bash-3.2$ ./a.out 63
Entropy Bits: 6
00111111
bash-3.2$ ./a.out 244
Entropy Bits: 8
11110100
bash-3.2$ ./a.out 19
Entropy Bits: 5
00010011
bash-3.2$
```

There is a mathematical formula for calculating entropy. The following
code shows this formula in action:

``` C
#include<stdio.h>
#include<stdlib.h>
#include<math.h>

int main (int argc, char **argv)
{

    if (argc == 1)
    {
        printf("Example usage: entropy 123\n");
        exit(-1);
    }

    
    double x = atof(argv[1]);;

    double log2x = log10(x)/log10(2);

    printf ("entropy: %f\n", log2x);
    
    return 0;
}
```
You can then compare the output between the two programs:

``` shell
bash-3.2$ ./to_binary 123
Entropy Bits: 7
bash-3.2$ ./entropy 123
entropy: 6.942515
bash-3.2$
```

You can see there is a slight mathematical 'glitch'. This can be fixed
with the `ceil()` function, which basically rounds up to the nearest
integer:

``` C
double log2x = ceil(log10(x)/log10(2));
```

Ah, but hang on, it looks like the formula might be wrong:

``` shell
bash-3.2$ ./to_binary 4
Entropy Bits: 3
00000100
bash-3.2$ ./entropy 4
entropy: 2.000000
bash-3.2$ 
```

We'll need to look into that a bit more ...

But it can be fixed easily enough like this:

``` C
double log2x = ceil(log10(x+1)/log10(2));
```

So, as per Shannon's law, the entropy of x is log2(x), and you use the
formula to work that out.

---

* Published: 2017-10-11 08:30:00 UTC
* Updated: 2017-11-15 17:30:00 UTC
* UUID: 09F99C84-B2DF-46C0-8030-8FBECE4A1590
