
# GAS OPTIMIZATIONS

##

## [G] Optimizing gas usage by packing state variables 

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper.

Struct can be packed fewer slots

state variables only assigned within constructor can be immutable 

struct can be used instead of mapping 

calldata can be used instead of memory for external functions 

Use constants when the values are not changed for state variable 

remove unused state variables

bools can be avoided 

 Functions guaranteed to revert when called by normal users can be marked payable 10 210

don't emit state variable when stack varibale available 

Superflows event

combine emits to avoid gas 



Optimize names of public/external functions 



STATE VARIABLES SHOULD BE CACHED IN STACK VARIABLES RATHER THAN RE-READING THEM FROM STORAGE 
Only missing instance 

The result of function calls should be cached rather than re-calling the function 3

Using storage instead of memory for structs/arrays saves gas

Only missing instances 

Multiple accesses of a mapping/array should use a local variable cache

 <x> += <y>/<x> -= <y> costs more gas than <x> = <x> + <y>/<x> = <x> - <y> for state variables  

Avoid contract existence check 


 
