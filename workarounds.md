# Workarounds

Note: This is all hacky documentation, examples are made up and may not even throw the error that is described in the issue, but it should paint the picture nonetheless. If you're deploying to prod, perhaps check all these solutions over. Make an issue or PR in this repo if something here is confusing. 

## Stack Too Deep EVM Exception
If you have too many local variables or params in a function (I believe the limit is 16), sometimes the EVM will complain, "Stack too deep". When this happens, you can either be a giga-chad and look at the EVM bytecode do see whats happening with your function to clean it up, or try the following solutions: 

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

### Solution 3
Use a struct to store the local variables. An advantage of a struct is that it's more readable compared to an array and it allows you to store variables of different types. [Compound](https://github.com/compound-finance/compound-protocol/blob/4a8648ec0364d24c4ecfc7d6cae254f55030d65f/contracts/CToken.sol#L480), [Aave](https://github.com/aave/protocol-v2/blob/c1ada1cb68bb26a39b6afdb299e58291b831f1ec/contracts/protocol/lendingpool/LendingPool.sol#L454) and [Uniswap V3](https://github.com/Uniswap/v3-core/blob/c05a0e2c8c08c460fb4d05cfdda30b3ad8deeaac/contracts/UniswapV3Pool.sol#L617) use this pattern.

``` Solidity
struct StackTooDeepLocalVars {
    uint256 three;
    uint256 four;
    uint256 five;
    uint256 six;
}

function stackTooDeepSolution1(uint256 one, uint256 two) external {
    StackTooDeepLocalVars memory localVars = StackTooDeepLocalVars(3, 4, 5, 6);

    uint32 seven = 7;
    uint32 eight = 8;
    uint32 nine = 9;
    uint32 ten = 10;

    uint256 store1 = someLib.computeSomething(
        one,
        two,
        localVars.three,
        localVars.four,
        localVars.five,
        localVars.six,
        seven,
        eight,
        nine,
        ten
    );

    uint256 store2 = someLib.computeSomething(
        two,
        one,
        localVars.three,
        localVars.four,
        localVars.five,
        localVars.six,
        seven,
        eight,
        nine,
        ten
    );
}
```

## Hack around to reduce contract size due to revert string
Sometimes, the contract size may get unncessarily large because of Error message in your `require` statement. 

### Issue
Lets say you want to print this message :
\
```Solidity
function setA(uint _A) public {
require(msg.sender == owner, "Restrcition error: Only Admin of the contract can set A");
..
}
```
Now, in VS code, you will see tailing yellow sign and when you hover over it, it would show that the Error message is too long. Becuase the error messages occopy space in the EVM.

### Solution
So, a work around to reduce is by using Error Codes instead of a full string. This is what [Aave v2](https://github.com/aave/protocol-v2/blob/master/contracts/protocol/libraries/helpers/Errors.sol) implements to reduce the contract size and compiler wont yell at you for trying to deploy a bulky cocntract.

So, the above can be written as
```Solidity
string const Only_Admin_Can_Set = "1";
function setA(uint _A) public {
require(msg.sender == owner, Only_Admin_Can_Set);
..
}
```


