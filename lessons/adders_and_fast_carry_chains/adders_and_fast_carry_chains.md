# Adders and Fast Carry Chains: Addition in FPGAs
## The Full Adder:  1-Bit at a Time
The [full adder](https://en.wikipedia.org/wiki/Adder_(electronics)) is the building block
of integer arithmetic in digital circuits. It consists of 3 1-bit inputs (a, b, carry_in) and one
2-bit output (sum).

| a | b | carry_in | sum[1:0] |
|:--|:--|:---------|:---------|
| 0 | 0 | 0        | 00       |
| 0 | 0 | 1        | 01       |
| 0 | 1 | 0        | 01       |
| 0 | 1 | 1        | 10       |
| 1 | 0 | 0        | 01       |
| 1 | 0 | 1        | 10       |
| 1 | 1 | 0        | 10       |
| 1 | 1 | 1        | 11       |

We can see from the truth table that sum is simply the 1-bit binary sum of three 1-bit integers
(a, b, carry_in). We often call the most significant bit (MSB) of sum (sum[1]) carry_out. Just as
with base 10 addition, we call the extra digit/bit produced by an addition a carry.

| a | b | carry_in | carry_out | sum |
|:--|:--|:---------|:----------|:----|
| 0 | 0 | 0        | 0         | 0   |
| 0 | 0 | 1        | 0         | 1   |
| 0 | 1 | 0        | 0         | 1   |
| 0 | 1 | 1        | 1         | 0   |
| 1 | 0 | 0        | 0         | 1   |
| 1 | 0 | 1        | 1         | 0   |
| 1 | 1 | 0        | 1         | 0   |
| 1 | 1 | 1        | 1         | 1   |

Like any other simple logic function we can implement a full adder using one or more
[LUTs](https://github.com/s-okai/hello-fpga/blob/master/lessons/lots_of_luts/lots_of_luts.md)
(almost all modern FPGAs will be able to implement a full adder using a single LUT). Now that we can
add 1-bit integers on an FPGA, how do we add n-bit integers? And why do we need carry_in? Why add
three 1-bit integers instead of two?

## Making Ripples in a Pond
In order to understand how we add two n-bit binary integers, it is helpful to think about how we add
two base 10 integers. We start from the right most (least significant) digits, we add them together
to get the least significant digit of the sum, and carry over a 1 if the sum is greater than 9. We
then add the carried 1 and the digits directly to the left to get the next digit of the sum. We
repeat this until we have hit run out of digits. Our addition for each digit of the sum is
essentially the sum of the two corresponding digits of the values we are trying to add and the
"carry in" from the previous digit (which is either 0 or 1). The special case is the least
significant digit, which always has a carry in of 0.

N-bit binary addition is done in the exact same way, except our digits can only be 0 or 1 instead of
0-9, and we carry a 1 to the next digit if the sum for the digit is greater than 1. We can thus
break the problem of addition into n 1-bit additions, each of which will have a carry in of 0 or 1
from the previous bit and produce a carry out of 0 or 1 to be passed to the next bit. This is
exactly what our full adder does! We simply need to chain the carry_out of each full adder to the
carry_in of the next, where the least significant bit (LSB) is once again the special case and
always has a carry_in of 0. This chain of 1-bit fuller adders is called a [ripple carry adder](http://www.electronics-tutorials.ws/combination/comb_7.html). It gets its name from the fact
that the carry "ripples" down the _carry chain_ from carry_in to carry_out of each full adder,
starting from the LSB and ending at the MSB.

TODO: Insert ripple carry adder diagram.

## Following the (Carry) Chain
The ripple carry adder gets its name from the wave front we create when we add one bit at a time.
Thus, we cannot add the next bit until the carry_out of the previous bit is calculated. Since there
is some delay inherent to any logic or interconnect, we see that the time it takes to complete an
addition is O(n) (that is, it is linear with respect to the number of bits we are adding), with the
longest path being from the inputs of the LSB to the outputs of the MSB. This can impose severe
limitations on our maximum clock frequency for large n. Surely there must be a better solution for
an operation that is so common in FPGA designs?

TODO: insert longest path diagram here

In fact, there is a solution: make the carry chain _fast_. Minimizing the delay from adder to adder
along the carry chain can help to keep the total delay in check. FPGA designers purposely minimize
the delay of a special set of interconnect between LUTs used specifically for carry chains. Since
we only care about the longest delay path when determining the maximum clock frequency of a design,
it makes perfect sense for FPGA designers to prioritize the carry chain over other interconnect.

There are limits to this: for wide integers and high clock frequencies, the delay along the carry
chain may still be too long. For these cases, there are other solutions such as pipelining and
[DSP blocks](https://www.altera.com/products/fpga/features/dsp/arria-v-cyclone-v-dsp-block.html).
However, many of the additions in a typical design are small enough such that fast carry chains
are more than adequate.

## References
* [Binary Adders](http://www.electronics-tutorials.ws/combination/comb_7.html)
