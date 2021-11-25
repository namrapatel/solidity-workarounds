# Workarounds

Note: This is all hacky documentation, examples are made up and may not even throw the error that is described in the issue, but it should paint the picture nonetheless. Make an issue or PR in this repo if something here is confusing. 

## Stack Too Deep EVM Exception
If you have too many local variables or params in a function (I believe the limit is 16), sometimes the EVM will complain, "Stack too deep". When this happens, you can either be a giga-chad and look at the EVM bytecode do see whats happening with your function, or try the following solutions: 

### Issue example

``` Solidity
function stackTooDeepExample(
    uint256 one, 
    uint256 two) external {

        uint256 three = 3;
        uint256 four = 4;
        uint256 five = 5;
        uint256 six = 6;
        uint256 seven = 7;
        uint256 eight = 8;
        uint256 nine = 9;
        uint256 ten = 10;

        uint256 store1 = someLib.computeSomething(one, two, three, four, five, six, seven, eight, nine, ten); 

        // error would be thrown at the following line, saying the stack is too deep and some local variables should be deleted.

        uint256 store2 = someLib.computeSomething(two, one, three, four, five, six, eight, seven, nine, ten); 
}
```

### Solution 1
Make an array to store similarly typed local variables because arrays hold a the exact size of memory you allocate them to, so no stack space is unused.

``` Solidity
function stackTooDeepSolution1(
    uint256 one, 
    uint256 two) external {
        uint256[] localVars = new uint256[](4);

        localVars[0] = 3; 
        localVars[1] = 4;
        localVars[2] = 5;
        localVars[3] = 6;
        uint32 seven = 7;
        uint32 eight = 8;
        uint32 nine = 9;
        uint32 ten = 10;

        uint256 store1 = someLib.computeSomething(one, two, localVars[0], localVars[1], localVars[2], localVars[3], seven, eight, nine, ten);
        
        uint256 store2 = someLib.computeSomething(two, one, localVars[0], localVars[1], localVars[2], localVars[3], eight, seven, nine, ten); 
}
```

The error should have gone away, if it doesn't add the following solution on top.

### Solution 2
Scoping a piece of code with { ... } has the same impact as placing that piece of code in a separate function. The UniswapV2 contracts use this [here](https://github.com/Uniswap/v2-core/blob/4dd59067c76dea4a0e8e4bfdda41877a6b16dedc/contracts/UniswapV2Pair.sol#L166).

``` Solidity
function stackTooDeepSolution1(
    uint256 one, 
    uint256 two) external {
        uint256[] localVars = new uint256[](4);

        localVars[0] = 3; 
        localVars[1] = 4;
        localVars[2] = 5;
        localVars[3] = 6;
        uint32 seven = 7;
        uint32 eight = 8;
        uint32 nine = 9;
        uint32 ten = 10;

        uint256 store1 = someLib.computeSomething(one, two, localVars[0], localVars[1], localVars[2], localVars[3], seven, eight, nine, ten); 

        {
        uint256 store2 = someLib.computeSomething(two, one, localVars[0], localVars[1], localVars[2], localVars[3], eight, seven, nine, ten);
        } 
}
```