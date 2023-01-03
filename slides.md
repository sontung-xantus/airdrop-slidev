---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: ./images/bg.jpeg
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Oxalus airdrop developer guideline
  Token Airdrop developer guideline

  Learn more at [Oxalus](https://oxalus.io)
# persist drawings in exports and build
drawings:
  persist: false
# use UnoCSS
css: unocss
---

# Airdrop

Token Airdrop developer guideline

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    Press Space for next page <carbon:arrow-right class="inline"/>
  </span>
</div>

<div class="abs-br m-6 flex gap-2">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:edit />
  </button>
  <a href="https://github.com/sontung-xantus/airdrop-slidev" target="_blank" alt="GitHub"
    class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---

# Intro

A cryptocurrency airdrop is a marketing strategy that involves sending coins or tokens to wallet addresses.
Oxalus use airdrop strategy for token, nft and event.

- ✉️ **Direct** - send token directly to users
- 📖 **On-chain whitelist** - everything is on chain
- 💾 **Server-side signed** - magic behind APTony sign-service
- 🌲 **Merkle tree** - off-chain list, verify on-chain, no retry, no deadline, but more complex

<br>
<br>

Also sharing some Solidity basic

<!--
You can have `style` tag in markdown to override the style for the current page.
Learn more: https://sli.dev/guide/syntax#embedded-styles
-->

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>


---

# Send Directly

```ts
import ABI from 'abi.json'

const address = '0xa123...'
const OxalusPassContract = await ether.getContractAt(ABI, address);
const tx = await oxalusPass.safeMint(address, item.name, item.number);
const res = await tx.wait();
```
<div class="grid grid-cols-2 gap-x-4 py-4">
<div v-click>


###### Pros

- Simple
- No gas for users

</div>

<div v-click>


###### Cons

- Costly: $0.15 for nft minting [BSCScan](https://bscscan.com/tx/0x4b549779c7cf8420d38779620c0e662cf4c9528609a5a97c435c066c0a5c5bec) ($1500/ 10k users)
- Painful with big list

</div>

</div>

<div v-click class="py-4">

###### Available tools
- [Token disperse](https://disperse.app/)
- [CoinTool](https://cointool.app/multiSender/bsc)
- [OxaSend](https://oxa-send.vercel.app/) 

</div>

<div v-click>Donate 🌮 for development and maintenance OxaSend</div>

---

# On-chain whitelist (Solidity basic)

Solidity types

```solidity {all|1-2|4-5|7-14|16|all}
// array
uint[] public arr;

// mapping
mapping(address => uint) public balances;

// struct
struct CampaignInfo {
  address paymentToken;
  uint256 startDate;
  uint256 duration;
  uint256 inviterCommission;
  uint256 buyerInvitedDiscount;
}

CampaignInfo[] public campaignInfo;
```


---

# On-chain whitelist (Implementation)

```solidity {all|1|2-7|9-18|all}
    mapping(address => uint) public whitelist;
    // admin setup
    function addWhitelist(address[] calldata users, uint[] calldata tokenIds) external onlyOwner { 
        for (uint i = 0; i < users.length; i++) {
            whitelist[users[i]] = tokenIds[i];
        }
    }

    function mint() external {    // users call for getting airdrop reward
        if (whitelist[msg.sender] == 0) {
            // throw error and revert tx
            revert NotWhiteList();
        }
        uint tokenId = whitelist[msg.sender];
        IERC721(nftAddress).transferFrom(address(this), msg.sender, tokenId);
    }
```

<div class="grid grid-cols-2 gap-x-4 py-4">
<div v-click>

###### Pros

- Gas saving [BSCScan](https://testnet.bscscan.com/tx/https://testnet.bscscan.com/tx/0xcd3fdcb55b109c8cc22bf456b8884381f5c297e449c5a938fd374251e2d173f1) ($517/ 10k users)
- Can add more logic for whitelisting
</div>
<div v-click>

###### Cons

- Costly
- Need setup on chain, no automation
</div>
</div>
---

# Server-side signed

<div class="h-full m-auto">
<div class="h-3/4 flex">
<img src="/images/verify.png" alt="" class="h-full">
<div>

#### Using asymmetric encryption

###### Smart contract
- Get public key from **signature** => get signer address from public key
- Get check address == backend signer address
- Execute airdrop

###### Backend
- Verify user request
- Update user balance in database (prevent double spending)
- Generate **signature** with signer private key 
</div>
</div>
<div class="grid grid-cols-2 gap-x-4 py-4">
<div v-click>

###### Pros

- No Gas
- can customize logic on backend before generate signature
</div>
<div v-click>

###### Cons

- Need to securely store private key
- Not good UX with retry (improved with deadline)
</div>
</div>
</div>
---

# Merkle tree

<div class="grid grid-cols-2 gap-x-4 py-4">
<img src="/images/merkle-tree.png" alt="" class="h-full">
<div>
<div v-click>

######  What's a hash?

A hashing function is a one way(non invertible function) that maps a set of inputs to a set of outputs **hash(s) -> p**


```
keccak("1") =                               = "c89efdaa54c0f20c7adf612882df0950f5a951637e0307cdcb4c672f298b8bc6"
keccak("2") =                               = "ad7c5bef027816a800da1736444fb58a807ef4c9603b7848673f7e3a68eb14a5"
keccak("Transfer(address,address,uint256)") = "ddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef"
```


</div>
</div>
</div>
---

# Merkle Proof

<div >
<img src="/images/proof.png" alt="" class="h-3/4">
<div v-click class="pt-8">

**Merkle Proof** is used to check whether the input data belongs to the Merkle Tree or not without having to reveal all the data that make up the Merkle Tree.


</div>
</div>

---

# Merkle Airdrop

1. Collect user address and airdrop token amount
   ```
    0x19171a5da52276b6a034CB859ddA1e905739F8B2 15000000000000000000
    0x04d1eC716Fe9AC219D59b9E4f0D64D3B4339642e 10000000000000000000
    0x14C06EC9402f7CD52dd0AF02979a350EAF133F76 20000000000000000000
   ```

2. Calculate the Merkle Root and save it on the smart contract
3. Users will then call the Merkle Airdrop contract to claim the registered token amount with input:
    ```
   amount
   proofs (all hash need to verify user data)
   ```

---

# 

<div class="grid grid-cols-2 gap-x-4 py-4">
<img src="/images/linus.png" alt="" class="h-full">
<div>
<div v-click>

# Talk is cheap. Show me the code.


</div>
</div>
</div>

---

# Learn More

[Github](https://github.com/topics/airdrop) · [Opensea](https://github.com/slidevjs/slidev) · [Uniswap](https://sli.dev/showcases.html)
---

