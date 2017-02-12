# Lots of LUTs
## The Truth is in the Tables
We know that all boolean functions can be expressed using only three basic
operators: and, or, not. each of these takes one or two boolean inputs and produces
a single boolean output. We often express these boolean operators and any boolean
functions composed from them as truth tables, which simply show which outputs are
produced for a given set or inputs.


And

| a | b | out |
| - | - | --- |
| 0 | 0 | 0   |
| 0 | 1 | 0   |
| 1 | 0 | 0   |
| 1 | 1 | 1   |

Or

| a | b | out |
| - | - | --- |
| 0 | 0 | 0   |
| 0 | 1 | 1   |
| 1 | 0 | 1   |
| 1 | 1 | 1   |

Not

| a | out |
| - | --- |
| 0 | 1   |
| 1 | 0   |

Now if we think of the input set as a single binary value, we find we can implement
truth tables using a hardware construct that you may be familiar with: ROM.

And

| address | out |
| ------- | --- |
| 00      | 0   |
| 01      | 0   |
| 10      | 0   |
| 11      | 1   |

Or

| address | out |
| ------- | --- |
| 00      | 0   |
| 01      | 1   |
| 10      | 1   |
| 11      | 1   |

Not

| address | out |
| --------| --- |
| 0       | 1   |
| 1       | 0   |

The input set becomes the address input of the ROM, and the output is simply the 
value stored at that address. Thus we could implement any boolean operation or
expression given a sufficiently large ROM.

## The Shape-shifting Gate
Now that we have established that we can use ROM as a substitute for traditional
logic gates, how do we create a programmable logic element that can be used in FPGAs?
The answer is RAM! RAM allows us to change the value stored at a specific address,
which means we can reprogram the truth table stored in it as we please. A given RAM
can be an and gate, an or gate, or any logical function that will fit in the table.

In fact, FPGAs contain [millions](https://www.altera.com/products/fpga/stratix-series/stratix-10/overview.html#family-table)
of small RAMs called **L**ook-**U**p **T**ables (**LUT**s). When chained in series or
parallel to other LUTs, they can be used to implement any logical function in the
same way gates would.

## No Free Lunch
The flexibility that LUTs give us over gates is not free. There are often significant
power and performance costs that come with it.

* Latency
* Area inefficient
* Expensive
* Power hungry
* Must be reprogrammed at power on






