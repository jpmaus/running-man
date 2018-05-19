# running-man
Generates MIDI Pitch Events From Integer Sequences
## Info
The CDF version of running-man should be runnable using this free [CDF Player](https://www.wolfram.com/cdf-player/).  To edit the running-man notebook, you'll need [Mathematica](http://www.wolfram.com/mathematica/).
##### under the hood, for begginers
### 1. Pitch Class Integer Notation
running-man uses [integer notation](https://en.wikipedia.org/wiki/Pitch_class#Integer_notation) to represent pitch classes.  Integer notation is a translation of pitch classes into natural numbers.  For example, if C = 0, then C# = 1, D = 2, D# = 3, and so on.  By extension, integer notation can also represent harmonies and scales:

| Hamony / Scale      | Note Names                   | Pitch Classes          |
| :---:      | :---:                   | :---:                  |
| C Maj      | (C, E, G)               | {0, 4, 7}              |
| G Min / D  | (D, G, Bb)              | {7, 10, 2}             |
| F Maj / C  | (A, C, F)               | {9, 0, 5}              |
| C Dorian   | (C, D, Eb, F, G, A, Bb) | {0, 2, 3, 5, 7, 9, 10} |
| D Lydian   | (D, E, F#, G#, A, B, C#)| {2, 4, 6, 8, 8, 10, 1} |
| C Shang    | (C, D, F, G, C, Bb)      | {0, 2, 5, 7, 10}   |

### 2. Frequently Used Functions
#### 2.1 Inbuilt Wolfram Language Functions
running-man frequently uses the following basic Wolfram Language functions.  For more detailed explanations of these functions, visit the [Wolfram Language Reference Site](http://reference.wolfram.com/language/).

- **Range[**_n_**]** generates the list {1,2,...,n}.
- **Table[**_f_, _n_**]** generates the list of values _n_ of function _f_.
- **With[**{_a_=_x_, _b_=_y_, ...}, _exp_**]** specifies that all occurrences of _a_, _b_, ... in _exp_ should be replaced by _x_, _y_, ... 
- **Take[**_lst_, _n_**]** gives the first _n_ elements of _list_.
- **Mod[**_m_, _n_, _o_**]** gives the remainder on division of _m_ by _n_ using an offset _o_. 
- **Map[**_f_, _lst_ **]** or _f_**/@**_exp_ applies function _f_ to each element in list _lst_. 
- **Flatten[**_lst_**]** flattens the nested list _lst_. 
- **Differences[**_lst_**]** gives the differences between each successive element in list _lst_.
- **LinearRecurrence[**_ker_, _int_ ,_n_**]** gives the sequence of length _n_ obtained by iterating the linear recurrence with kernel _ker_ starting with initial values _int_.

#### 2.2 Integer Sequence Functions
The Wolfram Language provides an extensive selection of inbuilt integer sequence functions.  For example, the fourth integer in the Fibonacci sequence (i.e., 1, 1, 2, 3, 4, ...) is given by simply entering:  

```mathematica
In[]:= Fibonoacci[4]
Out[]:= 3
```
running-man uses functions such as **Table[]** and **Range[]** to iterate these sequence functions:

```mathematica
In[]:= Fibonacci[Range[12]]
Out[]:= {1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144}
```
```mathematica
In[]:= Table[Fibonacci[x], {x, 1, 12}]
Out[]:= {1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144}
```
[The Online Encyclopedia of Interger Sequences](https://oeis.org/) provides formulae for integer sequences that are not inbuilt into the Wolfram Language.  Some OEIS entries also provide these formulae using inbuilt Wolfram Language functions.  For example, the OEIS entry for the square numbers [OEIS A000290](https://oeis.org/A000290) includes the following code for generating the square numbers using the Wolfram Language **LinearRecurrence[]** function: 

```mathematica
In[]:= LinearRecurrence[{3, -3, 1}, {0, 1, 4}, 12]
Out[]:= 0,1,4,9,16,25,36,49,64,81,100,121,144
```
#### 2.3 Moving Window Function
running-man uses a moving window function:

```mathematica
MovingWindow[f_, exp_, n_, o_] := 
  Module[{len = Length[exp], end}, end = Min[n, len] - 1;
   Table[Apply[f, {exp[[i ;; i + end]]}], {i, 1, len - end, o}]];
```
###### c.f., _Mathematica Cookbook_, Sal Mangano: 44

The **MovingWindow[**_f_, _lst_, _n_, _o_**]** function moves a window over _lst_ of function _f_ by groups of _n_ at an offset of _o_.  For example, using **MovingWindow[]** with function **Take[**_list_,_n_**]** and offset and group both set to 3 gives:  
```mathematica
In[]:= MovingWindow[Take, Fibonacci[Range[9]], 3, 3]
Out[]:={{1, 1, 2}, {3, 5, 8}, {13, 21, 34}}
```
An offset of 1 gives:
```mathematica
In[]:= MovingWindow[Take, Fibonacci[Range[9]], 3, 1]
Out[]:= {{1, 1, 2}, {1, 2, 3}, {2, 3, 5}, {3, 5, 8}}
```
An offset of 3 and a group 1 gives:
```mathematica
In[]:=MovingWindow[Take, Fibonacci[Range[9]], 1, 3]
Out[]:={{1}, {3}, {13}}
```

### 3. MIDI Event Generating Functions
#### 3.1 The Mono Function
running-man uses the **Mono[]** function to generate a list of MIDI pitch events from a preselected mode:
```mathematica
Mono[exp_] := 
  With[{arb = exp}, Take[arb, {#}] & /@ Mod[d@F[ln], mB, mI]];
```
###### c.f., _Programming Avro Part_, [link](https://aestheticcomplexity.wordpress.com/2011/11/11/programming-arvo-part/)
##### 3.1.1 Variables of the Mono Function 
The variables of the **Mono[]** function are nested in Mathematica's **Manipulate[]** template, because of this fact they are assigned by way of the **Manipulate[]** template's modest GUI.  However, when the running-man notebook is first initialized, the variables of the **Mono[]** function (e.g., _d_, _F_, _len_, ...) are automatically assigned the following values:  

|             Variable                    | Assignment |
| :---: | :--- |
|mB = 7| The Modular Divisor|
|mI = 1| The Modular Offset |
|d = Flatten| If d is set to Flatten, pitches are given from successive cardinalities. <br /> If d = Differences, pitches are given from successive differences.|
|F = Fibonacci[Range[len]]| The integer sequence function used to generate pitches with the <br /> Range function nested such that it generates a list instead of single integer.| 
| len = 12 | The length of the pitch set to be generated |
|dor = {0, 2, 3, 5, 7, 9, 10, 0, 2, 3, 5, 7, 9, 10} | The Integer Notation of the Dorian mode | 

```mathematica
In[]:= Mono[dor]
Out[]:= {0, 0, 2, 3, 7, 0, 9, 10, 9, 9, 7, 5}
```
Change a variable to generate pitches from differences rather than from cardinalities:
```mathematica
In[]:= d = Differences;
In[]:= Mono[dor]
Out[]:= {10, 0, 0, 2, 3, 7, 0, 9, 10, 9, 9}
```
Following the Fibonacci sequence (1, 1, 2, 3, 5, ...).  The 1st element in _dor_ is 0, the 2nd element is 0, the 3rd element is 2, the 4th element is 3, the fifth element is 7, etc.

|             Function                   | 1     | 2     | 3     | 4     | 5     | 6     | 7     | 8     | 9     | 10    | 11    | 12    |
| :---:                              | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | 
|dor PCs                             | 0     | 2     | 3     | 5     | 7     | 9     | 10    | 0     | 2     | 3     | 5     | 7     |
|F<sub>*n*+1</sub> mod 7             | 1     | 1     | 2     | 3     | 5     | 1     | 6     | 7     | 6     | 6     | 5     | 4     |
|F<sub>*n*+1</sub> mod 7 /@ dor PCs  | 0     | 0     | 2 | 3 | 7 | 0 | 9 | 10 | 9 | 9 | 7 | 5 |
|Δ F<sub>*n*+1</sub> mod 7 /@ dor PCs|N/A    |10     | 0 | 0 | 2 | 3 | 7 | 0 | 9 | 10 | 9 | 9|


#### 3.1 The Harmonize Function
The **Harmonize[]** function gives a nested sublist (i.e., aggregate) of any three consecutive PCs that constitute a major or a minor triad. 
```mathematica
Harmonize[exp_] := 
 With[{n1 = Take[exp, {1}], n2 = Take[exp, {2}], 
   n3 = Take[exp, {3}]},
  If[n2 == (n1 + 0) && n3 == (n1 + 0) || 
    n2 == (n1 + 0) && n3 == (n1 + 3) || 
    n2 == (n1 + 0) && n3 == (n1 + 7) || 
    etc. etc.
```
#### 3.2 The Poly Function
The **Poly[]** function runs a **MovingWindow[]** of **Harmonize[]** over the lists given by **Mono[]**.  

```mathematica
Poly[exp_] := MovingWindow[Harmonize, Mono[m], 3, 3];
```
In this way, the **Poly[]** function allows the generation of diatonic, polyphonic, MIDI files from integer sequences.
## Future Work
running-man is pretty simple code.  I tried messing around with serializing the MIDI event durations (the Y and N buttons on the GUI), but I never figured out a procedure that produced appropriate sounding results.  I also tried expanding the **Harmonize[]** function to include four-voice (e.g., 6ths, 7ths), and even five-voice (e.g., 9ths) harmonies, but that only ended up making the whole program too unmanageable.  Please mess around with the code and share!  You'll notice that upon initialization (BASE M = 11), the MIDI events are the same as the verse for The Running Man.  The structure of The Running Man, from section to section, is simply an adjustment of the BASE M parameter (part a = 2, part b = 4, part c = 5, etc.).      
