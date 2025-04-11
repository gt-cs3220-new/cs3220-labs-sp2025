# CS3220 Lab #6 : Systolic Array Design I

100 pts in total, will be rescaled into 13% of your final score of the course.  

**Part 1: Systolic Array**: 100 pts

**Bonus: Non-squared Systolic Arrays**: 20 pts

***Submission ddl***: Apr. 14th

In this lab, you will explore the design of a systolic array and its integration with a RISC-V CPU using the AXI4 protocol. Systolic arrays, first introduced in the late 1980s, have since become a fundamental tool for a wide range of applications, spanning from scientific computing to machine learning. The core concept revolves around employing a regular grid of simple processing elements (PEs) to execute computations in a data-flow fashion. These PEs are interconnected in a structured manner, facilitating the streaming of data through the array. The systolic array architecture excels in efficiently handling tasks such as matrix multiplication, convolution, and other linear algebra operations. An exemplary illustration of this concept can be found in the Google TPU, which leverages systolic arrays and vector machines. One of the major key features of systolic arrays is the systolic data propagation, which is a very efficient and scalable way to move data through the array, which we will focus a lot in this lab.

## Part 1: Systolic Array:

In this section, you'll finish the implementation of a MxN systolic array, with output stationary design. We will focus only on the matrix multiplication operation. 

**IMPORTANT!! Otherwise you will be lost during this lab! Please be sure to read through the original systolic array paper before you start:** 

- [link1](https://www.princeton.edu/~kung/papers_pdf/New%20Folder/VLSI%20Array%20Processors.pdf) 

To enhance your comprehension and offer more specific guidance on implementation, we have included a tutorial report detailing the systolic array design and implementation. This report will direct you on where to begin and outline the necessary steps:
- [link2](CS3220_systolic_array_luke_zhang.pdf)



### Problem Description


The code skeleton is provided in: 
- [systolic_array.v](systolic_array.v)
- [MAC.v](MAC.v)
- [ctrl.v](ctrl.v)
- [adder.v](adder.v)
- [multiplier.v](multiplier.v)


Out of the above files, you will need to work on [systolic_array.v](systolic_array.v), [MAC.v](MAC.v), and [ctrl.v](ctrl.v) to complete the systolic array design for Part 1.


Finish all the TODOs in the code; or you can start from scratch if you prefer. The only major requirement is that you are not allowed to change the interface of the systolic array [systolic_array.v](systolic_array.v). Although we provide the interfaces for [MAC.v](MAC.v) and [ctrl.v](ctrl.v), you are free to change them as you see fit.

Todos in the skeleton codes provided are intended to guide you through the implementation process and provide some hints. However, you are free to modify the code as you see fit.



Useful hints:
- Pay very special attention to the timing of the signals. At what clock cycle should the signals be expected?
- What if two signals are expected to arrive at the same time? But one of them is a little falling behind? Remember Flip-Flop is a very useful tool to help you with this; output of a Flip-Flop is always one clock cycle behind the input; can you use this to help you with the timing?
- Generate block in Verilog is a good way to save repetitive code.
- When debugging, start with small sized systolic array, and zoom in into a single MAC unit. This will help you understand the timing of the signals.
- More hints may be available in the code skeleton, please read through them carefully.


### Testing Procedures

Testing pattern and assumptions:
- The input data will be pre-skewed and pre-rotated. You don't need to implement the skewing and rotating logic. For example:
    - For matrix multiplication A*B=C: 
![Alt text](imgs/m1.png)
    - A: Streamed to ```row_data_in``` [systolic_array.v](systolic_array.v) from left to right as (note topmost is the first row): ![Alt text](imgs/m2.png)
    - B: Streamed to ```col_data_in``` [systolic_array.v](systolic_array.v) from top to bottom as (note leftmost is the first column): ![Alt text](imgs/m3.png)
    - C: Expected to be streamed out from ```row_data_out``` [systolic_array.v](systolic_array.v) from left to right as (note topmost is the first row): ![Alt text](imgs/m4.png)
- Multiple matrix multiplication operations will be performed in sequence. Your design is expected to be able to handle the next matrix multiplication operation immediately after the previous one is finished.
    - Assume the same matrix multiplication operation as above is performed again, the input data will be streamed in the same way as above.
    - A: Streamed to ```row_data_in``` [systolic_array.v](systolic_array.v) from left to right as (note topmost is the first row): ![Alt text](imgs/m5.png)
    - B: Streamed to ```col_data_in``` [systolic_array.v](systolic_array.v) from top to bottom as (note leftmost is the first column): ![Alt text](imgs/m6.png)
    - C: Expected to be streamed out from ```row_data_out``` [systolic_array.v](systolic_array.v) from left to right as (note topmost is the first row): ![Alt text](imgs/m7.png)

Test script, a `NUM` number of random tests of A (`M`x`K`) * B (`K`x`N`) matrix multiplications will be performed, on systolic array with size `M`x`N`. You can run the test script by:

```
make systolic_array ROWS=${M} COLS=${N} K=${K} NUM_TESTS=${NUM}
cat results.log | grep PASSED 
```


### Grading Details

During the actual grading for Part 1, we will use 
- `NUM >= 1000`
- `N = K`

Detailed parameters will not be disclosed.


Grading command: `chmod +x run.sh && ./run.sh`

You design should be robust and the grading rubrics as follows:
- Test different `SEED` (**20pts**)
- Test different `MULT_LAT` (Multiplier Latency) (**20pts**)
- Test different `M` and `N` (**20pts**)
- Test different `NUM` (**20pts**)
- Test combined parameter sweep (**20pts**)


## Bonus: General-sized Systolic Arrays

In the previous section, we worked with systolic arrays with size restricted to `N`=`K`. Now, let's explore what happens when the array dimensions `N`=`K` are not equal. Try experimenting with different parameters in the systolic array implementation (you may also adjust the number of trials in the testing script). You'll likely observe that the systolic array fails certain test cases.

**Bonus 1 (5pts)**: Debug General-sized Array Failures

Analyze the waveforms and code of the failing cases. Write a brief explanation (250 words maximum) identifying why the implementation fails when working with non-square systolic arrays.

**Bonus 2 (15pts)**: Support General-sized Arrays

Based on your analysis from Bonus 1, modify the original code to support general-sized systolic arrays, i.e., cases of `N`!=`K`.  You may need modify both the entire implementation. Your implementation must:

- Maintain the same input and output stream interface as the previous section
- Avoid using large multiplexers (scaled to array dimensions) for input or output selection


**For cases where `N` < `K`, we will use a modified testing script to squeeze the bubbles within your results**
- Why? (First read the last hint below) In these cases, a single MAC's own results won't be immediately ready after it bypass results from the previous MACs. Therefore, MACs will output zero in between.
- What if `N` > `K`? 


**Hints**
- Consider the timing from a single MAC unit's perspective:
    - When will results be ready for different array dimensions and different sizes?
    - What happens if a MAC's result is ready, but it needs to wait for results from MACs to its right?

- Think about maintaining output stream order
    - Could the synchronous buffer (from the previous lab; provided here) be useful here?

- **You can adjust the buffer size of the synchronous buffer according to the actual difference between `N` and `K` or simply use a large buffer size, e.g., 128.**
    - Why?: What if the buffer is full or empty? We should not be reading or writing to the buffer, which is like some stall of the computation. However, at current lab, we have not included the implementation to propagate the stall through the systolic array. The next lab will cover this part.


Grading command (your bonus code should still work with the original grading command): 
- `chmod +x run.sh && ./run.sh`
- `chmod +x run_bonus.sh && ./run_bonus.sh`

## Submission

+ Provide a zip file containing your source code. Generate the submission.zip file using the command `make submit`. Avoid manual zip file creation to prevent any issues with the autograding script, which could lead to a 30% score deduction.
* Late submission policy: 3 hours grace period  (Gradescope will be setup soon.)



## Rules and Potential Violations 
1. adder and multiplier instances have to be used and used as they are
2. systolic array IOs **and** default parameters cannot be changed
3. for MAC's parameters, as long as we can pass the MULT_LAT parameter in from the top module (systolic_array.v), it's fine
4. Any changes to Makefile, test*.cpp, adder and multiplier module definition will not be considered


## FAQ 
[Q] Useful lecture links for systolic array design?

https://www.youtube.com/watch?v=cmy7LBaWuZ8&pp=ygUPc3lzdG9saWMgYXJhYXkg 

