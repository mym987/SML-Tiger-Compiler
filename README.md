# SML-Tiger-Compiler
Written by Jordan Ly (jl455), Kevin Do (kkd10), and Cody Lieu (cal53)

A basic compiler for Andrew Appel's Tiger language written in SML for ECE 553 Compiler Construction.

This compiler was tested on SML/NJ. The assembly produced can be run in QtSpim as-is. The final compiler is in the `reg` folder.

To test, run `sml < a` at the shell. This compiles `k1.tig`, producing `k1.tig.s` and printing some diagnostic output. Note that `k1.tig.s` includes the Tiger runtime and standard library. The user-compiled code is at the label `tig_main`.

## List of Optimizations

#### Type-checker
Purely functional records, instead of using a (mutable) name type to handle recursive record types

#### IR
##### Dead-code elimination / Code simplification
```
	if 0 then _ else _ => JUMP not CJUMP (test101.tig)
	if 1 then _ else _ => JUMP not CJUMP (test102.tig)

	WHILE 0 => no code or labels (test113.tig)
	WHILE 1 => JUMP not CJUMP at end (test114.tig)

	FOR with low > hi => (test115.tig)
	FOR with low = hi => no jumps (test116.tig, test117.tig)

	ARRAY[0] => doesn't generate mult instruction (test118.tig)

	add by 0 => no add instruction (test120.tig)
	subtract by 0 => no sub instruction (test121.tig)
	
	mult by 0 => 0 (test122.tig)
	mult by 1 => no mult (test123.tig)
```

##### Strength reduction
```
	mult by power of 2 => LSHIFT not MULT (test119.tig)
```
	
##### Constant Folding
```
	CONST + CONST => CONST not BINOP (test103.tig)
	CONST - CONST => CONST not BINOP (test104.tig)
	CONST * CONST => CONST not BINOP (test105.tig)
	CONST / CONST => CONST not BINOP (test106.tig)
	CONST = CONST => JUMP not CJUMP (test107.tig)
	CONST <> CONST => JUMP not CJUMP (test108.tig)
	CONST < CONST => JUMP not CJUMP (test109.tig)
	CONST <= CONST => JUMP not CJUMP (test110.tig)
	CONST > CONST => JUMP not CJUMP (test111.tig)
	CONST >= CONST => JUMP not CJUMP (test112.tig)
```

##### Miscellaneous
```
	String literals get allocated only once per unique string (test100.tig)
```

#### Instruction Selection
```
Tr.CONST 0 uses $zero, not li into some register (test124.tig)
```

#### Register allocation
```
Temps are colored with registers in the same order each time. This maximizes register reuse, so that we save the "higher-numbered" registers for when there is a lot of register pressure.

Coalesce (test101.tig). Notice that the answer 4 is immediately assigned to $v0, not to some other temp than moved to $v0.
```

#### Proc entry exit
```
Callee-saved registers (the $s registers) are only saved and restored at the beginning and end of a function if they are actually used. (Notice how test101.tig does not save any $s registers, but merge.tig does)
```
