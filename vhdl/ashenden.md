Chapter 1

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

