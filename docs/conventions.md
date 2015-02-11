###Legends

* ->> : represents message to multiple non-NAE
* -> : represents message to one non-NAE
* | : sentinle
* [*Operation1*, *Operation2*, .. *OperationN*] : denotes sequential operations
* {*Operation1*, *Operation2*, .. *OperationN*} : denotes parallel operations
* (*Operation1*)(*Operation2*) .. (*OperationN*) : denotes sequential dependent operations, for instance, *Operation2* requires results from *Operation1* and produces results for *Operation3*, $ may be used to represent the result from previous operation.
* condition ? A : B : C/C++ syntax
* condition ? A : represents _condition ? A : NoOp_
* ! denotes NOT
* Manager < Address > : group managing Address
* PLoop x in xs : do : represents a parallel loop 
