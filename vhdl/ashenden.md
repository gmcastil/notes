Chapter 1

A VHDL entity declaration describes the external view of the abstraction
referred to as an entity, that is defined by a module and its ports.

A VHDL architecture body of an entity describes that entities internal
implementation.  There may be a number of a different architecture bodies of the
one interface to an entity, which correspond to alternative implementations
which have the same function.

```vhdl
entity fdre is
  port ( d   : in  bit;
         clk : in  bit;
         q   : out bit );
end fdre

architecture basic of fdre is
begin
  fdre_basic : process is
  begin
    wait until clk;
    q  <= d after 2 ns;
  end process fdre_basic;
end architecture basic;

```

