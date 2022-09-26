# Chapter 1

## VHDL Modeling Concepts

A VHDL entity declaration describes the external view of the abstraction
referred to as an entity, that is defined by a module and its ports.

A VHDL architecture body of an entity describes that entities internal
implementation.  There may be a number of a different architecture bodies of the
one interface to an entity, which correspond to alternative implementations
which have the same function.

```vhdl
-- This is an entity declaration
entity fdre is
  port ( d   : in  bit;
         clk : in  bit;
         q   : out bit );
end fdre;

-- This is an architecture body, which describes an implementation of the
-- entity function (there may be more than one)
architecture basic of fdre is

begin
  fdre_basic : process is
  begin
    wait until clk;
    q  <= d after 2 ns;
  end process fdre_basic;

end architecture basic;

-- This is a component instance - note that it uses the `entity` keyword,
-- references the current workspace (hopefully more on this later), and the use
-- of the `port map` construct, which presumably uses port ordering to determine
-- how entity ports are structurally connected
architecture struct of reg1 is
  signal clk_in : bit;

begin
  d0 : entity work.fdre(basic)
         port map (d0, clk_in, q0);
end architecture struct;

```

The author makes an effort to identify three stages of simulation:
1. Analysis
2. Elaboration
3. Execution

Analysis is essentially syntactic and and semantic analysis for the purposes of
identifying errors. Notably, if this step is successful, an intermediate
representation of the unit is created and stored in a library.

Elaboration is the act of working through the design hierarchy and creating all
of the objects defined in declarations with the end result being a collection of
signals and processes, and possibly variables.  In order to be simulated, a
model must be reducible to a collection of signals and processes.

The final stage is the execution of the model.  This section contains a couple
of paragraphs about how simulation engines work that may be useful for future
reference.  For reasons that are beyond my current understanding, languages that
were originally intended for the simulation of hardware models were chosen as
the languages to describe how to build hardware to a synthesis engine.  Talk
about crazy.

## Lexical Elements and Syntax

Comments are as one would expect, with the caveat that nested `/* ... */` style
comments are not supported for obvious reasons.  Also, VHDL-87, -93, and -2002
only support single-line comments.  It looks like it wasn't until much later
that support for `/* ... */` style comments were added.

Identifiers have the same basic rules as one would expect, with the massive
proviso that values are case-insensitive.  The same nonsense of extended
identifiers in VHDL that we find in Verilog exists as well, and these *are* case
sensitive.  So, if you're writing synthesizable code and using extended
identifiers, then you deserve whatever happens to you.  And of course, VHDL-87
does not support this kind of foolishness.

The descriptions in the text about which keywords are used by which version of
the language are horribly written so take care in referring to this section in
the future.

Special symbols are about as one might expect from similar languages

Strings, numbers (real and integer literals), and character literals are similar
to C. Bases are expressed using the sharp (_#_) character, with the base
preceding it.  Examples are `16#DEADBEEF`, `2#1010_1010`, and `10#1234`.  Real
numbers and exponentials are also supported.

Bit strings are represented in a similar fashion, but with the values in
quotation marks and the base specified with a letter.  Examples are
`B"0100_0100"`, `X"DEAD_BEEF"`.  The difference between numbers and bit strings
is that bit strings are represented by a number of bits (as opposed to reals,
which might not be able to be done this way).  It doesn't appear that something
like `2#0001` is equivalent to `4B"0001"` since the latter is explictly defined
as 4 bits and the former is an integer literal and only requires one bit in
order to fully represent.  It might be interesting to see if these are equal in
a simulator.  Also, signed and unsigned support appears to have been added to
bit string literals in VHDL versions after VHDL-2002.

## Extended Backus Naur Form (in brief)

This is relatively useful summary of the rules of EBNF required to understand
(hopefully) the rest of the text.

# Chapter 2

Four types of objects:
- Constants
- Variables
- Signals
- Files

Note that constants and varialbes are the only objects that can store data for
use in a model.  These kinds of distinctions were not always clear to me.  The
text saves signals for chapter 5 and files for much later.

## Constants and Variables

Constants differ from literals in that they actually have a type associated with
them, which is helpful.  Variables cannot be accessible by more than one
process, which is somewhat unclear if variables can be read by more than one
process.  Probably, what is meant is that they cannot be assigned to by more
than one process.

Distinction between signal and variable assignment.  A variable assigment
(:=) immediately alters the value of the variable and replaces it with the new
value.  A signal assignment (<=) schedules the assignment to be made at some
later time.

My understanding thus far is that this construct in VHDL
```vhdl
  variable foo : integer;
  foo := 16x"1234";
```
is syntactically the same as this one in Verilog
```verilog
  integer foo;
  assign foo = 16'h1234;
```
These don't seem too bad, although it's unclear how initialization works when it
comes to synthesis.
