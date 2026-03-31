A decentralized insurance primitive for EVE Frontier tribes, built on Sui.

See it here! rift-mercantile.vercel.app



What it is

RIFT MERCANTILE lets EVE Frontier tribes pool SUI, sell coverage policies to members, and automatically pay out when a covered pilot dies in space. It's an on-chain financial protection layer built specifically for EVE Frontier's player-driven economy turning the inevitable losses of exploration and combat into a manageable, trust-minimized risk.



How it works

A tribe leader deploys an insurance pool directly on Sui, deposits SUI as the reserve, and sets payout and premium amounts. Members registered to the pool can purchase a policy through the dApp. When they die in EVE Frontier, they file a claim, the dApp verifies their death against live Stillness chain data using suix_queryEvents, and if verified, the contract releases the payout automatically.



The Policy NFT

Every purchased policy mints a Policy NFT to the buyer's wallet, a real Sui object representing their coverage certificate. It contains the holder's address, pool ID, and a purchase timestamp anchored on-chain. When a claim is filed and verified, the contract consumes the NFT (burning it permanently) and transfers the payout. One claim per policy, enforced by the Move contract.



Built in Move on Sui

The core protocol is a Move smart contract deployed on Sui testnet. It manages pool state as a shared Sui object, enforces membership checks before policy purchase, mints Policy NFTs via transfer::transfer, and handles claim payouts via coin::take. Events are emitted for every action, PoolCreated, PolicyPurchased, ClaimPaid ,making the full audit trail queryable on-chain forever. The contract uses clock::timestamp_ms() for tamper-proof policy timestamps.



Token support

Today RIFT MERCANTILE uses SUI as the reserve and premium currency, pools are funded in SUI, premiums are paid in SUI, payouts are in SUI. The architecture is designed to extend to any Sui-native token: the pool's Coin<T> type parameter can be swapped to support EVE's native token or any future Frontier-native asset once available on Sui, making RIFT MERCANTILE a general-purpose insurance primitive for the entire ecosystem.



EVE Vault Integration

RIFT MERCANTILE connects natively with EVE Vault, CCP's official wallet,  based on the Sui Wallet Standard. Players connect their EVE account directly. The dApp detects EVE Vault automatically on page load, then when a player chooses to execute coverage, the player is prompted to approve, and submits signed Programmable Transaction Blocks using the sui:signAndExecuteTransaction feature. Kill verification cross-references the connected wallet's on-chain character identity against live Stillness killmail events, tying the insurance claim directly to the player's in-game identity.



On-chain Construction

Every step of the protocol is verifiable on Suiscan testnet:





Contract: 0x0f31f8801c3b81f65f8dd0617aad7143411273ec11ea8f6ee7a9eae7d7e22b65

The RIFT MERCANTILE Move smart contract. All protocol logic lives here: create_pool, buy_policy, claim, add_member. Immutable once deployed. https://suiscan.xyz/testnet/object/0x0f31f8801c3b81f65f8dd0617aad7143411273ec11ea8f6ee7a9eae7d7e22b65




Live Pool (RIFT MERCANTILE ALPHA): 0x46299b5aab478290cea2261dfc1812c97276c810d4cbde9a3b01659362d88acc 
The active insurance pool object on-chain. Shared Sui object holding the 0.5 SUI reserve, member list, and policy settings. Anyone can interact with it. https://suiscan.xyz/testnet/object/0x46299b5aab478290cea2261dfc1812c97276c810d4cbde9a3b01659362d88acc




Policy NFT (purchased via EVE Vault): 0x425493084fe057a6fa5b07cf6b6de2d6d7d5243ce83a3126e45becf95c365950
An NFT sitting in the EVE Vault wallet. Records holder address, pool ID, and purchase timestamp. Consumed permanently when a claim is filed and payout released. https://suiscan.xyz/testnet/object/0x425493084fe057a6fa5b07cf6b6de2d6d7d5243ce83a3126e45becf95c365950




Policy purchase tx: DpczbKhQHZaMncgGWSB8jjJwPh5jzjwKxziCnqB8EPjq
The Sui transaction where the Policy NFT was minted. Signed by EVE Vault via the dApp. Proves the full end-to-end flow: wallet connected → premium paid → NFT received. https://suiscan.xyz/testnet/tx/DpczbKhQHZaMncgGWSB8jjJwPh5jzjwKxziCnqB8EPjq




Stillness World Package: 0x28b497559d65ab320d9da4613bf2498d5946b2c0ae3597ccfda3072ce127448c
CCP's official EVE Frontier game contract on Sui testnet. Source of all in-game data — characters, killmails, assemblies, gate jumps. RIFT MERCANTILE reads killmail events from this package to verify deaths. https://suiscan.xyz/testnet/object/0x28b497559d65ab320d9da4613bf2498d5946b2c0ae3597ccfda3072ce127448c




Killmail Registry: 0x7fd9a32d0bbe7b1cfbb7140b1dd4312f54897de946c399edb21c3a12e52ce283
The on-chain object storing all Stillness killmail records. Every player death emits a KillmailCreatedEvent from here. RIFT MERCANTILE queries this to confirm a covered pilot died after their policy was purchased before releasing any payout. https://suiscan.xyz/testnet/object/0x7fd9a32d0bbe7b1cfbb7140b1dd4312f54897de946c399edb21c3a12e52ce283


Live: rift-mercantile.vercel.app







Notes

¹ Currency: SUI is used as the reserve and premium token for this submission because EVE Frontier's native in-game currencies (LUX/EVE tokens) are not yet available as Sui-native assets. The contract is architected as Coin<T> and will support native Frontier tokens when they become available on-chain.


² Kill verification dependency: Claim verification relies on killmail event data indexed from the Stillness chain via suix_queryEvents. The accuracy and latency of this data is dependent on CCP's on-chain event infrastructure. In the current testnet environment, killmail propagation can lag by minutes to hours.


³ Scope of submission: This submission demonstrates the technical possibility of decentralized tribe fleet insurance on Sui,  not a production-ready financial product. The economic parameters (premium rates, payout ratios, pool sizing, actuarial modeling) are placeholders. The value being submitted is the protocol architecture, the Move contract, the EVE Vault integration pattern, and the proof that this class of financial primitive is buildable on EVE Frontier today.









Notes on the future:

In building this product, I notice that it was a primitive to more robust opportunities. Something I always considered was "Could we assign a risk rating, or credit rating, to a player, especially if they are paid to be a courier." And the answer to that viewpoint, I think, is one day. 



There are some V2 concepts worth exploring based off this project that I didn't have the time to commit to, but should there be another hackathon, I would love to.



V2 Concepts to explore, either as part of the dAPP, or its own.

1. POD-based claim verification Replace the killmail event polling with player-submitted PODs. Player retrieves their signed killmail POD from the game, pastes it into the claim form, dApp sends to /pod/verify. 


2. Open membership pools Currently tribe leaders manually add members by wallet address. V2 introduces an application flow. any player can request to join a pool, leader approves or rejects on-chain. Creates a real marketplace of competing insurance providers with different risk profiles, reputations, and rates. Basically, universal healthcare but universal insurance. lol. 


3. Risk-based premiums Use a player's on-chain kill/death history from killmail events to price premiums dynamically. Die frequently? Pay more. Never died? Pay less. First on-chain actuarial table in EVE, premiums reflect actual risk rather than a flat rate set by the pool leader.



4. Oracle cosigning for trustless claims Add a serverless function as a lightweight oracle. When a player files a claim, the oracle independently queries the Stillness chain, verifies the kill, and cosigns the transaction. The Move contract requires both the player signature AND the oracle signature to release funds. Closes the trust gap in the current frontend-verified model.



5. Pool-to-pool reinsurance Large pools can backstop smaller ones. If RIFT MERCANTILE ALPHA runs dry, a reinsurance pool covers the overflow. Creates a layered risk structure that mirrors how real insurance markets work. and gives veteran corps a way to earn yield by underwriting smaller tribe pools. This is what I call "when the primitive creates derivatives" 



6. SSU-based physical presence Wire RIFT MERCANTILE to a Smart Storage Unit in Stillness. The SSU becomes the physical insurance office in space. Players fly to it, interact with it, and get the dApp in the in-game browser with EVE Vault connected automatically. The office location becomes strategically relevant. destroy the SSU, disrupt the insurance market. Now inherently, this should be placed in jump points, or perhaps people can load their own contract to their own SSU to access the insurance marketplace.
