# Solidity notes in short

### Components

1.  Interfaces :
    
    - functions declared in interfaces are implicitly 'virtual'
    - interfaces should be correctly implemented (return values) wrt EIPs.
2.  Library :
    
    - libraries are deployed and its code is called using delegatecall
    - `using lib for X`, `import lib;  lib.fnCall()`
3.  state vars
    
    - getter function automatically generated for public state vars
    - private vars : note that everything that is inside a contract is visible to all observers external to the blockchain.
    - accessing varialbes inside contract and outside contract has suble difference. outside with getters, inside is directly.
4.  functions : members are .selector
    
    - signature : transfer(address,uint256). return value is not part of signature, hence also not the selector
    - selector : bytes4 ( hash ( signature ) ),   `contract.function.selector`
    - auto decodes call/delegatecall 'fnshipment'
    - return variables can be used as any other local variable.
    - mappings cannot be input output for functions.
5.  `constructor(args) base(args) [payable] {}`
    
6.  receive
    
    - `receive() external payable {}`
    - call to contract with ether value and empty calldata
    - no fn sig, no fn selector, can't have args, can't return anything, must be external, must be payable
7.  fallback
    
    - `fallback() external [payable] {}`
    - `fallback(bytes calldata input) external [payable] returns(bytes memory) {}`
    - When you 'call' to contract where the function signature does not exist.
    - abi.decode to extract payload from the 'shipment'
    - no fn sig, no fn selector
    - note : best for sending arbitrary data package to the contract
8.  modifier
    
    - does not accept arguments
    - `_;` represents function body, can be included multiple times.
9.  Errors :
    
    - `require(condition,"msg")` : reverts with msg
        
    - `error errName(parms);` ,  `revert errName(params);`
        
    - require vs revert : throwing of errors
        
10. Events
    
    - Event logs are stored on blockchain but not accessible from within contracts not even from the contract that created them
    - 'indexed' arguments make it easy for searching and filtering through the logs

### Types : declaration, location, initialization, access, conversion, manupulation, efficiency

<ins>Value types passed by creating a new copy</ins>

1.  bool : true, false (small letters)
    
2.  uint : internal processing on raw data to function like human integers to base 10.
    
    - division is rounding to floor
        
    - underscores neglected by compiler
        
    - 1eX is exponentation to base 10
        
    - can't be converted to bytes, first need to convert to bytes32
        
3.  address
    
    - msg.sender is of type address
    - implicit to normal, explicit to payable
    - Explicit conversions to and from address are allowed for uint160, bytes20 and contract types.
    - Only 'address' and 'contract'(if it can receive Ether) can be converted to 'address payable'.
4.  Contracts :
    
    - can be explicitly converted to and from the *address* type.
        
    - functions can take in contract type arguments
        
5.  fixed size bytesN : bytes4, bytes32 <ins>(except bool and integer)</ins>
    
6.  Enums
    
    - like variables with max 256 different values. \[0,1, . . , 255\]
        
    - They can be explicitly converted to/from integers starting from zero.
        
7.  function types
    
8.  custom types
    
    - type C is V
    - has no members no conversion nothing, everyting to be manual
    - C.wrap( ) and C.unwrap( )

<ins>Reference types are different names for same variable</ins> (stored in storage)

1.  Structs
    
    - initialization of struct is json like or object like setting one by one property .
    - variables are tightly packed, so the order of variables matter.
2.  Array structure
    
    - string : UTF-8 char, no length or index access. members are string.concat(s1,s2, . . )
    - bytes : raw literal(except bool and integer) imagine like dynamic string, with index access.  bytes.concat(b1,b2, . . )
    - custom array : fixed size T\[n\] , dynamic size T\[ \] and members are length, push(x), pop( ) (cant get entire array from getter fn)
        - memory arrays are fixed size only.  `T[] memory a = new T[](n);`
        - array slices for calldata arrays only : payload\[4:\]
    - Solidity automatically checks for out-of-bounds index access and will revert.
3.  Mappings : can be only in storage
    
    - key needs to be simple - value type, bytes, string
    - value can be any type

<ins>location</ins> : storage, memory, calldata

<ins>Types: Conversion</ins>

- uint256 to bytes : uint256 - bytes32 - bytes
- uint256 to string : not supported
- string to bytes32 : string - bytes - bytes32

### Globals

1.  address : balance, <ins>call, delegatecall</ins>
    
    - `(bool success,bytes memory data)= address.call{value:.., gas:..}(bytes memory fn/shipment);`
        
    - `(bool success, bytes memory data) = address.delegatecall{gas:..}(bytes memory fn/shipment);`
        
    - when delegatecalling, state var slots placement should be preserved.
        
    - fnshipment : abi.encodeWithSignature(signature, args, . . )
        
    - payload : packaged data ,  shpment : abi.encode(payload) ,  abi.decode(shipment, (types))
        
    - recieving contract auto decodes payload and identifies the function called.
        
    - forwards all of the gas provided by user(tx initiator), 'value' modifier only for call.
        
    - only address object can do this not contract object.
        
    - returns 'true' even if the address called is non-existent, account existence should be checked prior to call/delegatecall
        
    - need to manually check for success
        
2.  abi encoding the data package
    
    - abi.encode(args, . .) : args are padded, no risk of hash collision
        
    - abi.encodeWithSignature(signature, args, . . . )
        
    - abi.decode<ins>(</ins>data,(types, . . ) )
        
3.  (bytes32 hash) = keccak256(bytes memory data)
    
4.  address signer = ecrecover( hash, v, r, s )
    
5.  address payable : balance, call, delegatecall, <ins>transfer, send</ins>
    
    - `address.transfer(amount);`  : reverts on failure
    - `address.send(amount);`  : returns (bool)
    - both forward 2,300 gas, not adjustable.
6.  controls
    
    - inheritance : fn behaviour may become more strict, bases are searched from right to left depth-first.
        - constructor execution order : B1, B2, D1
        - virtual, override, this, super: function up in fn overriding
        - private - only contract, internal - inheritance tree, public - everyone, external - only outsiders
    - behaviour : view, pure
    - if, else if, else, for, while, continue, break
    - unchecked { } : arithmetic may potentially overflow underflow
    - ( condition ) ? true do X : otherwise Y
    - try, catch can helps with errors external call
7.  global operators
    
    - delete : assigns default values, note that it has no effect on mappings. delete can operate on individual key value though.
    - new : create contract instances, fixed size arrays in memory
    - this :
    - arithmetic operators : \*\* exponet, % modulo
    - logic operators : ! negation
    - bitwize operators : &lt;< left shift, &gt;> right shift
8.  global variables : blockhash, blocknumber (only works for the 256 most recent blocks, excluding the current one)
    
9.  uint : (wei, gwei, ether) (seconds, minutes, hours, days, weeks)
    

### yul (Inline-assembly)

get access to memory and storage, while the stack is abstracted away

1.  inline assembly syntax
    
    - no semicolons
        
    - one assembly block is independent of other, no common namespace
        
    - read and write to any storage slots, 'x_slot' slot to which x is pointing
        
    - 'let' declare new variable, creates new storage slot. variable removed at end of the block.
        
    - local variables of functions can be used inside of assembly blocks
        
    - there is no 'else' statement, instead 'if', 'switch' available
        
2.  types: everything is a uint or bytes
    
3.  memory layout : Solidity reserves four 32-byte slots
    
    - 0x00 : scratchspace
        
    - 0x20 : scratchspace
        
    - 0x40 : free memory pointer ( go use memory as this points to)
        
    - 0x60 : zero slot, used as default value for uninitialized variables
        
    - Note : Dynamically sized arrays occupy one slot to point to the value in memory, one slot to indicate length, then one slot for each element.
        
4.  storage layout
    
    - static
        - Storage layout starts at slot 0.
        - The data is stored in the right-most byte(s).
        - If the next value can fit into the same slot (determined by type), it is right-aligned in the same slot, else it is stored in the next slot.
        - Immutable and constant values are not in storage, therefore they do not increment the storage slot count.
    - dynamic
        - A mapping slot is the Keccak-256 hash of the key value concatenated with the storage slot.
        - A dynamically sized array stores the current length in its slot, then its elements are stored sequentially starting at the Keccak-256 hash of the slot number.
        - Byte arrays and strings are stored the same way as other dynamic arrays unless the length is 31 or less. Then it is packed into one slot and the right-most byte is occupied by two times the length.
    - inheritance
        - Storage slots in a parent contract precedes the the child contract in order of inheritance.
5.  events
    
    - Events have up to four indexed topics. The first topic is always the Keccak-256 hash of the event signature.
    - Non-indexed topics are logged by storing them in memory and passing to the log instruction a pointer to the start of the data and the length of the data.
6.  errors : consist of a four byte error selector and the error data.
    

# Extra

### Miscellaneous

1.  Contracts have a 24kb size limit above which deployment is not possible
    
2.  Literals
    
    - Address : 20 bytes hexadecimal passing eip55 checksum
    - Integer : Underscores for digit seperation are semantically ignored
    - String : printable ASCII characters and a set of escape characters
    - Hexadecimal : prefixed with hex"literal"
    - Unicode Literals: prefixed with unicode"literal", valid UTF-8 sequence and a set of escape characters (emojis possible here)
3.  type(I).interfaceId: A bytes4 value containing the EIP-165 interface identifier of the given interface I.
    
4.  indexed adds up to three event parameters to a special data structure known as “topics” instead of the data part of the log. Topics allow you to search for events. All parameters without the indexed attribute are ABI-encoded into the data part of the log
    
5.  a++ and a-- return then update, ++a and --a update then return
    
6.  ABI and the address are sufficient for an application to interact with the contract.
    
7.  selfdestruct( ) : Destroy the current contract, sending its funds to the given Address and end execution.
    
8.  An external 'call' can use at most 63/64 of the gas currently available at the time of the call. Thus, depending on how much gas is required to complete a transaction, a transaction of sufficiently high gas (i.e. one such that 1/64 of the gas is capable of completing the remaining opcodes in the parent call) can be used to mitigate this particular attack.
    
9.  fringe conversion
    
    - uint converted to smaller type, higher-order bits are cut off
    - uint converted to larger type, padded on the left
    - bytesN converted to smaller type, cut off the bytes to the right
    - bytesN converted to larger type, pad bytes to the right.
