MPEG-1
======
SIF	Standard or Source Input Format
GOP	Group of pictures is a seris of one or more coded frames

Four types of coded frames:

I (intra) frames
P (predicted) frames
B (bidirectional) frames
D (DC) frames

Macroblock
----------
Consists of a 16-sample x 16-line set of Y components and the same corresponding
two 8-sample x 8-line Cb and Cr components.

- These are specific to MPEG and may mean something else in different contexts
- This description given is also specific to the color subsampling in use, which
  for MPEG-1 and MPEG-2 is always YCbCr 4:2:0 (not co-sited) see Table 3.3 in
  Edition 3.
- Key statement here: note that a Y block refers to one-fourth the image size as
  the corresponding Cb and Cr blocks.  

Several keys to keep in mind when trying to understand macroblocks and blocks.
First, the macroblock dimensions and composition are specific to a particular
color subsampling ratio.  The same 16 x 16 sample macroblock that contains 4
Y blocks and 1 Cr and 1 Cb block *on top of each other* with 4:2:0 color
subsampling would contain a different number of blocks that represented
different information wiht a different color subsampling ratio selected.

Drawing these out in the same way that the textbook does is helpful - the
essential detail is to recognize that the spatial arrangement of samples of
YCbCr values matters very much in determining how many blocks are in a
macroblock, their meaning, and what each block contains. The other important
detail is to keep in mind the requirement that the DCT has of operating upon 8 x
8 arrays of samples.  Really, it's operating on a sequence of state vectors and
the operator is rotating from one set of basis vectors, where the values are the
spatial amplitude of the color subsamples and the other set, where the values
are essentially just the frequencies.

Question: Is the quantizer step scale for the frequency coefficients in the
quantizer matrix what we refer to as the quality?

There are two types of macroblocks in I frames - one, the intra-d, uses the
curren quantizer scale; the other, the intra-q, defines a new value for the
quantizer scale.  If it's intra-q, then the decoder uses it to calculate DCT
coefficients from the transmitted ones (there is a 5-bit quantizer scale
factor).  If it's intra-d, no quantizer scale is sent.

Question: How do you determine intra-d vs. intra-q I macroblocks within
I-frames?

