# Variables and Blocking
## VHDL Variables
VHDL is my favorite HDL because, despite how clumsy it can be, there are some very nice tricks you
can use which have no Verilog/SystemVerilog equivalent. One of these is variables.

Consider the following example:
```vhdl
process (clk) begin
    if rising_edge(clk) then
        if sel0 = '1' then
            x <= (a + b) + c;
        else
            x <= (a + b) - c;
        end if;
    end if;
end process;
```

We can see here that the expression (a + b) is used in both cases in both the if and else cases.
It would be nice if we could write this without and code duplication.

```vhdl
a_plus_b <= a + b;

-- A bunch of code here...

process (clk) begin
    if rising_edge(clk) then
        if sel0 = '1' then
            x <= a_plus_b + c;
        else
            x <= a_plus_b - c;
        end if;
    end if;
end process;
```

This is better. However, we have now separated the expression (a + b) from the code block dependent
on it. In this case it does not matter all that much because a_plus_b makes it very clear what
the expression is, but there are a lot of other cases where we might have a more complicated
expression that is not easily described by a signal name.

```vhdl
process (a, b) begin
    if sel1 = '1' then
        complex_logic <= a << 1;
    else
        complex_logic <= b >> 1;
    end if;
end process;

-- A bunch of code here...

process (clk) begin
    if rising_edge(clk) then
        if sel0 = '1' then
            x <= complex_logic + c;
        else
            x <= complex_logic - c;
        end if;
    end if;
end process;
```

Now this becomes problematic. It is not clear what complex_logic is unless we actually look at where
is assigned. This is where variables are useful.

```vhdl
process (clk)
    variable complex_logic : std_logic_vector(7 downto 0);
    begin
    if rising_edge(clk) then
        if sel1 = '1' then
            complex_logic := a << 1;
        else
            complex_logic := b >> 1;
        end if;

        if sel0 = '1' then
            x <= complex_logic + c;
        else
            x <= complex_logic - c;
        end if;
    end if;
end process;
```

Using a variable complex_logic, we've eliminated the need for an extra process.

### Signals vs. Variables
But wait! Shouldn't the value of complex_logic be updated when the process has finished execution
like any other signal assigned in a process? The answer is no: unlike signals, variables are always
*updated at assignment* rather than at the end of process execution. There a few consequences of
this behavior:
* **Variables are always treated as the outputs of combinational logic, even when they are in a
sequential block.** Combinational outputs are always a pure function of their inputs. That is, their
values always reflect the values of their inputs at that moment, and thus, they are state-less.
Because variables are updated at assignment rather than at a clock edge (or some other synchronous
event), they too always reflect the values of their inputs at that moment.

* **The scope of a variable is always limited to the process it is declared in.** If a signal x is
assigned in a clocked process, then x is by definition a direct output of a flip-flop, since its
value is only updated on a rising clock edge. But what about variables such as complex_logic? It is
clearly assigned in a clocked process, yet we stated above that we can treat it as an output of
combinational logic. In order to resolve this contradiction, the designers of VHDL chose to [only
allow variables to be declared in processes, function, and
procedures.](http://www.ics.uci.edu/~jmoorkan/vhdlref/var_dec.html) Since variables cannot be
referenced outside of a process, it prevents the unwitting developer from mistakenly using it as the
output of a flip-flop. This is contrary to blocking assignment in Verilog/SystemVerilog. More on
this below.

## Blocking Assignment in Verilog/SystemVerilog
If you are a Verilog/SystemVerilog developer, you may have been upset by the fact that I stated that
Verilog/SystemVerilog has no equivalent to VHDL variables. "Just use blocking (=) assignment!" you
might say. However, blocking assignment is not equivalent to a variable, and is, in fact, much more
dangerous to use.

Let's consider the example above, implemented in SystemVerilog.
TODO: switch to systemverilog syntax
```verilog
always_ff @(posedge clk) begin
    if sel1 begin
        complex_logic = a << 1;
    end else begin
        complex_logic = b >> 1;
    end

    if sel0 begin
        x <= complex_logic + c;
    end else begin
        x <= complex_logic - c;
    end
end
```

It works! All we have to do is substitute the variable complex_logic with blocking assignments,
and we get the same behavior. However, there is a slight problem. Remember when we said that a
VHDL variable cannot be referenced outside of a process to avoid being interpreted as both an
output of combinational logic and an output of a flip-flop? That is not true for blocking assignment
in Verilog/SystemVerilog. If a signal is referenced below the blocking assignment, inside of the
same sequential block, it is interpreted as an output of combinational logic just as a VHDL variable
would. However, if it is referenced above the blocking assignment, or outside of the sequential
block containing the blocking assignment, it is interpreted as the output of a flip-flop. This
effectively means that there are two signals with the same name: one is the output of combinational
logic, and the other is the output of a flip-flop that is fed by the same combinational logic. With
blocking assignment, where we reference a signal matters. This is misleading, and thus should be
avoided at all costs.

```verilog
always_ff @(posedge clk) begin
    if sel1 begin
        complex_logic = a << 1;
    end else begin
        complex_logic = b >> 1;
    end

    if sel0 begin
        x <= complex_logic + c; // complex_logic is combinational here.
    end else begin
        x <= complex_logic - c;
    end
end

always_ff @(posedge clk) begin
    if sel0 begin
        y <= complex_logic + c; // complex_logic is the output of a flip-flop here.
    end else begin
        y <= complex_logic - c;
    end
end
```

## References
* [VHDL variables](http://www.ics.uci.edu/~jmoorkan/vhdlref/var_dec.html)
