Temporary Allocation    

[![](../img/logo.png)](index.html)

*   [Home](../index.html)
*   [About](../about.html)
*   [Code](#)
*   [Documentation](../documentation.html)

* * *

  
  

*   [Temporary Allocation](#nav-register-allocation)

Temporary Allocation
--------------------

Registers are allocated and released for the temporary storage of intermediate computation through two simple functions.

*   int get\_register(): Allocates a free register from the register pool (R0 - R19) and returns the index of the register, returns -1 if no free register is available.
*   int free\_register(): Frees the last register that was allocated,returns 0 if success, returns -1 if the function is called with none of the registers being allocated.

*            [Github](https://github.com/silcnitc)
*   [![Creative Commons License](../img/creativecommons.png)](http://creativecommons.org/licenses/by-nc/4.0/)

[Go up](#navtop)

*   [Home](../index.html)
*   [About](../about.html)