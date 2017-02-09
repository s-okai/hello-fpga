# Multiplexers and Latches
## The Humble Multiplexer
The 2-input multiplexer (or mux for short) is one of the basic building blocks of RTL design. It
takes 2 n-bit inputs (we will call them a and b) and a select bit (sel) and has a single n-bit
output (out). If sel = 0, out is set to a, otherwise it is set to b.

There are several ways to implement a mux. A common choice is the humble if-else statement.

SystemVerilog:
```verilog
logic [n-1:0] a;
logic [n-1:0] b;
logic sel;
logic [n-1:0] out;

always_comb
    if sel == 0 begin
        out <= a;
    end else begin
        out <= b;
    end
```

VHDL
```vhdl
signal a : std_logic_vector(n-1 downto 0);
signal b : std_logic_vector(n-1 downto 0);
signal sel : std_logic;
signal out : std_logic_vector(n-1 downto 0);

process (a, b, sel) begin
    if !sel then
        out <= a;
    else
        out <= b;
    end if;
end process;
```

MyHDL
```python
def mux(a, b, sel, out):
    @always_comb
    def logic():
        if not sel:
            out.next = a
        else:
            out.next = b
    return logic
```

## Latches
You can also have m-input n-bit muxes. Here's what this looks like for m = 3.

SystemVerilog:
```verilog
logic [n-1:0] a;
logic [n-1:0] b;
logic [n-1:0] c;
logic [1:0] sel;
logic [n-1:0] out;

always_comb
    if sel == 0 begin
        out <= a;
    end else if sel == 1 begin
        out <= b;
    end else if sel == 2 begin
        out <= c;
    end
```

VHDL
```vhdl
signal a : std_logic_vector(n-1 downto 0);
signal b : std_logic_vector(n-1 downto 0);
signal c : std_logic_vector(n-1 downto 0);
signal sel : std_logic_vector(1 downto 0);
signal out : std_logic_vector(n-1 downto 0);

process (a, b, c, sel) begin
    if sel = b"00" then
        out <= a;
    elsif sel = b"01" then
        out <= b;
    elsif sel = b"10" then
        out <= c;
    end if;
end process;
```

MyHDL
```python
def mux(a, b, c, sel, out):
    @always_comb
    def logic():
        if sel == 0:
            out.next = a
        elif sel == 1:
            out.next = b
        elif sel == 2:
            out.next = c
    return logic
```

Some of you may have noticed an issue with the implementations above: we've inferred a latch! How?
Well let us look step through the cases one at a time:
* If sel = 0, then out is set to a.
* If sel = 1, then out is set to b.
* If sel = 2, then out is set to c.
* If sel = 3, then out is set to ???.

Wait a minute, what should happen to out when sel = 3? Some of you are probably thinking "obviously
it keeps its previous value." And you would be mostly correct. It will keep the previous value that
it was assigned to. However, remember that "keeping its previous value" implies that out is
stateful. The only way in which this is possible is to have a memory element, in this case a latch,
which holds the previous value of out. Since out implementation is now has a memory element in it,
it is no longer combinational (remember, combinational logic cannot contain memory elements). In
fact, if you tried to compile the SystemVerilog example, you would find that it would give you an
error saying that the implementation is not combinational (the others would only give warnings that
a latch was inferred). It is also worth noting that we should always avoid latches, especially in
FPGA implementations, because their behavior tends to be unpredictable and because FPGAs do not
contain latches as native memory elements.

## Where did we go wrong?
When we wrote out 3-input mux, our if statements were not "assignment complete." That is, out is
not assigned for all possible inputs, in this case when sel = 3. If the 3-input mux case, this is
simple to see since which value out is assigned is dependent only on sel. But at some point we may
have a case like this:

SystemVerilog:
```verilog
logic [n-1:0] a;
logic [n-1:0] b;
logic sel0;
logic sel1;
logic [n-1:0] out;

always_comb
    if sel0 begin
        out <= a;
    end else if sel1 begin
        out <= b;
    end
```

VHDL
```vhdl
signal a : std_logic_vector(n-1 downto 0);
signal b : std_logic_vector(n-1 downto 0);
signal sel0 : std_logic;
signal sel1 : std_logic;
signal out : std_logic_vector(n-1 downto 0);

process (a, b, sel0, sel1) begin
    if sel0 = '1' then
        out <= a;
    elsif sel1 = '1' then
        out <= b;
    end if;
end process;
```

MyHDL
```python
def foo(a, b, sel0, sel1, out):
    @always_comb
    def logic():
        if sel0:
            out.next = a
        elif sel1:
            out.next = b
    return logic
```

This is still a fairly elementary example, but we see that we can have if statements based on
independent inputs which can add even more confusion to the mix.

## How do we prevent this?
The easiest way to make sure that we do not have "assignment incomplete" if statement is to *always*
include and else case in which we assign outputs with default values. In the case of the 3-input
mux, we can simply map both sel = 2 and sel = 3 to input c.

SystemVerilog:
```verilog
logic [n-1:0] a;
logic [n-1:0] b;
logic [n-1:0] c;
logic [1:0] sel;
logic [n-1:0] out;

always_comb
    if sel == 0 begin
        out <= a;
    end else if sel == 1 begin
        out <= b;
    end else begin
        out <= c;
    end
```

VHDL
```vhdl
signal a : std_logic_vector(n-1 downto 0);
signal b : std_logic_vector(n-1 downto 0);
signal c : std_logic_vector(n-1 downto 0);
signal sel : std_logic_vector(1 downto 0);
signal out : std_logic_vector(n-1 downto 0);

process (a, b, c, sel) begin
    if sel = b"00" then
        out <= a;
    elsif sel = b"01" then
        out <= b;
    else then
        out <= c;
    end if;
end process;
```

MyHDL
```python
def mux(a, b, c, sel, out):
    @always_comb
    def logic():
        if sel == 0:
            out.next = a
        elif sel == 1:
            out.next = b
        else:
            out.next = c
    return logic
```

Note that similar issues can arise when using case statements, so make sure to always include a
default case when using them.

## Summary
* Assignment incomplete if statements lead to inferred latches. This can happen for any conditional
structure,including case statements.
* Always include an else or default case in which you assign default values for all outputs.
