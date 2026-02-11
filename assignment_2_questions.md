# Preamble: 

This assignment introduces the creation of custom statistics and SimObjects in gem5. In particular we will be implementing a [GShare](https://www.ece.ucdavis.edu/~akella/270W05/mcfarling93combining.pdf) branch predictor and modifying the statistics collected in the simulator to provide a more complete overview of branch prediction information. The following is a review from assignment 1:

In the C++ backend to gem5, every class has a Stat Group struct that stores the different statistics printed in your stats.txt . Statistics can be of several types, the most common being scalar, vector, distribution, and formula stats:

* **Scalar Stats:** Scalar stats are a single value and are commonly used for counters. They can be incremented and assigned to just like any C++ numeric value (`=`, `++`, `--`, ...).   
* **Vector Stats:** Vector stats are like an array of counters, and as an example, could be used to represent some counter over all the threads in the processor.    
* **Distribution Stats:** Distribution stats are indexable like vectors, but each index represents a “bucket”, which can correspond to a range.   
* **Formula Stats:** Formula stats are defined as equations that are evaluated at the very end of the simulation. The equation must consist solely of other stats, variables qualified with constant(), or just constants (1, 2, 3, etc).

To add stats to a `simObject`, You need to:

1. First declare the stats as attributes of a child class inheriting from the `statistics::Group` class, which is a container for all the stats in the `simObject`. This child class is located in the header file of the `simObject`.   
2. The stats should then be defined in the corresponding C++ file with the macro `ADD_STAT(..)`, which constructs the stat, and is meant to be used in the member initialization list of the child class. Most builtin `simObjects`, (such as the execute stage), have already been implemented, so you will only have to add yours to the existing `statistics::Group`.

The data for the statistics you will implement can be obtained in the IEW files (Instruction Execute and Writeback). To implement a statistic, you should first register the new statistics in `src/cpu/o3/iew.hh` along with the preexisting statistics in the struct `executeStats`. Then you should add your stats to the member initialization list in the `executeStats(...)` constructor in `src/cpu/o3/iew.cc`.
   
---

Note: They call branches **Conditional Control instructions**

Check the following documentation for gem5’s representation of instructions (`dyn_inst,` `static_inst`).   
There are methods which tell you if an instruction is a direct branch, what its branch target is, whether it was mispredicted, and whether it was taken.  
(`gem5/src`)  

[https://doxygen.gem5.org/develop/o3\_2dyn\_\_inst\_8hh\_source.html](https://doxygen.gem5.org/develop/o3_2dyn__inst_8hh_source.html)  
[https://doxygen.gem5.org/develop/cpu\_2static\_\_inst\_8hh\_source.html](https://doxygen.gem5.org/develop/cpu_2static__inst_8hh_source.html) 

Note that the method to get branch target returns a PCStateBase, which is a wrapper around an instruction address for the program counter.  
[gem5::PCStateBase Class Reference](https://doxygen.gem5.org/develop/classgem5_1_1PCStateBase.html)  
[https://doxygen.gem5.org/develop/base\_2types\_8hh\_source.html](https://doxygen.gem5.org/develop/base_2types_8hh_source.html)

---

# Section 1 (14 points) (stats) 

Question 1\. **(3 Points)** Implement the following 8 statistics for **Direct Branches** (branches whose destination is known at compile time). The stats should be implemented in the existing stat collection function, `updateExeInstStats()` in `iew.cc`: 

1. Total Forwards Taken   
2. Total Forwards Not Taken   
3. Total Backwards Taken   
4. Total Backwards Not Taken   
5. Mispredicted Forwards Taken   
6. Mispredicted Forwards Not Taken   
7. Mispredicted Backwards Taken   
8. Mispredicted Backwards Not Taken 

(In addition, you may want to collect misprediction rates here for **Q1. C**, however you can also calculate these manually)

1. **(1 point)** List the function which calls `updateExeInstStats()` in `iew.cc`.  

	  
	

2. **(2 points)** The IEW can receive squashed instructions, which should not be counted in the stats in the `updateExeInstStats()` function. How is this handled in the code? Provide a brief description.

 

Question 2\. **(8 points)** For each benchmark below, run the SiFiveP550 config with `10^8` instructions with the `LocalBP()` and the `BiModeBP()` branch prediction strategies separately.   
You will be running 2 different SPEC benchmarks: ([https://www.spec.org/cpu2017/Docs](https://www.spec.org/cpu2017/Docs))

1. **LBM**, “Lattice Boltzmann Method”, a method used to simulate incompressible fluids in 3D 

**`obtain_resource`**`("riscv-spec-lbm-run-se")`

2. The **MCF** program, “a program used for single-depot vehicle scheduling in public mass transportation”

**`obtain_resource`**`("riscv-spec-mcf-run-se")`  
(if you are curious these are found in `429-resources/benchmarks`)  
Prediction accuracy:

1. **(3 points)** Express your numbers in 8 tables (2 benchmarks x 2 predictors x 2 tables) of the following type:

| Total | Taken | Not Taken |
|-----------|-------|-----------|
| Forward   | 4,244,722 | 10,438,966 |
| Backward  | 2,400,275 | 556,458 |

| Mispredicted | Taken | Not Taken |
|-----------|-------|-----------|
| Forward   | 42,050 | 73,457 |
| Backward  | 8,835 | 8,749 |


(Report the frequencies of your branch prediction stats of the Cartesian product (forward, backward) x (taken, not taken) 

For LBM + LocalBP
| Total | Taken | Not Taken |
|-----------|-------|-----------|
| Forward   | 5149 | 16045 |
| Backward  |  557034 | 2437 |

| Mispredicted | Taken | Not Taken |
|-----------|-------|-----------|
| Forward   | 940 | 463 |
| Backward  | 72 | 162 |

-------------------------------------------------------------------------
For LBM + BiModeBP
| Total | Taken | Not Taken |
|-----------|-------|-----------|
| Forward   | 4148 | 17249 |
| Backward  |  556751 | 2758 |

| Mispredicted | Taken | Not Taken |
|-----------|-------|-----------|
| Forward   | 18 | 650 |
| Backward  | 6 | 418 |

-------------------------------------------------------------------------

For MCF + LocalBP
| Total | Taken | Not Taken |
|-----------|-------|-----------|
| Forward   | 5021280 | 10975324 |
| Backward  |  3866142 | 538945 |

| Mispredicted | Taken | Not Taken |
|-----------|-------|-----------|
| Forward   | 312594 | 708351 |
| Backward  | 91281 | 94122 |

-------------------------------------------------------------------------

For MCF + BiModeBP
| Total | Taken | Not Taken |
|-----------|-------|-----------|
| Forward   | 5300174 | 10755516 |
| Backward  |  3739711 | 531035 |

| Mispredicted | Taken | Not Taken |
|-----------|-------|-----------|
| Forward   | 172477 | 217518 |
| Backward  | 72708 | 14604 |


2. **(1 point)** Why is the total number of predicted branches here higher than the number of committed branches?  
     
     Predictions are made for some branches that are later squashed and not committed.
     
3. **(2 points)** Report the misprediction rates for the total forwards and the total backwards both branch prediction strategies.   
     For LBM + LocalBP => Forward -> (940 + 463)/(5149 + 16045) = 0.0662 ~ 6.62% 
   						  Backward-> (72 + 162)/(557034 + 2437) = 0.000418 ~ 0.042%
     For LBM + BiModeBP =>Forward -> (18 + 650)/(4148 + 17249) = 0.0312 ~ 3.12%
   						  Backward-> (6 + 418)/(556751 + 2758) = 0.000758 ~ 0.076%
     For MCF + LocalBP => Forward -> (312594 + 708351)/(5021280 + 10975324) = 0.0638 ~ 6.38%
   						  Backward-> (91281 + 94122)/(3866142 + 538945) = 0.0421 ~ 4.21%
     For MCF + BiModeBP =>Forward -> (172477 + 217518)/(5300174 + 10755516) = 0.0243 ~ 2.43%
   						  Backward-> (72708 + 14604)/(3739711 + 531035) = 0.0204 ~ 2.04%
   
     
     
5. **(2 points)** When considering the accuracy of predicting **forward and backward** branches separately, can you claim that one type was predicted better than the other? What about **predicted taken and not taken**? Can you conclusively conclude whether forwards or backwards branches are better predicted? Provide a hypothesis to explain your results.  
   * In particular, discuss whether or not your results support the idea that backward branches are taken more regularly than forward branches.
	
	Backward branches were predicted better than the forwards branches in all cases. From the table we can notice that backward branches are mostly taken, this is because backward branches are usually for/while 		loops which are mostly taken whereas the forward branches are more unpredictable.  
	  
Question 3\. **(3 points)** How often are branches committed in the provided benchmarks? 

1. **(1 points)** Provide the percentage of branches relative to the total number of instructions committed. (This is a built in stat in the commit stage)  

	562,027/100,000,001 = 0.562%
2. **(2 points)** Does this vary between branch predictors? (Ignore extremely small differences due to simulation artifacts.

	No, the percentage of branches committed does not vary between branch predictors. 

# Section 2 (13 points) (branch predictor)

Question 1\. **(7 points)** Gem5  branch predictor questions:

1. **(1 point)** Consider the bimodal predictor scheme you have discussed in class. Provide the name of an existing predictor in Gem5 whose implementation most closely/exactly implements this scheme. 

	  
	  2bit_local
	

2. **(1 point)** The Gem5 branch predictor API passes around branch prediction history with a (C) struct. Look at `bi_mode.cc`’s `BPHistory` struct, what is the only parameter you would need to implement a GShare branch predictor yourself?  
     
     globalHistoryReg
     
3. **(1 point)** Modern processors typically stick with phts with a power of two number of entries. However, if we really wanted a pht with `x` entries where `x` is not a power of two, this can be achieved by calculating the key, and taking it modulo `x`. Describe a potential problem with indexing a pht with non-power of two number of entries using the method in question f?  
     
     Using modulo with a non-power of two PHT can lead to uneven distribution of branch mapping and increased aliasing, potentially reducing accuracy.
     
4. **(1 point)** Let `i` be the number of local counter bits you have. Let `x` (i.e. `010`) be a saturating counter (think of this as an integer). Write a simple `C` expression that returns the prediction of the sat counter. (1 for taken, 0 for not taken)  
   [https://en.cppreference.com/w/c/language/operator\_arithmetic.html](https://en.cppreference.com/w/c/language/operator_arithmetic.html)  
     
     (x >= (1 << (i - 1)))
     
5. **(1 point)** Let `2^n` be the number of pht entries. Let PC be the address of the branch. Write a simple C expression(s) that returns the correct number of lower order bits of the PC.   
     
 	 PC & ((1 << n) - 1)

6. **(1 point)** Let h be the global history, which is `k` bits long. Let `b` be the `n` lower order bits of the PC. Assume `k` is at most `n`. The GShare predictor XOR’s the global history with the upper order bits of `b` to obtain the pht index. Write a simple C expression that calculates the PHT index using `h`, `b`, `n`, and `k`.  

	 ((( (b >> (n - k)) ^ h ) << (n - k)) | (b & ((1 << (n - k)) - 1)))

7. **(1 point)** Suppose the number of pht entries, `x`, is less than `2^n` and `x` is not a power of two.  Given an index value `i` calculated based on `n` as in the previous question, write an expression that maps all possible values of `i` to the range `[0, x)`.  
     
 	 i % x

8. **(2 point)** How do you calculate the number of index bits given the number of PHT entries? (The answer should be a common mathematical function.) How does this limit the number of entries a PHT can have?

	 Since hardware indexing uses binary address bits, this limits PHT sizes to powers of two. Therefore, number of index bits required for a PHT with N entries is ⌈log₂(N)⌉.
	
9. **(1 point)** Read the source files for the built in satCounter class in gem5. Does it implement the saturating counter state diagram discussed in class? If not, what is the scheme it follows?

	

10. Bonus: **(2 points)** The ChampSim simulator is another trace based CPU simulator. In its GShare branch predictor implementation, they fold the entire PC into their index. We will not be doing this, but   
      
    One way to do this is to take the lower (n-1) bits of the PC, XOR that with the next lowest n-1 bits, and then XOR with the next n-1 bits, until the entire PC has been folded. Write some C code that will put the folded result into a variable `b`, initialized to 0\. (HINT: Use a for loop)

**Preamble:**  
Implement GShare. In the GShare branch predictor, the branch address is XORed with the Global History Register (GHR). 

The values for:

* The PHT table size,  
* the global history length,   
* and the number of saturating counter bits

should be parameters (added in `gem5/src/cpu/pred/BranchPredictor.py`), and not hard coded. We recommend looking at other branch predictors in `gem5/src/cpu/pred` to help you design your GShare predictor.

To calculate the index of your PHT, XOR the global history with the upper bits of the branch address. Reference your answers to the previous questions.

You can easily create index masks in gem5 using the `mask(<number of bits>)`   
function. I.e. `mask(5) = 0b00011111`. 

Question 2\. **(3 points)** Create the tables for your `GShareBP()` implementation as in Section 1 Q2.

1. **(1 point)** Give tables  
     
     
     
2. **(2 points)** If you had to pick only one prediction scheme (Out of Local, TAGE, and GShare) that would be used for both benchmarks, which would it be and why?

# 

# Section 3 (16 points) (textbook-esque)

Question 1: **(3 points)** When the GShare branch predictor was created**[^1]**, it was introduced along with a sister algorithm, GSelect, which \<describe implementation\>

1. Explain why the GShare performs better than the GSelect? (Note you may have to consult the paper.)  

   The XOR of PC and GR provides more even distribution for the PHT whereas when concatenated in GSelect, it may lead to same entry for different branch pattern causing higher conflict aliasing, poor prediction
   and waste of storage.

Question 2: **(4 points)**  
The original paper proposing the GShare Branch predictor scheme also specifies that you can vary the amount of global history bits used. In the case that the number of global history bits is less than the number of index bits, the entire PHT is always indexed by the amount of index bits from the lower branch address, and the global history is XORed with the upper bits of the local index bits.

1. **(1 point)** Consider a branch with a taken/not taken pattern 0110011001100…, where 1 indicates taken, and 0 indicates not taken. Using the state transitions discussed in class, write out the states for a 2 bit saturating counter with an initial state of weak taken (10), when it encounters this pattern. What is the steady misprediction rate?

| T/NT | 0 | 1 | 1 | 0 | 0 | 1 | 1 | 0 | 0 | 1 | 1 | 0 | 0 | ... |
|------|---|---|---|---|---|---|---|---|---|---|---|---|---|-----|
| State|10 |00 |01 |11 |10 |00 |01 |11 |10 |00 |01 |11 |10 |00  |


in the pattern 0110 which has 4/4 predictions wrong
Therefore it has 100% misprediction rate

2. **(1 point)** Consider the same taken/not taken pattern as the previous question. Give the states a saturating counter using Gem5’s implementation would go through, and provide the steady misprediction rate. Assume the counter starts in the weak taken state (10) as before.

| T/NT | 0 | 1 | 1 | 0 | 0 | 1 | 1 | 0 | 0 | 1 | 1 | 0 | 0 | ... |
|------|---|---|---|---|---|---|---|---|---|---|---|---|---|-----|
| State|10 |01 |10 |11 |10 |01 |10 |11 |10 |01 |10 |11 |10 |01  |


in the pattern 0110 which has 3/4 predictions wrong
Therefore it has 75% misprediction rate

3. **(2 points)** Find a taken/not taken pattern that would result in a 100% steady misprediction rate on the Gem5 implementation and  a 50% steady misprediction rate on the version discussed in class. Once again, assume both predictors start in the weak taken (10) state.

	 01010101
   
Question 3 **(3 points)**:  
 A (3,2) correlator predictor is a correlator predictor that has a global register with three bits and in which each entry of the PHT has two bits. For a (3,2) correlated prediction scheme that has a global history register and one pattern history table, show the contents of the history register and the PHT after the following sequence of branch executions have taken place:

* 1 taken branch  
* 2 not-taken branches  
* 2 taken branches  
* 2 not-taken branches  
* 2 taken branches

Assume that:

* The prediction bits in the PHT use 2-bit prediction, implemented with saturating counters. The counters are incremented when a branch is taken and decremented when a branch is not taken. Each counter is initialized to the weak taken state, as illustrated in the textbook and in the lecture slides. Assume this state has the value 10\.  
* The global history register is initialized to zero.  
* The addresses of the nine branch instructions are: ffff0000, ffff0001, ffff0010, ffff0000, ffff0000, ffff0000, ffff0011, ffff0010, ffff0000... but do you need this information?  
* The PHT should be shown as updated after the execution of the last branch.

  	Global History Register = 011
  
  	Final PHT:

| Index | Counter |
|-------|---------|
| 000   | 11 |
| 001   | 11 |
| 010   | 01 |
| 011   | 01 |
| 100   | 11 |
| 101   | 10 |
| 110   | 01 |
| 111   | 10 |



**(2 points)** Question 4; Bonus for extra credit (not mandatory):  
After getting a sense of how prediction correctness varies with changes in the size of the hardware data structures the question that microarchitects usually face, is:

 "If I have a certain amount of chip area that I can devote to branch prediction, how should I divide it among the various branch data structures for correlating branch prediction, the global history registers, the pattern history table (and the branch target buffer)?"

With this question in mind, experiment with some different configurations for your gshare branch predictor. Specifically, try to find the optimal configuration of your gshare predictor using a limit of 4096 bits worth of space. You can limit the number of instructions on your simulation to some even smaller value than normal to perform many simulations and try many configurations (this is our way of telling you to do a parameter sweep\!).

**(1 point)** 1\. The best time to be critical and to offer constructive suggestions about an assignment is soon after you complete it. Please include in your report the following:

1. A narrative description of any difficulties or misunderstandings that made the assignment unnecessarily difficult, or that led you to waste time.  
2. Suggestions of changes that could make this assignment more interesting, more relevant or more challenging in future editions of this course.

**(2 points)** 2\. Make use of proper typesetting and overall style  
**(2 points)** 3\. Meeting notes**:** See the [collaboration](https://cmput429.github.io/429-website/docs/policies/assignments/collab) requirements for what to include.  
**(1 point)** 4\. Partner Evaluation: Please complete the [confidential partner evaluation](https://docs.google.com/forms/d/e/1FAIpQLScBJDjn2iV35X4DD5Bdua4ZEPAWeMQB9FZ-g3HCT2f_324fCw/viewform?usp=sf_link) for your partner. If you complete the evaluation, you get a point.

[^1]:  **1\.** McFarling, S.. Combining Branch Predictors. *WRL Technical Note TN-36*, 1993\.
