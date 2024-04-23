
# Building an Art Market Platform and deploy on Celo the Blockchain

### Table of contents
1. Introduction
2. Prerequisites
3. Understanding Art Market Smart contract
4. Creating the Art Market Smart Contract
5. Deploying to Celo with Remix
6. Interacting with the Smart Contract
7. Conclusion
#### Introduction
In this tutorial, we'll explore how to build an Art Market contract on the Celo network using Solidity and deploy it through Remix. This contract will provide a decentralized platform for artists to showcase their artwork, buyers to purchase art, and collectors to trade valuable pieces. Let's dive in!

#### Prerequisites
To build an art market platform on the Celo network, you'll need:

* Basic understanding of Solidity and smart contracts.
* Development environment setup: For this tutorial, we'll use Remix, To access to Remix IDE (https://remix.ethereum.org/),an online code editor.
* Metamask Wallet [metamask.io](https://https://metamask.io/download/)
* Internet access.

#### Understanding Art Market Assistant Smart contract
The Art Market contract will serve as a decentralized marketplace for buying, selling, and trading artworks on the Celo blockchain. It will allow artists to register their artwork, set prices, and receive payments securely. Buyers can browse through the available artwork, and purchase directly.
##### Key functionalities
* Artwork Creation: Artists can create new artworks by providing details such as title, description, and price using the createArtwork function. This functionality enables artists to list their artworks for sale on the platform.

* Artwork Purchase: Buyers can purchase artworks by calling the purchaseArtwork function and sending the required amount of Celo. The contract verifies if the artwork is available and if the sent value is sufficient, then facilitates the transfer of ownership and payment to the artist.

* Artwork Availability Tracking: The contract keeps track of the availability of artworks using the available field in the Artwork struct. This ensures that artworks can only be purchased when they are available for sale.

* Ownership Tracking: The contract maintains ownership information for each artwork by storing the address of the artist who created it. This allows for transparent ownership tracking and ensures that artists receive payment when their artworks are sold.

* Event Logging: Important actions such as artwork creation and purchase are logged using events (ArtCreated and ArtPurchased). This enables users and external systems to track and react to changes in the contract state.

* Withdrawal of Funds: The contract owner can withdraw funds from the contract using the withdraw function. This functionality allows the owner to access the proceeds from artwork sales.
##### Design Principles:

* Security: Prioritizes the security and integrity of art transactions, safeguarding against fraud and counterfeit artworks.
* Transparency: Maintains a transparent and immutable record of all art transactions on the blockchain, enhancing trust and confidence among users.
* Fairness: Ensures fair and equitable treatment of artists, collectors, and buyers, promoting a sustainable and inclusive art market ecosystem.

### Creating the Art Market Smart Contract:
To implement the Art Market Contract, we'll create a smart contract file named "ArtMarket.sol" in your code editor(Remix). Let's define the contract and its key components.

1. Pragma Directive:
```solidity=
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;
```
This line specifies the compiler version of Solidity to be used. ^0.8.19 means any version greater than 0.8.0. Usually, you would want to use the latest Solidity compiler version, as a new version usually implies either new features or optimizations.

2. Contract Declaration:
```solidity=
contract ArtMarket{
    address public owner;
    uint public artCounter;
      }
```
This declears start the ArtMarket contract and its state variables.
State Variables: These variables hold persistent data on the blockchain.

* owner: This variable stores the address of the contract owner, who has special privileges such as withdrawing funds.
* artCounter: This variable keeps track of the number of artworks created, serving as an identifier for new artworks.

3. Structs:
```solidity=
struct Artwork {
        uint id;
        address payable artist;
        string title;
        string description;
        uint price;
        bool available;
    }
```
This part of the code defines a struct called Artwork, A struct is a custom data type that allows you to combine different types of data (like strings, booleans, addresses, etc.),In this contract it is used to represent an artwork listing.
* id: Unique identifier for the artwork.
* artist: Ethereum address of the artist who created the artwork.
* title: Title or name of the artwork.
* description: Description of the artwork.
* price: Price of the artwork, denoted in wei.
* available: Boolean indicating whether the artwork is available for purchase

4. Mappings:
```solidity=
    // Mapping to store artworks by their IDs
    mapping(uint => Artwork) public artworks;
```
A mapping is a key-value store that allows you to associate values with unique keys. In the context of the Art Market smart contract, the artworks mapping is used to store information about each artwork registered on the platform.
 Here's how the mapping works:
* Key: Each artwork is assigned a unique identifier, represented by an unsigned integer (uint). This identifier serves as the key for the mapping.
* Value: The value associated with each key is an instance of the Artwork struct, which contains details such as the artwork's title, description, artist address, price, and availability status.
When an artist creates a new artwork, the contract generates a unique ID for the artwork (using artCounter) and uses this ID as the key to store the artwork details in the artworks mapping. When someone wants to retrieve information about a specific artwork, they can use the artwork's ID as the key to access its details from the mapping.
5. Events:
```solidity=
    event ArtCreated(uint id, address artist, string title, uint price);
    event ArtPurchased(uint id, address buyer, string title, uint price);
    event Withdrawal(address receiver, uint amount);

```
These events are emitted to log significant actions within the contract. They provide a way for external systems to monitor and react to changes in the contract state. In this case,` ArtCreated,ArtPurchased and Withdrawal` events are emitted when new artworks are created,purchased and when withdrawal is made respectively.
Events in Solidity are an essential feature used to facilitate communication between smart contracts and external applications, such as user interfaces or other smart contracts. They are crucial for transparency, auditability, and real-time monitoring of contract activities. 

5. Modifiers:
```solidity=
modifier onlyOwner() public {
        require(msg.sender == owner, "Only the owner can call this function.");
        _;
    }
```
Modifiers are primarily used for access control, allowing developers to restrict access to certain functions based on specific conditions. The `onlyOwner` modifier ensures that only the contract owner can execute certain privileged actions within the contract.

6. Constructor:
```solidity=
constructor() {
        owner = msg.sender;
        artCounter = 0;
    }
```

This constructor function is executed only once when the contract is deployed to the blockchain, Here the constructor initializes the contract owner address and the art counter,It ensures that valid addresses are provided for the contract.

* `owner = msg.sender;`: This line assigns the address of the account that deployed the contract (msg.sender) to the owner variable. The contract owner typically has special privileges, such as withdrawing funds or performing administrative tasks.
* `artCounter = 0;`: This line initializes the artCounter variable to zero. artCounter is used to assign unique identifiers to artworks created on the platform. By setting it to zero initially, the contract ensures that the first artwork created will have an ID of 1 (since IDs typically start from 1)

7. **Functions**
 In Solidity are named blocks of code within a contract that execute specific tasks, potentially taking input parameters and returning values. They are crucial for defining the behavior and organizing the functionality of smart contracts. In this contract, key functions include createArtwork for adding new artworks, purchaseArtwork for buying them, receive for handling Ether payments, and withdraw for withdrawing funds.

 **createArtwork Function**
```solidity=
function createArtwork(string memory _title, string memory _description, uint _price, bool _available) public {
    require(bytes(_title).length > 0, "Title cannot be empty.");
    require(bytes(_description).length > 0, "Description cannot be empty.");
    require(_price > 0, "Price must be greater than zero.");

    artCounter++;
    artworks[artCounter] = Artwork(artCounter, payable(msg.sender), _title, _description, _price, _available);
    emit ArtCreated(artCounter, msg.sender, _title, _price);
}
```
This function allows an artist to create a new artwork and add it to the marketplace.
* Parameters: `_title` (title of the artwork), `_description` (description of the artwork), `_price` (price of the artwork), `_available` (availability status of the artwork).
* It checks that the title, description, and price are valid (not empty and greater than zero).
* Increments the artCounter to generate a unique ID for the artwork.
* Creates a new Artwork struct with the provided details and adds it to the artworks mapping.
* Emits an ArtCreated event to log the creation of the artwork.


**purchaseArtwork Function**
```solidity=
 function purchaseArtwork(uint _id, uint256 _price) public payable {
     uint totalPrice = msg.value; // Initialize total price with the sent Ether
    totalPrice += artworks[_id].price; // Add the price of the artwork to the total price

    require(artworks[_id].available == true, "Artwork is not available.");
    require(msg.value < _price, "Insufficient funds.");

    artworks[_id].available = false;
    artworks[_id].artist.transfer(msg.value);
    emit ArtPurchased(_id, msg.sender, artworks[_id].title, _price);
}
```
This function allows a buyer to purchase an artwork from the marketplace.
* Parameters: _`id` (ID of the artwork to purchase), `_price` (price agreed upon for the purchase).
* It calculates the total price by adding the value sent with the transaction (msg.value) to the price of the artwork.
* Checks that the artwork is available for purchase and that the sent value is sufficient to cover the price.
* If the conditions are met, marks the artwork as unavailable, transfers the payment to the artist, and emits an ArtPurchased event to log the purchase.


**receive Function**

```solidity=
  receive() external payable { }
```

This fallback function is called when the contract receives Celo without any function call data.
* It allows the contract to accept payments without calling any specific function.
* Marked as payable to indicate that it can receive Ether.


 **withdraw Function**
 ```solidity=
function withdraw() public onlyOwner {
    uint balanceBefore = address(this).balance;
    (bool success, ) = payable(owner).call{value: balanceBefore}("");
    require(success, "Withdrawal failed");
    uint256 amountWithdrawn = address(this).balance - balanceBefore;
    emit Withdrawal(msg.sender, amountWithdrawn);
}
```
This function allows the contract owner to withdraw funds from the contract.
* It transfers the entire balance of the contract to the owner's address.
* Emits a Withdrawal event to log the withdrawal.


Each function serves a specific purpose within the contract, enabling the creation, purchase, and withdrawal of funds related to artworks in the Art Market.

##  Deploying to Celo with Remix
1. Open Remix IDE: Go to https://remix.ethereum.org/ to access Remix IDE.
2. Create or import the contract: You can either create a new file and copy-paste the contract code into it, or import an existing file if you already have the code.
3. ![Screenshot 2024-04-08 192238](https://hackmd.io/_uploads/S1nEX9GxA.png)

4. Compile the contract: Click on the "Solidity Compiler" tab on the left sidebar, then click the "Compile" button to compile the contract. Ensure that there are no errors in the compilation output.
![Screenshot 2024-04-06 235702](https://hackmd.io/_uploads/rJ0Nb9ZgC.png)
5. Set up your MetaMask,Now to proceed, ensure you have connected Metamask or the provider you are using to CELO alfajores. You can follow this guide to connect CELO to Metamask.

Next, select `injected provider`  under your environments to connect to Metamask and click on deploy button.


6. Deploy the contract: Switch to the "Deploy & Run Transactions" tab on the left sidebar. Here, you'll see your contract listed under the "Deployed Contracts" section. Click on the contract name to deploy it.
7. ![Screenshot 2024-04-08 154606](https://hackmd.io/_uploads/HkQoZqWeA.png)

8. Interact with the contract: Once deployed, you'll see the contract interface below. This interface provides access to the contract's functions and allows you to interact with it. You can call functions like createArtwork, purchaseArtwork, and withdraw, and provide the required parameters.
9. Test each function: Enter the necessary parameters for each function you want to test, such as title, description, price, etc. Then, click the "Transact" button to execute the function.
10. Check the output: After executing a function, you'll see the transaction details and any emitted events in the console below. You can verify that the contract behaves as expected based on the function's logic and event emissions.
11. Debug and iterate: If you encounter any issues or unexpected behavior, you can debug the contract using Remix's debugging tools and make necessary adjustments to the code. Rinse and repeat until the contract functions correctly.

By following these steps, you can effectively test the Art Market smart contract using Remix IDE and ensure its functionality meets your requirements.
When you are done compiling and deploying your smart contract you can expect to see your smart contract which will look like this.
![Screenshot 2024-04-05 205629](https://hackmd.io/_uploads/SyrTt9AyR.png)


### Possible Improvement
There are several potential improvements that could enhance the functionality, security, and usability of the Art Market smart contract:

* Access Control: Implementing more granular access control mechanisms can enhance security and prevent unauthorized users from executing critical functions. For example, allowing only the contract owner to add or remove artworks, while permitting anyone to purchase them.

* Error Handling: Enhance error handling mechanisms to provide clearer error messages and gracefully handle exceptional scenarios. This ensures users understand why their transactions fail and improves the overall user experience.

* Gas Optimization: Optimize contract functions to minimize gas costs and improve efficiency, particularly for operations like artwork creation and purchase, which involve storage reads and writes.

* Event Logging: Expand the use of events to log important contract actions and state changes comprehensively. This provides transparency and allows external systems to react to contract events in real-time.

* Metadata Storage: Store additional metadata for each artwork, such as artist information, creation date, and genre. This enriches the artwork data and enables more sophisticated search and filtering functionalities.

* Escrow Mechanism: Implement an escrow mechanism to hold funds in escrow until the buyer confirms receipt of the artwork. This adds an extra layer of security and builds trust between buyers and sellers.

* Standardization: Consider adhering to industry standards, such as ERC-721 for non-fungible tokens (NFTs), to ensure interoperability with other platforms and ecosystems. This can facilitate integration with NFT marketplaces and decentralized finance (DeFi) protocols.

* User Interface: Develop a user-friendly interface or frontend application to interact with the smart contract, making it more accessible to non-technical users and improving the overall user experience.


### Conclusion
In this tutorial, we've delved into a Solidity smart contract tailored for the art market on the celo blockchain. We've uncovered its inner workings, such as using structures to hold artwork details, mappings to store and retrieve art data, and events to keep track of important actions. Through functions like creating artwork, buying art, and withdrawing funds, we've seen how this contract enables smooth art transactions.
Moreover, we've highlighted potential enhancements, like beefing up security measures, refining implementation aspects, adding new features, and addressing usability challenges. By addressing these areas, we can ensure the contract becomes even more reliable and user-friendly, paving the way for a vibrant and accessible art marketplace on the blockchain. Hope this tutorial has shed light on the exciting possibilities ahead.








 