# zkWallet (Social Recovery) <!-- omit in toc -->

![zkWalletImage](images/zkWallet.png)

zkWallet is a smart contract wallet using zk-SNARKS to restore a user's access to their smart contract wallet, without revealing any information of the user's and guardians.

The project is currently on [Harmony Testnet](https://explorer.pops.one/) and the frontend is hosted on [Vercel](https://github.com/vercel/vercel).

zkWallet Link:

<https://zkwallet.simplicy.io/>

Guardian Registration Link:

<https://guardian.simplicy.io/registration>

Guardian Recovery Link:

<https://guardian.simplicy.io/>

zkWallet Demo Video:
<https://youtu.be/u2cMV73a3lY>

## Table of Contents <!-- omit in toc -->

- [Background](#background)
  - [What is social recovery](#social-recovery)
  - [Our approach](#our-approach)
    - [Overview](#flow-overview) 
    - [Guardian Registration flow](#guardian-registration)
    - [Regular flow for a Owner](#owner-flow)
    - [Recovery Flow for Guardians](#recovery-flow)
    - [Guardian Management Flow for an Owner](#guardian-management)
- [Project Structure](#project-structure)
  - [circuits](#circuits)
  - [contracts](#contracts)
  - [guardian-service](#guardian-service)
  - [guardian-ui](#guardian-ui)
  - [zkWallet-ui](#wallet-ui)
- [Zero Knowledge Structure](#zk-structure)
- [Run Locally](#run-locally)
  - [Clone the Repository](#clone-the-repository)
  - [Get the Submodules](#get-submodules)
  - [Run circuits](#run-circuits)
  - [Run contracts](#run-contracts)
  - [Run Guardian Service](#run-guardian-service)
  - [Run Guardian UI](#run-guardian-ui)
  - [Run Wallet UI](#run-wallet-ui)
- [Deployed contracts](#deployed-contracts)
  - [Devnet](#deployed-devnet)
  - [Testnet](#deployed-testnet)

## Background

### What is social recovery
Social recovery is a smart contract wallet technique whereby a user is able to grant trusted persons the ability to restore the user access to their account. This is useful in case the user accidentally gets locked out of their account - access can simply be restored and the user is able to again access their account. 

Vitalik released a [post](https://vitalik.ca/general/2021/01/11/recovery.html) about the need for wider adoption of social recovery wallets. Multi-sigs have thus far been the standard for more secure custody of funds, but require multiple parties to coordinate on transactions. Using just a single key paradigm (e.g. Metamask) can lead to loss of funds if the private key is lost, or theft if a hacker acquires the single private key. Thus, we have social recovery wallets. 

Social Recovery wallets are designed to mitigate against two scenarios: 
1. The user loses their private key 
2. A hacker obtains a user's private key

The image below describes the flow of a social recovery wallet. A single owner is able to sign off on transactions, but a set of guardians is able to change the owner (the signing key). 

![image](images/diag2.png)

<a name="our-approach"/>

### Our approach
The previous approach places the Ethereum address of the guardian in plain text on-chain. We seek to instead keep the guardian address private, whilst still allowing social recovery functionality.

To do this, when a user is adding a guardian, using zero knowledge addGuardians() with Semaphore. Semaphore allows Ethereum users to prove their membership of a group and send signals such as votes or endorsements without revealing their original identity. 

<a name="flow-overview"/>

#### Overview
![FLow overview](./images/flow-overview.png)

<a name="guardian-registration"/>

#### Guardian Registration flow
- Guardian register to **register-service**
- Guardian sign with identitycommitment
- After approval, the identitycommitment of the guardian is saves in the **factory contract**

| Step 1                                                                     | Step 2                                                                        |
| :---                                                                       | :---                                                                          |
|<img src="images/guardian-registration.png" alt="drawing" width="400"/>     | <img src="images/guardian-sign.png" alt="drawing" width="400"/>               |                                                
| mongodb result:                                                            | <img src="images/mongodb-after-guardian-sign.png" alt="drawing" width="400"/> |

<a name="owner-flow"/>

#### Regular flow for a Owner
Social recovery wallets are meant to minimze the burden that an owner faces when making transactions. Thus, the flow consists of just a call `acceptOwnership()` with the new nominee wallet address

<a name="recovery-flow"/>

#### Recovery Flow for Guardians
In the event that an owner loses their private key, guardians can be notified and a recovery process can be kicked off. 
- Guardian calls `**recover**` with the address of the newOwner.
- `majority` number of guardians call `recover` with the newOwner.
- If majority agreed with the newOwner, the new owner address will become a nominee wallet address

<a name="guardian-management"/>

#### Guardian Management Flow for an Owner
Owners have the ability to swap out guardians in the case that a guardian's keys are compromised or a guardian becomes malicious. 
- Owner calls `removeGuardian` or ``removeGuardians` with hash of a guardian's. 
- Owner calls `addGuardian` or `addGuardians` with the hash of the guardian's. This queues the guardian for adding – the guardian can only be added after a time delay of 3 days. 

<a name="project-structure"/>

## Project Structure

The project has five main folders within packages directory:

- packages/zkWallet-contracts/circuits
- packages/zkWallet-contracts/contracts
- packages/guardian-service
- packages/guardian-ui
- packages/zkwallet-ui

<a name="circuits"/>

### circuits

The [circuits folder](/packages/zkWallet-contracts/circuits/) contains all the circuits used in zkWallet.
To learn more about the zkSocialRecoveryWallet circuits, read the [README file](/packages/zkWallet-contracts/circuits/README.md) inside the `circuits` folder.

<a name="contracts"/>

### contracts

The [contracts folder](/packages/zkWallet-contracts/contracts) contains all the smart contracts used in zkWallet.

To learn more about the zkSocialRecoveryWallet smart contracts, read the [README file](/packages/zkWallet-contracts/contracts/README.md) inside the `contracts` folder.


<a name="guardian-service"/>

### guardian-service
The [guardian-service folder](/packages/guardian-service/) contains guardian backend build with [Nestjs](https://nestjs.com/).

<a name="guardian-ui"/>

### guardian-ui
The [guardian-ui folder](/packages/guardian-ui/) contains frontend build with [Next.js](https://nextjs.org/).

### zkWallet-ui
The [zkwallet-ui folder](/packages/zkwallet-ui/) contains frontend for wallet management build with [Next.js](https://nextjs.org/).


<a name="zk-structure"/>

## Zero Knowledge Structure

The following graphic shows the structure of the most important zero knowledge elements of the zkSocialRecoveryWallet project.

```text
├── packages
│   ├── zkWallet-contracts
│   │   ├── build
│   │   │   ├── snark-artifacts
│   │   │   │   ├── semaphore.wasm
│   │   │   │   ├── semaphore.zkey
│   │   ├── circuits
│   │   │   ├── semaphore.circom
│   │   │   ├── ecdh.circom
│   │   │   ├── tree
│   │   │   │   ├── hasherPoseidon.circom
│   │   │   │   ├── poseidon
│   │   │   │   │   ├── poseidonHashT3.circom
│   │   │   │   │   ├── poseidonHashT4.circom
│   │   │   │   │   ├── poseidonHashT5.circom
│   │   │   │   │   ├── poseidonHashT6.circom
│   │   ├── contracts
│   │   │   ├── contracts
│   │   │   │   ├── facets
│   │   │   │   │   ├── GuardianFacet.sol
│   │   │   │   │   ├── RecoveryFacet.sol
│   │   │   │   │   ├── SemaphoreFacet.sol
│   │   │   │   │   ├── SemaphoreGroupsFacet.sol
│   │   │   │   ├── recovery
│   │   │   │   │   ├── IRecovery.sol
│   │   │   │   │   ├── IRecoveryInternal.sol
│   │   │   │   │   ├── Recovery.sol
│   │   │   │   │   ├── RecoveryInternal.sol
│   │   │   │   │   ├── RecoveryStorage.sol
│   │   │   │   ├── semaphore
│   │   │   │   │   ├── base
│   │   │   │   │   │   ├── SemaphoreCoreBase
│   │   │   │   │   │   │   ├── ISemaphoreCoreBase.sol
│   │   │   │   │   │   │   ├── ISemaphoreCoreBaseInternal.sol
│   │   │   │   │   │   │   ├── SemaphoreCoreBase.sol
│   │   │   │   │   │   │   ├── SemaphoreCoreBaseInternal.sol
│   │   │   │   │   │   │   ├── SemaphoreCoreBaseMock.sol
│   │   │   │   │   │   │   ├── SemaphoreCoreBaseStorage.sol
│   │   │   │   │   │   ├── SemaphoreGroupsBase
│   │   │   │   │   │   │   ├── ISemaphoreGroupsBase.sol
│   │   │   │   │   │   │   ├── ISemaphoreGroupsInternal.sol
│   │   │   │   │   │   │   ├── SemaphoreGroupsBase.sol
│   │   │   │   │   │   │   ├── SemaphoreGroupsBaseInternal.sol
│   │   │   │   │   │   │   ├── SemaphoreGroupsBaseMock.sol
│   │   │   │   │   │   │   ├── SemaphoreGroupsBaseStorage.sol
│   │   │   │   │   ├── extensions
│   │   │   │   │   │   ├── SemaphoreVoting
│   │   │   │   │   │   │   ├── ISemaphoreVoting.sol
│   │   │   │   │   │   │   ├── ISemaphoreVotingInternal.sol
│   │   │   │   │   │   │   ├── SemaphoreVoting.sol
│   │   │   │   │   │   │   ├── SemaphoreVotingInternal.sol
│   │   │   │   │   │   │   ├── SemaphoreVotingStorage.sol
│   │   │   │   │   ├── ISemaphore.sol
│   │   │   │   │   ├── ISemaphoreGroups.sol
│   │   │   │   │   ├── ISemaphoreInternal.sol
│   │   │   │   │   ├── Semaphore.sol
│   │   │   │   │   ├── SemaphoreInternal.sol
│   │   │   │   │   ├── SemaphoreStorage.sol
│   │   │   │   ├── utils
│   │   │   │   │   ├── cryptography
│   │   │   │   │   │   ├── IncrementalBinaryTree
│   │   │   │   │   │   │   ├── IIncrementalBinaryTree.sol
│   │   │   │   │   │   │   ├── IIncrementalBinaryTreeInternal.sol
│   │   │   │   │   │   │   ├── IncrementalBinaryTreeInternal.sol
│   │   │   │   │   │   │   ├── IncrementalBinaryTreeStorage.sol
│   │   │   │   │   │   ├── Hashes.sol
│   │   │   │   │   ├── Constant.sol
│   │   │   │   ├── verifier
│   │   │   │   │   ├── Verifier20.sol
├── packages
│   ├── guardian-ui
│   │   ├── public
│   │   │   ├── semaphore.wasm
│   │   │   ├── semaphore_final.zkey
```

To see the full directory structure of the smart contracts, see [README file](/packages/zkWallet-contracts/contracts/README.md) inside the `contracts` folder.

<a name="run-locally"/>

## Run Locally

<a name="clone-the-repository"/>

### Clone the epository

```bash
git clone https://github.com/zkWallet/zkWallet-docs.git
```
<a name="get-submodules"/>

### Get the Submodules

```bash
git submodule update --init --recursive
```

<a name="run-circuits"/>

### Run circuits

To run cicuits, go inside the `contract` folder:

```bash
cd packages/zkWallet-contracts/
```

Run:

```bash
yarn compile:circuits
```

See also the follow the intructions in the [README file](/packages/zkWallet-contracts/contracts/circuits/README.md) in the  **`circuits`** folder.

<a name="run-contracts"/>

### Run contracts

To run contracts, go inside the **`contracts`** folder:

```bash
cd packages/zkWallet-contracts/
```

Then, follow the intructions in the [README file](/packages/zkWallet-contracts/README.md) in the `contracts` folder.

<a name="run-guardian-service"/>

### Run Guardian service

To run the Guardian Service, go inside the **`guardian-service`** folder:

```bash
cd packages/guardian-service
```

Then, follow the intructions in the [README file](/packages/guardian-service/contracts/README.md) in the `guardian-service` folder.

<a name="run-guardian-ui"/>

### Run Guardian ui

To run the Guardian Ui, go inside the **`guardian-ui`** folder:

```bash
cd packages/guardian-ui
```

Then, follow the intructions in the [README file](/packages/guardian-ui/README.md) in the `guardian-ui` folder.

<a name="deployed-contracts"/>

## Deployed contracts

<a name="deployed-devnet"/>

### Devnet

 Name                   | Harmony One Devnet                              | Version | Status   |
| :---                  | :---                                            |  :---   | :---     |
| Alice Wallet          | 0xf398d338297aF005e8Af3eA6aAfcAeBd4d5038A1      | 0.0.1   |          |
| Bob Wallet            | 0xd6c82C755368F56693fC7a5b11Bd9f6ae3619211      | 0.0.1   |          |
| Verifier16            | 0x8d2171b7dDC7F6B0f04657835a6b19F04B821353      | 1.0.0   | verified |
| Verifier17            | 0x64D87dC944Fb6956Ff09b4a1da2BfBFf4F78f976      | 1.0.0   | verified |
| Verifier18            | 0xa8007c877333508A07eA9f16CF2a2415057Ec0BC      | 1.0.0   | verified |
| Verifier19            | 0x434a85Da614E439929e7480464B18c1cea043B19      | 1.0.0   | verified |
| Verifier20            | 0x21176AA38497bdeab3CdB4368CFF53c428B001f7      | 1.0.0   | verified |
| Verifier21            | 0x48164EcB1CB426C72DCE0421F4426daE264724a7      | 1.0.0   | verified |
| Verifier22            | 0x50c3B09c4b478bd88D0ED14B056AcaFb679Bd345      | 1.0.0   | verified |
| Verifier23            | 0xbDc6e16F514e8d8098ABc90Ab40482d5CbD7c9Aa      | 1.0.0   | verified |
| Verifier24            | 0x6c3Ebf4d06e9595248fA7cc506ad87CffE445dee      | 1.0.0   | verified |
| Verifier25            | 0x6fCe423A2fDDe6788a27a99b59e0Cb40579CE988      | 1.0.0   | verified |
| Verifier26            | 0xCD81D914F1032F7d4c4AB088546c7d70A438cFC3      | 1.0.0   | verified |
| Verifier27            | 0x9eE3cC20Eb1e6a695192A9EDB0512694d31d81bf      | 1.0.0   | verified |
| Verifier28            | 0x18145aa8e5eb71959EaAC9Be2B383A070E80300A      | 1.0.0   | verified |
| Verifier29            | 0x5a6A9c1412179ef061CDF328E6b66BB8c5F337B6      | 1.0.0   | verified |
| Verifier30            | 0x532E815c80198b78512858F4cf125be4858c5e9A      | 1.0.0   | verified |
| Verifier31            | 0xf3a727DFC9eC0ca43934Aa5c9424EeB040C9Fb7c      | 1.0.0   | verified |
| Verifier32            | 0xaA048F0280d8F8B72f6E0Ed3D17E91ba1d1a8387      | 1.0.0   | verified |
| PoseidonT3            | 0x1fB6C0Cc4b846a4A25B41f3AAD59b4C719474E3F      | 1.0.0   |          |
| CountersFacet         | 0x02C1384694653D88919a1eF6EB8d3906a731640b      | 0.0.1   |          |
| ERC20Facet            | 0x5BA6985e2F04cA4Ef362dCFF0Ac793E1715F2E10      | 0.0.1   |          |
| ERC721Facet           | 0xEA33dC1D03A626C899dfA6bD9BA61AcEce886AF1      | 0.0.1   |          |
| GuardianFacet         | 0x3d4370D915c52E801963533E0aC54EAF4a57177b      | 0.0.1   |          |
| RecoveryFacet         | 0xC509433465D6e3b60CA192e81659BBEDffE7fd3b      | 0.0.1   |          |
| SemaphoreFacet        | 0xF6f822A0aaE0CDd6dDb6c0BA7284a74B006A0824      | 0.0.1   |          |
| SemaphoreGroupsFacet  | 0x7bA44FAF27B18d04Af0950f30617B5AAACceC038      | 0.0.1   |          |
| SemaphoreVotingFacet  | 0x43133D828f1E4c209eB60B4fEbD01221C72E4Ca4      | 0.0.1   |          |
| SimplicyWalletDiamond | 0x2338c67c687F3d6A28C27Df15325F06A6BF1C627      | 0.0.1   |          |
| WalletFactoryDiamond  | 0xb9853e45b3537975035159ece4E9F53EC18480Fd      | 0.0.1   |          |
| WalletFactoryFacet    | 0x58aD3eEFB1832ef971CDe8e45A7239E7dd8f9444      | 0.0.1   |          |

<a name="deployed-testet"/>

### Testnet

 Name                   | Harmony One Testnet                             | Version |
| :---                  | :---                                            |  :---   |
| Alice Wallet          |                                                 | 0.0.1   |
| Bob Wallet            |                                                 | 0.0.1   |
| Verifier16            | 0xEC4b5d3cB604D4771473F9E58394A45619C38BB2      | 1.0.0   |
| Verifier17            | 0x9BBCD95F0D06C6ff992E2baC305cC7C004101214      | 1.0.0   |
| Verifier18            | 0x4cEcE384bCB377EB7a3d39449a96478f5fa5fa72      | 1.0.0   |
| Verifier19            | 0x9671cBE0C930DC94085118dB0197e861abF74e02      | 1.0.0   |
| Verifier20            | 0x01F5C5E69AC1bcBC41Cef0a90A43A8e5A79cFb69      | 1.0.0   |
| Verifier21            | 0x8d2171b7dDC7F6B0f04657835a6b19F04B821353      | 1.0.0   |
| Verifier22            | 0x64D87dC944Fb6956Ff09b4a1da2BfBFf4F78f976      | 1.0.0   |
| Verifier23            | 0xa8007c877333508A07eA9f16CF2a2415057Ec0BC      | 1.0.0   |
| Verifier24            | 0x434a85Da614E439929e7480464B18c1cea043B19      | 1.0.0   |
| Verifier25            | 0x21176AA38497bdeab3CdB4368CFF53c428B001f7      | 1.0.0   |
| Verifier26            | 0x48164EcB1CB426C72DCE0421F4426daE264724a7      | 1.0.0   |
| Verifier27            | 0x50c3B09c4b478bd88D0ED14B056AcaFb679Bd345      | 1.0.0   |
| Verifier28            | 0xbDc6e16F514e8d8098ABc90Ab40482d5CbD7c9Aa      | 1.0.0   |
| Verifier29            | 0x6c3Ebf4d06e9595248fA7cc506ad87CffE445dee      | 1.0.0   |
| Verifier30            | 0x6fCe423A2fDDe6788a27a99b59e0Cb40579CE988      | 1.0.0   |
| Verifier31            | 0xCD81D914F1032F7d4c4AB088546c7d70A438cFC3      | 1.0.0   |
| Verifier32            | 0x9eE3cC20Eb1e6a695192A9EDB0512694d31d81bf      | 1.0.0   |
| PoseidonT3            |                                                 | 1.0.0   |
| ERC20Facet            |                                                 | 0.0.1   |
| ERC721Facet           |                                                 | 0.0.1   |
| GuardianFacet         |                                                 | 0.0.1   |
| RecoveryFacet         |                                                 | 0.0.1   |
| SemaphoreFacet        |                                                 | 0.0.1   |
| SemaphoreGroupsFacet  |                                                 | 0.0.1   |
| SemaphoreVotingFacet  |                                                 | 0.0.1   |
| SimplicyWalletDiamond |                                                 | 0.0.1   |
| WalletFactoryDiamond  |                                                 | 0.0.1   |
| WalletFactoryFacet    |                                                 | 0.0.1   |