# Solidity Best Practice

Read the [solidity style guide](http://solidity.readthedocs.io/en/latest/style-guide.html) first.

Then [ConsenSys best practice](https://github.com/ConsenSys/smart-contract-best-practices).

Further as follows:

* DO NOT add prefix "_" on the function argument called "self" which postioned first in library. But add it on every other function arguments. And DO NOT add it on return params.
* Add version v1…n for updatable contracts.
* Recommend to ```delete``` obsolete state variables for saving gas.
* Use ```contract.new()``` in every unit test file's ```beforeEach``` area, then write every ```it``` test case.
