# Smart Contract Review 
## Task: Aavegotchi RaffleContract 
[Link To Contract](https://github.com/aavegotchi/raffle/blob/main/contracts/RafflesContract.sol)

*Introduction: This is basically a raffle smart contract with random number generation using the Chainlink VRF (Verifiable Random Function) service, the contract allows participants to enter a rafle by providing an ERC1155 tokens as tickets.*

**Indepth Review of All functions and implementations of the contract**

This is a Solidity contract with several import statements at the beginning. These import statements pull in the necessary interfaces and contracts needed for this contract to function properly. 

## Imports includes:
IERC1155.sol: This imports the ERC1155 token standard interface. 

**LinkTokenInterface.sol:** This is basically importing the interface for the chainlink token. It specifies the functions that a contract must implement to interact with the chainlink token.

**IERC173.sol:** Imports the ERC173 ownership standard. This specifies a way to manage ownership of a contract and allows for ownership to be transferred to another address.

**IERC165.sol:** Imports an interface of the ERC165 standard. Basically provides a way for contracts to advertise which interfaces they support.

---
## Structure Definitions
The contract itself contains two struct definitions: **AppStorage** and **Raffle**. The AppStorage struct is used to hold state variables for the contract. This struct contains several mappings that are used to store data related to the raffles that are being managed by the contract, including supported interfaces, a list of raffles, nonces for VRF keyHash, a mapping from request IDs to raffle IDs, a VRF keyHash, a fee, and the contract owner's address.

The Raffle struct is used to represent an individual raffle that is managed by the contract. It contains several mappings that are used to store data related to the raffle, including the raffle item indexes for each ticket address, a list of raffle items, the entries made by participants, the entrants, the random number generated by the VRF process, and a boolean indicating whether a random number is pending. It also includes the date when the raffle ends.

Additionally, the contract includes a struct definition for **Entry**, which represents an entry made by a participant into a raffle item. It includes the index of the raffle item, a boolean indicating whether prizes have been claimed, and the range of ticket numbers entered for the raffle item.

There is a struct definition for **RaffleItemPrize**, which represents a prize that can be won for a particular raffle item. It includes the address of the ERC1155 token contract, the quantity of tokens, and the token ID. 

The **RaffleItem** struct represents a single raffle item, including the address and ID of the ERC1155 token contract, the ID of the raffle item, the total number of tokens entered into the raffle for that item, and a list of potential prizes that can be won for that item.

---
## Contract Review

The RafflesContract contract implements the IERC173 and IERC165 interfaces, which are used to manage ownership and check whether a contract implements a specific interface respectively.

The s variable is of the AppStorage struct type and is used to access all the state variables in the contract. The **AppStorage** struct defines the state variables used by the contract.

The **im_link** and **im_vrfCoordinator** variables are immutable variables that hold the addresses of the Chainlink LINK token contract and the VRF coordinator contract, respectively.

The **ERC1155_ACCEPTED** constant is defined to hold the value returned by the onERC1155Received function when a contract accepts receipt.

The **RaffleStarted**, **RaffleTicketsEntered**, **RaffleRandomNumber**, and **RaffleClaimPrize** events are defined to emit information about each raffle events.

The constructor function initializes the contract's state variables, including the **contract owner**, the **VRF coordinator** contract address, the **LINK token contract address**, the **VRF key hash**, and the **VRF fee**. It also adds support for the IERC165 and IERC173 interfaces.

The **supportsInterface** function is an implementation of the IERC165 interface and returns a Boolean indicating whether a contract implements a specific interface.

The **nonces** function takes a _keyHash parameter and returns the nonce for the given _keyHash. It is used to keep track of the nonces associated with VRF key hashes from which randomness has been requested.



The **drawRandomNumber** function is called to generate a random number for a specific raffle. This function first checks that the raffle exists, and that its end time has passed. It then checks that a random number has not already been generated for this raffle, and that there is not already a pending request for a random number. If these conditions are met, it sets randomNumberPending to true, indicating that a request for a random number has been made.

The **requestRandomness** function is then called to initiate a request for a random number from the Chainlink VRF service. This function transfers the required fee in LINK tokens to the VRF coordinator contract and passes the key hash and seed values as encoded parameters. It then generates a VRF seed value using makeVRFInputSeed function, increments the nonce value, and generates a request ID using makeRequestId function. The request ID is used to identify the response from the VRF service.

The **makeVRFInputSeed** function takes the key hash, user-supplied seed, requester's address, and nonce values as input, and generates a hash value using the keccak256 hash function. The resulting hash is used as the input seed value for the VRF coordinator.

The **makeRequestId** function takes the key hash and VRF seed values as input, and generates a unique request ID using the keccak256 hash function. This request ID is used to identify the response from the VRF service.

When a response is received from the VRF service, the **fulfillRandomness** function is called to process the response. This function checks that the response matches the request ID, and that the random number generated is within the specified range. If these conditions are met, the random number is stored in the raffle struct, and the randomNumberPending flag is set to false, indicating that the random number has been generated. 


**rawFulfillRandomness:** This function is a callback function that is called by the VRF Coordinator when it receives a valid VRF proof. It takes two parameters, _requestId, which is a bytes32 type variable representing the unique ID for the VRF request, and _randomness, which is a uint256 type variable representing the random number generated by the VRF. Within the function, it first checks that the function is being called by the VRF Coordinator and then checks that the corresponding raffle for the given _requestId exists and that its raffle end time has already passed. If these conditions are met, the random number generated by the VRF is stored in the raffle's randomNumber field and the randomNumberPending flag is set to false. Finally, an event is emitted containing the raffle ID and the generated random number.

**changeVRFFee:** This function is used to change the fee amount that is paid for VRF random numbers. It takes two parameters, _newFee, which is a uint256 type variable representing the new fee amount, and _keyHash, which is a bytes32 type variable representing the VRF key hash. Within the function, it checks that the caller is the contract owner and then updates the contract's fee and keyHash fields with the new fee amount and VRF key hash.

**removeLinkTokens:** This function is used to remove the LINK tokens from the contract that are used to pay for VRF random number fees. It takes two parameters, _to, which is the address to send the tokens to, and _value, which is a uint256 type variable representing the number of tokens to transfer. Within the function, it checks that the caller is the contract owner and then calls the transfer function of the LINK token contract to transfer the specified amount of tokens to the specified address.

**linkBalance:** This function is used to retrieve the current balance of LINK tokens held by the contract. It returns a uint256 type variable representing the current LINK token balance of the contract.

**startRaffle**
This function starts a raffle by taking in a duration for the raffle and an array of RaffleItemInput structs that describe what ERC1155 tickets can be entered for what ERC1155 prizes. The prizes that can be won are then transferred to this contract.

The function first checks that the caller is the contract owner and that the raffle duration is greater than or equal to 1 hour. It then calculates the end time for the raffle based on the duration and initializes a new Raffle struct 

For each RaffleItemInput struct in the input array, the function checks that there are at least one prize associated with it and that the combination of ticket address and ticket ID hasn't already been used in this raffle. If these checks pass, the function initializes a new RaffleItem struct in the raffle's raffleItems array, adds the ticket address and ticket ID to the raffle's raffleItemIndexes mapping, and transfers the associated prizes to this contract.


**onERC1155Received:** This function is a standard ERC1155 callback function that is called when a user sends ERC1155 tokens to this contract. The function verifies that the transferred tokens are intended for a raffle and that the raffle is still open for participation. If the transfer is valid, the function returns a bytes4 value indicating that it accepts the transfer. If the transfer is not valid, the function reverts.

**getRaffles:** This function returns an array of RaffleIO struct that contain simple information about each raffle that has been created. The RaffleIO struct has three fields: raffleId, raffleEnd, and isOpen.

**raffleSupply:** This function returns the total number of raffles that have been created.

**raffleInfo:** This function returns detailed information about a specific raffle. The function takes a raffleId parameter and returns three values: raffleEnd, an array of RaffleItemOutput structures that contain information about each raffle item in the raffle, and randomNumber, which is the random number that will be used to determine the raffle winner. The RaffleItemOutput struct has four fields: ticketAddress, ticketId, totalEntered, and raffleItemPrizes. The raffleItemPrizes field is an array of RaffleItemPrizeIO structures that contain information about each prize associated with the raffle item. The RaffleItemPrizeIO struct has three fields: prizeAddress, prizeId, and prizeQuantity.


**getEntries** This function retrieves ticket information for a single entrant (address) in a raffle. It takes two parameters: _raffleId specifies which raffle to get ticket information about, and _entrant is the address of the entrant to retrieve information for. It returns an array of EntryIO structs, each of which contains information about a ticket entered by the given entrant.

**ticketStats** This function retrieves data about the tickets entered into a raffle. It takes one parameter: _raffleId specifies which raffle to get data for. It returns an array of TicketStatsIO structs, each of which contains information about a type of ERC1155 ticket entered into the raffle.

**enterTickets** This function allows a user to enter ERC1155 tokens for raffle prizes. It takes two parameters: _raffleId specifies which raffle to enter tickets for, and _ticketItems is an array of TicketItemIO structs, each of which contains information about an ERC1155 ticket to enter into the raffle.

The code also contains three structs: EntryIO, TicketStatsIO, and TicketItemIO.

EntryIO is a struct that contains information about a single ticket entered by an entrant. It has the following fields:
ticketAddress: the address of the ERC1155 contract for the ticket
ticketId: the ID of the ERC1155 ticket
ticketQuantity: the number of ERC1155 tokens in the ticket
rangeStart: the starting index of the range of tokens in the ticket
rangeEnd: the ending index of the range of tokens in the ticket
raffleItemIndex: the index of the raffle item associated with the ticket
prizesClaimed: a boolean indicating whether any prizes associated with the ticket have been claimed
TicketStatsIO is a struct that contains statistics about a type of ERC1155 ticket entered into a raffle. It has the following fields:
ticketAddress: the address of the ERC1155 contract for the ticket
ticketId: the ID of the ERC1155 ticket
numberOfEntrants: the number of unique addresses that entered tickets for this ticket type
totalEntered: the total number of ERC1155 tokens entered for this ticket type
TicketItemIO is a struct that contains information about an ERC1155 ticket to enter into a raffle. It has the following fields:
ticketAddress: the address of the ERC1155 contract for the ticket
ticketId: the ID of the ERC1155 ticket
ticketQuantity: the number of ERC1155 tokens to enter for the ticket.

**claimPrize**: This is the claimPrize function of the Raffle contract. It allows an entrant or the contract owner to claim prizes that have been won in a raffle. The function takes three parameters:

_raffleId: the ID of the raffle where the prizes were won.
_entrant: the address of the entrant who won the prizes.
_wins: an array of ticketWinIO structs that represent the winning entries and prizes that were won.
The function first checks that the raffle with the given ID exists and that the random number has been generated. It then checks that the function caller is either the entrant or the contract owner.

The main logic of the function is as follows:

Loop through each ticketWinIO struct in the _wins array.
Verify that the entryIndex provided exists and is not a duplicate.
Loop through each PrizesWinIO struct in the prizes array of the ticketWinIO struct.
Verify that the raffleItemPrizeIndex provided exists and is not a duplicate.
Loop through each winningPrizeNumbers value in the PrizesWinIO struct.
Verify that the winningPrizeNumbers value exists and is not a duplicate.
Verify that the winningPrizeNumbers value actually won.
Transfer the prizes to the winner.
The lastValue variable is used in the loops to ensure that values are less than the length of the relevant arrays and to prevent duplicates.

The function emits a RaffleClaimPrize event for each prize that is claimed, which includes the raffle ID, entrant address, prize address, prize ID, and the number of winning prize numbers.

Finally, the function transfers the prizes from the contract to the entrant using the safeTransferFrom function of the ERC1155 token standard.

---
## Thoughts
Overall, the code is well written and follows best practices of moduar solidity smart contract development.

## Suggestions

Add a check in the raffleInfo function to ensure that the raffle is not expired before allowing users to retrieve information about it. This can be done by adding the same require(raffleEnd > block.timestamp, "Raffle: Raffle has expired"); check used in the onERC1155Received function.

It’s more gas efficient to use custom errors to require statements

