# Solidity Best Practice

Read the [solidity style guide](http://solidity.readthedocs.io/en/latest/style-guide.html) first.

Then [ConsenSys best practice](https://github.com/ConsenSys/smart-contract-best-practices).

Further as follows:

* DO NOT add prefix "_" on the function argument called "self" which postioned first in library. But add it on every other function arguments.
* Add version v1…n for updatable contracts.
* Recommend to ```delete``` state variables for saving gas.
* Write test case as long as possible in one ```it``` area, to avoid the potential logic pollution and confusions through multi ```it```s.
* Use ```exception``` first rule: Even in get* functions, throw an exception when the item not found or something like not fitting the query logic, instead of returning false or empty value or error code or something else.
