###Legend

* `->>` represents sending a message to multiple non-NAE
* `->` represents sending a message to one non-NAE
* `|` represents the sentinel accumulating incoming messages from a group
* `{Operation1, Operation2, .. OperationN}` denotes sequential operations
requires results from `Operation1` and produces results for `Operation3`. `$` may be used to represent the result from previous operation.
* `condition ? A : B` C/C++ conditional operator syntax; if `condition` is true, then `A` else `B`
* `condition ? A` represents `condition ? A : no-op`, i.e. if `condition` is false, nothing happens
* `!` denotes NOT
* `ManagersOf < Address >` represents the group managing `Address`
* `Loop x in xs : do` represents a loop
