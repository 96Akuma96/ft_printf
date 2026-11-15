*This project has been created as part of the 42 curriculum by zben-ham.*

# ft_printf

## Description

`ft_printf` is a recreation of the C standard `printf()`. The goal is to learn
how variadic functions work in C (`stdarg.h`) and to build a small, readable,
extensible library: `libftprintf.a`.

Supported conversions: `c` `s` `p` `d` `i` `u` `x` `X` `%`

It returns the number of characters printed, like the original. The buffering
of the original `printf()` is not implemented: output goes straight to
`write()`.

| File | Content |
|---|---|
| `ft_printf.h` | Prototypes and includes |
| `ft_printf.c` | `ft_printf`, the dispatcher `ft_convert`, and `ft_putptr` |
| `ft_print_utils.c` | `ft_putchar`, `ft_putstr`, `ft_putnbr`, `ft_puthex` |
| `Makefile` | Builds `libftprintf.a` with `ar` |

## Instructions

Build the library:

```
make
```

Other rules: `make clean`, `make fclean`, `make re`.

Use it in a program:

```c
#include "ft_printf.h"

int	main(void)
{
	ft_printf("Hello %s, you are %d years old (0x%x)\n", "world", 42, 42);
	return (0);
}
```

```
cc -Wall -Wextra -Werror main.c -L. -lftprintf -o test
./test
```

## Algorithm and data structure

**Parsing.** `ft_printf` walks through the format string with a single index.
A normal character is printed directly. On a `%`, the next character is the
conversion specifier and is handed to a dispatcher (`ft_convert`). A lone `%`
at the very end of the string stops the loop and prints nothing, as in glibc.

**Variadic arguments.** One `va_list` is initialised with `va_start` and passed
to the dispatcher **by pointer**, so each `va_arg` advances the same cursor.
Passed by value, every conversion would read a copy and get the same argument
again. `va_end` is called before returning.

**Dispatcher.** One `if` per specifier. Each reads its argument with the
correct type (`int` for `c`, `d`, `i`; `char *` for `s`; `void *` for `p`;
`unsigned int` for `u`, `x`, `X`) and calls the matching print function.

**Return value.** Every print function returns the number of bytes it wrote
and `ft_printf` sums them, so no global variable is needed.

**Number printing (recursion).** `ft_putnbr` and `ft_puthex` use the same idea:
if the number is at least the base, call the function on `n / base` first,
then print the last digit `n % base`. The deepest call prints first, so digits
come out in the right order without a buffer or a reversing step. For
hexadecimal, a lookup string (`"0123456789abcdef"` or the uppercase one) maps
a digit value to its character. It is passed as a parameter so `%x` and `%X`
share one function.

**Why `long` in `ft_putnbr`.** It takes a `long` so that:
- `INT_MIN` can be negated without overflow, with no special case;
- values up to `UINT_MAX` fit, so `%d`, `%i` and `%u` share one function.

`ft_puthex` takes an `unsigned long` so it can also print 64-bit pointers.

**`%p`.** Prints `0x` followed by the address in lowercase hexadecimal. A `NULL`
pointer prints `(nil)` and a `NULL` string for `%s` prints `(null)`, matching
glibc.

**Why this design.** Recursion keeps every function short and needs no memory
allocation, so no leaks are possible. Dispatching in one place makes adding a
conversion easy: one new `if` and one new print function.

## Resources

**References**
- `man 3 stdarg`
- `man 3 printf`
- `man 2 write`
- The 42 Norm

**Use of AI (Claude)**

| Part of the project | How AI was used |
|---|---|
| Understanding `stdarg.h` | Explained `va_list`, `va_start`, `va_arg`, `va_end`. I practiced with toy functions (`sum`, `max`) that I wrote and debugged with AI hints. |
| Testing and review | AI helped design test cases against the real `printf()` (`INT_MIN`, `UINT_MAX`, `0`, `NULL` string and pointer, lone `%`) and caught a header/prototype mismatch and Norm errors. |


All output and return values were compared against the original `printf()`,
and the code was checked with `norminette`.