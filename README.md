# solidity-gas-optimizations

Note:-

- Use assembly only if you understand what you are doing as it can lead to vulnerabilities if implemented wrontly.
- Never compromise on security for gas savings. Security should always be prioritized than gas optimizations.
- There might be a few repetitions. I will remove them when I find them.

# solidity-gas-optimizations

1. Ternary operation is cheaper than if-else statement. But in some latest solidity versions this is optimized by default.
2. Instead of multiplication or division by powers of 2 use bit shift left or bit shift right operators. Egs:- Shift left by 5 instead of multiplying by 32. Just make sure there is no possibility of overflow/underflow as bit shifting operators are not protected for overflow/underflow even if we use above 0.8.0 solidity versoin. bit shifting operators are not arithmetic operators so it is not protected for overflow/underflow.
3. Whenever possible, instead of (> or <) use (>= or =<). Example: instead of if(x>10) use if(x>=11)
4. Instead of using if(x==true) or if(x==false) use if(x) or if(!x).
5. ++i costs less gas than i++.
6. i += j or i -= j Costs More Gas Than i = i + j or i = i - j For State Variables.
7. Use an unchecked block when operands can’t underflow/overflow.
8. Pack structs by putting data types that can fit together next to each other. If the variables are accessed together in functions this will save gas as only one sload is required instead of having multiple sloads. First variable will be a sload cold further variable access will be a sload warm access.
9. Use calldata instead of memory for function arguments that do not get mutated.
10. If one function calls another function and if same storage is read on both and with no changes to the storage in the transaction, then we can pass the storage from one function to another using as a calldata parameter and use it there. It is useful cases where the called function is a internal or private function.
11. Use assembly code to check for address(0). Use a helper assembly function to do it. You can use your custom error message. 0xd92e233d00000000000000000000000000000000000000000000000000000000 --> selector for `ZeroAddress()`.
    
    ```
    error ZeroAddress();
    
    ```
    
    function assembly_notZero(address toCheck) public pure returns(bool success) { assembly { if iszero(toCheck) { let ptr := mload(0x40) mstore(ptr, 0xd92e233d00000000000000000000000000000000000000000000000000000000)
    
    revert(ptr, 0x4) } } return true; }
    
12. Remove unused variables.
13. Do not pre-declare variable with default values. Don't explicitly initialize default variables. Egs: don't do uint x=0.
14. Use custom errors instead of error strings .Also Make sure the error size doesnot exceed 32 bytes.
15. use !=0 instead of > 0 as validation for unsigned integers. This works as unsigned integers cannot take negative values.
16. use external instead of public for functions that are not called internally
17. Caching the array length outside a loop saves reading it on each iteration, as long as the array’s length is not changed during the loop.
18. for multiple reads , cache storage data in functions to use memory instead of reading from storage multiple times.
19. Upgrade pragma version to atleast 0.8.4. (0.8.19 can be used for more gas savings).
20. Using both named returns and a return statement isn’t necessary. Removing unused named return variables can reduce gas usage and improve code clarity.
21. Redundant variable assignments should be avoided.
22. Avoid repeated addition, instead use multiplication. it happens in loops soemtimes
23. The result of a function call should be cached rather than re-calling the function multiple times from a state changing function.
24. Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead as they have to be masked and casted to read values. It can help in case of packings in structs and reading the packed variables together in few sloads. But if they are alone then they cost more gas. When using elements that are smaller than 32 bytes, your contracts gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size. Use a larger size then downcast where needed.
25. Use the existing Local variable/global variable when equal to a state variable to avoid reading from state.
26. Don't emit storage values instead of the memory ones.
27. Validation checks that involve constants should come before checks that involve state variables, function calls, and calculations.
28. While emitting use msg.sender instead of the variable containing that address, if the msg.sender validation is already done in the function which will guarantee to revert if wrong.
29. Do not calculate constants. Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.
30. If statements that use && can be refactored into nested if statements.
31. Caching a variable that is used just once wastes Gas as there will be unnecessary extra memory storage and read along with sload.
32. Importing an entire library/contract while only using a few function should be avoided. Import only the required parts. Or use interface to only get the required functions.
33. Break out of the loop if the required conditions are met.
34. Use immutable or constant for values that never change after deployment. State variables only set in the constructor and never changes should be declared immutable.
35. When one function calls another function, using the same modifier in both the functions is might be redundant in some cases where the called function is a internal or private function.
36. Some times we check for a condition that will never occur due to other restrictions. So we can remove that redundant check to save gas.
37. Use assembly in place of `abi.decode` to extract calldata values more efficiently as we can just extract the required amount of variables.
38. If we only require a part of a single return variable (bytes type) from a function then we can use Bytes.trimToSize() functionality to trim and get just the required size of data.
39. Constructors can be declared `payable` to save gas. Declaring it payable will reduce some checks that checks if msg.value==0 or not which costs gas.
40. If there are multiple redundant loops with same iteration doing different operations then we can reduce it to one single loop.
41. While extending on top of other contracts like governor contracts, if we are only using certain parts we need to check which states are unreachable or reachable and do checks only for reachable states.
42. Caching a mapping/array value in a storage pointer when the value is accessed multiple times saves ~40 gas per access due to not having to perform the same offset calculation every time.
43. If one function calls another function and if same storage is read on both and with no changes to the storage in the transaction, then we can pass the storage from one function to another using as a calldata parameter and use it there.
44. A do while loop will cost less gas since the condition is not being checked for the first iteration. But its not recommended to use it as there are chances that it might be implemented wrongly.
45. Refactor event to avoid emitting data that is already present in transaction data.
46. Avoid emitting constants.
47. If a variable is declared as one type then no need to re cast it again to the same type. It happens by mistake sometimes.
48. Avoid Unnecessary multiplication and division by 1e18. Instead, just make sure all multiplications are done before division, that will prevent rounding in the same manner.
49. Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.
50. Use solidity version 0.8.19 to gain some gas boost. Reference -> [https://soliditylang.org/blog/2023/02/22/solidity-0.8.19-release-announcement/](https://soliditylang.org/blog/2023/02/22/solidity-0.8.19-release-announcement/)
51. Use assembly for simple setters. (Example:- to write address storage values in functions and constructors instead of normal assignment.) function changeRevenueAddress(address _newRevenueAddress) external onlyOwner { assembly {
    
    sstore(revenueAddress.slot, _revenueAddress) }
    
    }
    
52. Multiple Address Mappings (or id mappings) Can Be Combined Into A Single Mapping Of An Address (or id) To A Struct, Where Appropriate.
53. bytes constants are more eficient than string constants. If the data can fit in 32 bytes, the bytes32 data type can be used instead of bytes or strings. Cast it to string when required.
54. Use ERC721A instead ERC721. Reference -> [https://nextrope.com/erc721-vs-erc721a-2/](https://nextrope.com/erc721-vs-erc721a-2/)
55. Sort Solidity operations using short-circuit mode //f(x) is a low gas cost operation //g(y) is a high gas cost operation
    
    //Sort operations with different gas costs as follows f(x) || g(y) f(x) && g(y)
    
56. Make sure Solidity’s optimizer is enabled. It reduces gas costs. If you want to gas optimize for contract deployment (costs less to deploy a contract) then set the Solidity optimizer at a low number. If you want to optimize for run-time gas costs (when functions are called on a contract) then set the optimizer to a high number.
57. Using delete instead of setting struct to 0 values saves gas.
58. Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. We can precalculate the address easily. References: [https://book.getfoundry.sh/reference/forge-std/compute-create-address](https://book.getfoundry.sh/reference/forge-std/compute-create-address)
59. Internal functions only called once can be inlined to save gas
60. Shorten the array rather than copying to a new one. Inline-assembly can be used to shorten the array by changing the length slot, so that the entries don’t have to be copied to a new, shorter array.
61. Checks, effects, interactions is a general best practice and can be applicable to more than just reentrancy concerns. When one of the following error scenarios applies, users pay gas for all statements executed up until the revert itself. By performing checks such as these as early as possible, you are saving users gas in failure scenarios without any sacrifice to the happy case costs.
62. Revert on no-op. Unfortunately it can be common for users to accidentally fire the same transaction multiple times. This can result from an unclear app experience, or users misunderstanding speed up / replace transaction. When this occurs the duplicate transaction should be a no-op, ideally reverting as early as possible to limit gas costs. Reverting in the case of a duplicate may also prevent the redundant transaction from being broadcasted at all since apps and wallets typically estimate gas before broadcasting and that would show that it’s expected to revert if the original has already been mined.
63. If a redundant block of code is used by multiple functions. Move the code to a private helper function and call this function from those functions which require this block of code.
64. Write element of storage struct to memory when used more than once. Basically cache it.
65. Division cannot underflow unless its divided by negative number. So we can use unchecked.
66. Make 3 event parameters indexed when possible
67. Avoid redundant transfers.
68. Using separate internal functions for non-repeating if blocks in more than one function wastes gas. Instead use the if block in the required function directly.
69. Private state variables cost lesser gas than public state variables.
70. While assigning struct values use the efficient way. (refer solodit timeswap contest (21/01/2023))
71. Check for Gas overflow during iteration (DoS attacks)
72. Move owner checks to a modifier for gas efficiency
73. Using UniswapV3 mulDiv function is gas-optimized.
74. Using Openzeppelin Ownable2Step.sol is gas efficient for ownership accept,revoke and transfer functions.
75. OpenZeppelin’s ReentrancyGuard contract is gas-optimized. Use it instead of using your custom mutex.
76. Remove development based imports that are only used for dev and tests before deployment. Egs: forge-std
77. Using XOR (^) and OR (|) bitwise equivalents (a != b || c != d || e != f) costs 571 ((a ^ b) | (c ^ d) | (e ^ f)) != 0 costs 498 (saving 73 gas).
78. Using a positive conditional flow to save a NOT opcode. Reference -> (solodit 14/01/2023 OpenSea Seaport 1.2 contest).
79. The optimized equivalent of or(eq(a, 2), eq(a, 3)) is and(lt(a, 4),gt(a, 1)) (saving 3 gas).
80. Internal/Private functions only called once can be inlined to save gas
81. If a condition for msg.sender is checked before executing further lines, then we can use msg.sender instead of the storage/memory variable which stored the address. if the msg.sender validation is already done in the function which will guarantee to revert if wrong.
82. Caching global variables is more expensive than using the actual variable(use msg.sender instead of caching it)
83. A modifier or public/internal function used only once and not being inherited should be inlined to save gas
84. When emitting an event that includes a new and an old value, it is cheaper in gas to avoid caching the old value in memory. Instead, emit the event, then save the new value in storage.
85. Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas.
86. Generate merkle tree offchain. Generate merkle tree onchain is expensive espically when you want to include a large set of value. Consider generating it offchain a publish the root when creating something.
87. Superfluous event fields. block.number and block.timestamp are added to the event information by default, so adding them manually will waste additional gas.
88. Expressions for constant values such as a call to keccak256(), should use immutable rather than constant.
89. Before transfer, we should check for amount being 0 so the function doesn't run when its not gonna do anything.
90. Instead of calculating a statevar with keccak256() every time the contract is made pre calculate them before and only give the result to a constant.
91. Remove `public` visibility from `constant` variables. constant variables are custom to the contract and won't need to be read on-chain - anyone can just see their values from the source code and, if needed, hardcode them into other contracts. Removing the public visibility will optimise deployment cost since no automatically generated getters will exist in the bytecode of the contract.
92. Use ERC721 instead of ERC721Enumerable when the contract already manages the nft state in another way.
93. Unnecessary redundant reinitializations can be avoided. Egs:- Initialize local variable outside of loop to save gas.
94. Where a variable is defined, it is important to avoid unnecessary expense prior to the appropriate validations.
95. Use `constant` instead of `immutable` for predetermined values. Also make sure they are private, to save gas.
96. Unless you are doing variable-packing in storage it is best to use always use uint256 because all other uint types get automatically padded to uint256 in memory anyway due to memory expansion. Same applies for function parameters
97. Use uint256(1) and uint256(2) for true/false instead of bool to save gas.
98. abi.encode() is less efficient than abi.encodePacked(). use abi.encodePacked() where possible, to save gas. But note that encode packed might lead to collisions.
99. Use selfbalance() instead of address(this).balance when getting your contract’s balance of ETH to save gas. function YULF() public view returns (uint256) { uint256 self; assembly { self :=selfbalance() }
    
    return self; }
    
100. Checking msg.sender to not be zero address is redundant. This check is redundant as no private key is known for this address, hence there can be no transactions coming from the zero address.
101. keccak256() should only need to be called on a specific string literal once It should be saved to an immutable variable, and the variable used instead. If the hash is being used as a part of a function selector, the cast to bytes4 should also only be done once
102. Use function instead of modifiers if there is significant gas savings. But note that modifiers is best pratise
103. Use named returns where appropriate
104. Use elementary types or a user-defined type instead of a struct that has only one member.
105. Deleting a struct is cheaper than creating a new struct with null values
106. Multiple accesses of a mapping/array should use a local variable cache
107. Access mappings directly rather than using redundant accessor functions
108. Remove unreachable code
109. Constructor parameters should be avoided when possible. Constructor parameters are expensive. The contract deployment will be cheaper in gas if they are hard coded instead of using constructor parameters.
110. block.timestamp and block.number are added to event information by default, explicitly adding them is a waste of gas.
111. Use assembly to hash instead of Solidity
112. Use assembly for math (add, sub, mul, div)
113. Don’t use SafeMath once the solidity version is 0.8.0 or greater. Version 0.8.0 introduces internal overflow checks, so using SafeMath is redundant and adds overhead.
114. Multiple if-statements with mutually-exclusive conditions should be changed to if-else statements. Also If two conditions are the same, their blocks should be combined.
115. Update to state var should ideally happen only once in a function. Optimize the calculations. But there are cases where multiple updates are necessary where we cannot do much.
116. OR conditions cost less than their equivalent AND conditions (“NOT(something is false)” costs less than “everything is true”) Remember that the equivalent of (a && b) is !(!a || !b) Even with the 10k Optimizer enabled: OR conditions cost less than their equivalent AND conditions.
117. It is cheaper to split struct if only part of it is updated frequently
118. xpressions for constant values such as a call to keccak256(), should use immutable rather than constant as using constants recalculate each time its used.
119. Use msg.sender instead of OpenZeppelin’s _msgSender() when meta-transactions capabilities aren’t used msg.sender costs 2 gas (CALLER opcode). _msgSender() represents the following: function _msgSender() internal view virtual returns (address payable) { return msg.sender; }
120. Using 1 and 2 rather than 0 and 1 saves gas for isLocked isNotLocked type of checks due to non-zero to non-zero interaction both the ways.
121. use type(uint256).max instead of 2**256 - 1
122. use 10e18 instead of 10**18
123. If function identifier contains zero bytes then it will cost less gas. calldata function identifier:- 1 byte - 16 gas if there is a non zero value calldata function identifier:- 1 byte - 4 gas is the byte is a zero value
124. Transaction costs go down if there is a lot of zeros in our ethereum address. vanity address generation -> researching more.
