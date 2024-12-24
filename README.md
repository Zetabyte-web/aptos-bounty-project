# Aptos NFT Marketplace dapp

## What is NFT?

NFT stands for Non-Fungible Token. It is a unique digital asset that represents ownership of a specific item or piece of content, such as art, music, videos, or collectibles.

## What is Aptos NFT Marketplace? 🌟

NFT Marketplace is an exciting use case deployed on the Aptos network that aims to help users create, list, purchase, and trade NFTs without any risk. Built on Move, the scalable programming language of Aptos, NFT Marketplace guarantees optimal speed and reliability for all your NFT business.

---

## 🔥 Project Overview

**Aptos NFT Marketplace** allows users to mint their own NFTs, put them up for sale, safely buy NFTs, and exchange tokens directly with other addresses. It is specifically intended to bring simplicity and general ease of use for people who are oriented to NFTs.

### Core Components

- **NavBar**: Some of the most important or frequently used sections of the marketplace are available on the navigation bar, such as the wallet connection and quick links enabling uninterrupted and seamless flow across the entire platform.

- **MarketView**: List of NFTs currently listed for sale, which can be sorted by scarcity or searched for an asset type. Users can also send NFTs directly from this view as well.

- **MyNFTs**: Offers opportunities to access applications to manage exemplary own NFT collections. For now, users can create a single listing of the NFTs they wish to sell or want others to bid on, sell NFTs to other users and purchase NFTs from other users, and finally, they can view the collection of the NFTs they have.

- **NFT Transfer**: NFTs can be sent to any wallet address securely. Found in the MarketView and NFTMarketplace categories making it easier to arrange to convey possessions based on users’ preferences.

- **Main App**: Composes orchestrate all fundamental interactions including das blockchain, wallets, and adds UI updates in real-time.

---

## Key Features

- **NFT Minting**: Construct highly customizable tokens as legal tender with essential information such as name, description, uri, and rarity besides aggregated metadata.
- **NFT Listing**: Offer NFTs on the market to make individual, fixed, and variable prices shortly.
- **NFT Purchase**: Safe and sound to purchase listed NFTs with a little marketplace commission.
- **NEW! NFT Transfer**: Transfer ownership of NFTs among different addresses without doing so through auction.
- **View Functions**: Here by checking an NFT you can view details of an NFT, the status of our marketplace and NFTs can be filtered by ownership, buy/sell status or rarity hierarchy.

---

## 🛠️ Setting Up the Developer Environment

### Prerequisites

- **Aptos CLI** installed.
- A deployed `NFTMarketplace` module on **Devnet**.
- **Petra Wallet** for testing.

### Steps to Get Started

1. **install Aptos CLI**:

   ```bash
   curl -fsSL "https://aptos.dev/scripts/install_cli.py" | python3
   ```

### Replace `your-marketplace-address`

Update the placeholder `your-marketplace-address` in the following files with your actual marketplace address:

- **NFTMarketplace.move** at **line 1**.
- **move.toml** at **line 6**.
- **MarketView.tsx** at **line 78**.
- **MyNFTs.tsx** at **line 38**.
- **App.tsx** at **line 15**.



2. **Initialize Aptos in a new terminal**:

   ```bash
   cd aptos4-q1-bounty
   cd contracts
   aptos init
   aptos move publish
   ```

3. **Clone the Repository in another new terminal**:

   ```bash
   cd aptos4-q2-bounty
   ```

4. **Install other Aptos CLI**:

   ```bash
   curl -fsSL https://fnm.vercel.app/install | bash
   source ~/.bashrc
   fnm install --lts
   ```
5. **Install Dependencies**:

   ```bash
   npm i @aptos-labs/wallet-adapter-react @aptos-labs/wallet-adapter-ant-design petra-plugin-wallet-adapter --legacy-peer-deps
   ```

6. **Install React Router**:

   ```bash
   npm install react-router-dom --legacy-peer-deps
   ```


7. **Start the Project**:
   ```bash
   npm start
   ```

---

## 🆕 Introducing: NFT Transfer Feature

#### What is NFT Transfer?

The `transfer_nft` function allows users to securely transfer ownership of an NFT to another address. Whether gifting an NFT or moving it between wallets, this feature eliminates the need for a sale process.

#### How NFT Transfer Works

1. **Verify Ownership**: Ensure the caller is the current owner of the NFT.
2. **Check for Transfer to Same Owner**: Prevent the transfer if the recipient is the current owner.
3. **Update Ownership**: Transfer ownership to the specified new address.
4. **Reset Sale Status**: Automatically reset the sale status and price of the NFT, so it’s no longer listed for sale after the transfer.

---

## 📜 NFT Transfer Code in `NFTMarketplace.move`

```move
public entry fun transfer_nft(
    account: &signer,
    marketplace_addr: address,
    nft_id: u64,
    new_owner: address
) acquires Marketplace {
    let marketplace = borrow_global_mut<Marketplace>(marketplace_addr);
    let nft_ref = vector::borrow_mut(&mut marketplace.nfts, nft_id);

    // Ensure the caller is the owner of the NFT
    assert!(nft_ref.owner == signer::address_of(account), 300);

    // Prevent transfer to the same owner
    assert!(nft_ref.owner != new_owner, 301);

    // Update NFT ownership and reset sale status and price
    nft_ref.owner = new_owner;
    nft_ref.for_sale = false;
    nft_ref.price = 0;
}
```

### Function Logic

1. **Borrowing the Marketplace Resource**:

   ```move
   let marketplace = borrow_global_mut<Marketplace>(marketplace_addr);
   ```

   Accesses the `Marketplace` resource located at `marketplace_addr` to modify its data.

2. **Accessing the NFT**:

   ```move
   let nft_ref = vector::borrow_mut(&mut marketplace.nfts, nft_id);
   ```

   Retrieves a mutable reference to the NFT in the marketplace.

3. **Ensuring the Caller is the Owner**:

   ```move
   assert!(nft_ref.owner == signer::address_of(account), 300);
   ```

   Validates that the caller owns the NFT.

4. **Preventing Transfer to the Same Owner**:

   ```move
   assert!(nft_ref.owner != new_owner, 301);
   ```

   Ensures the recipient address is not the same as the current owner.

5. **Updating NFT Ownership and Sale Status**:
   ```move
   nft_ref.owner = new_owner;
   nft_ref.for_sale = false;
   nft_ref.price = 0;
   ```
   Updates the ownership, sale status, and price of the NFT.

---

## 🔑 MarketView: The Transfer Process in TypeScript

### Code Implementation in `MarketView.tsx`

```typescript
const handleTransferClick = (nft: NFT) => {
  setSelectedNft(nft);
  setIsTransferModalVisible(true);
};

const handleCancelTransfer = () => {
  setIsTransferModalVisible(false);
  setSelectedNft(null);
  setRecipientAddress(""); // Clear the recipient address
};

const handleConfirmTransfer = async () => {
  if (!selectedNft || !recipientAddress) return;

  try {
    const entryFunctionPayload = {
      type: "entry_function_payload",
      function: `${marketplaceAddr}::NFTMarketplace::transfer_nft`,
      type_arguments: [],
      arguments: [
        marketplaceAddr, // Marketplace address
        selectedNft.id.toString(), // NFT ID
        recipientAddress, // New owner's address
      ],
      data: {}, // Add this line to include the data property
    };

    const response = await (window as any).aptos.signAndSubmitTransaction(
      entryFunctionPayload
    );
    await client.waitForTransaction(response.hash);

    message.success("NFT transferred successfully!");
    setIsTransferModalVisible(false);
    handleFetchNfts(rarity === "all" ? undefined : rarity); // Refresh NFT list
  } catch (error) {
    console.error("Error transferring NFT:", error);
    message.error("Failed to transfer NFT.");
  }
};
```

### **1. `handleTransferClick`**

Triggered when the user clicks to transfer an NFT.

```typescript
const handleTransferClick = (nft: NFT) => {
  setSelectedNft(nft);
  setIsTransferModalVisible(true);
};
```

- **Purpose**: Sets the selected NFT for transfer and displays the transfer modal.
- **Parameters**:
  - `nft`: The NFT object selected for transfer.

---

### **2. `handleCancelTransfer`**

Handles the cancellation of the transfer process.

```typescript
const handleCancelTransfer = () => {
  setIsTransferModalVisible(false);
  setSelectedNft(null);
  setRecipientAddress("");
};
```

- **Purpose**: Resets the transfer process by hiding the modal, clearing the selected NFT, and resetting the recipient address.

---

### **3. `handleConfirmTransfer`**

Executes the actual transfer of the NFT on the blockchain.

#### **Steps**

#### **1. Initial Validation**

```typescript
if (!selectedNft || !recipientAddress) return;
```

- **Purpose**: Ensures that:
  - An NFT has been selected (`selectedNft` is not null).
  - A valid recipient address (`recipientAddress`) is provided.
- **Action**: If either condition fails, the function exits early without executing further.

---

#### **2. Constructing the Transaction Payload**

```typescript
const entryFunctionPayload = {
  type: "entry_function_payload",
  function: `${marketplaceAddr}::NFTMarketplace::transfer_nft`,
  type_arguments: [],
  arguments: [marketplaceAddr, selectedNft.id.toString(), recipientAddress],
};
```

- **`type`**: Specifies the payload type, which is "entry_function_payload".
- **`function`**: The fully qualified name of the Move function to execute. It includes:
  - `marketplaceAddr`: The blockchain address of the deployed NFT Marketplace.
  - `transfer_nft`: The specific function to call.
- **`type_arguments`**: An empty array since this function doesn't use generic type parameters.
- **`arguments`**: A list of parameters required by the Move function:
  1. `marketplaceAddr`: The address of the NFT Marketplace contract.
  2. `selectedNft.id.toString()`: The ID of the NFT being transferred, converted to a string.
  3. `recipientAddress`: The blockchain address of the new owner.

---

#### **3. Submitting the Transaction**

```typescript
const response = await(window as any).aptos.signAndSubmitTransaction(
  entryFunctionPayload
);
```

- **Purpose**: Sends the transaction payload to the Aptos blockchain using the Aptos wallet adapter.
- **`signAndSubmitTransaction`**:
  - Prompts the connected wallet (e.g., Petra) to sign the transaction.
  - Submits the signed transaction to the blockchain.
- **Response**: Contains details about the submitted transaction, including a unique transaction hash.

---

#### **4. Waiting for Confirmation**

```typescript
await client.waitForTransaction(response.hash);
```

- **Purpose**: Waits for the blockchain to confirm the transaction.
- **`response.hash`**: The unique hash of the transaction used to track its status.

---

#### **5. Success Feedback**

```typescript
message.success("NFT transferred successfully!");
setIsTransferModalVisible(false);
handleFetchNfts(rarity === "all" ? undefined : rarity);
```

- **`message.success`**: Displays a success notification to the user.
- **`setIsTransferModalVisible(false)`**: Closes the transfer modal.
- **`handleFetchNfts`**: Refreshes the NFT list:
  - If `rarity` is "all", it fetches all NFTs.
  - Otherwise, it filters the list by the specified rarity.

---

#### **6. Error Handling**

```typescript
} catch (error) {
  console.error("Error transferring NFT:", error);
  message.error("Failed to transfer NFT.");
}
```

- **`catch (error)`**: Captures any errors that occur during the transaction process.
- **Logging**: Outputs the error details to the console for debugging.
- **User Feedback**: Displays an error notification.

---

## 🔎 Testing and Validating Marketplace Features

#### Checklist

- **Connect Wallet**: Check the connection of wallet and view wallet address and balance at the same time.
- **Mint NFTs**: Begin testing and check the NFTs are mintage and residing on ones collection.
- **List NFTs for Sale**: Check whether the NFTs listed in the marketplace are present at the correct price as mentioned against each of them.
- **Purchase NFTs**: Buying non fungible tokens should be tested and all transactions should be seamless.
- **Transfer NFTs**: Transfer the NFTs you’d like to another wallet address and then, using the Ownership History tab, verify the change.

---

## Transfer NFTs

#### 1. Open the Marketplace

- Navigate to the **Marketplace** tab.
- View all NFTs you own or those listed in the marketplace.

#### 2. Select NFT to Transfer:

- Browse through the NFTs in your collection and find the one you wish to transfer.
- Click the **"Transfer"** button under the selected NFT to open the transfer details.

#### 3. Enter Recipient’s Address:

- A pop-up window will appear where you are prompted to enter the recipient’s address (the address to which the NFT will be transferred).
- Ensure that the address entered is valid.

#### 4. Review Transfer Details:

- In the pop-up, review the details of the NFT you are transferring (e.g., its name, description, rarity, and the recipient's address).
- Confirm that the transfer details are correct.

#### 5. Approve Transaction in Wallet:

- A **signature request** will appear in your connected wallet (such as Petra wallet).
- Review the transaction details and click **"Approve"** to confirm the transfer.
- The wallet will facilitate the transfer of the NFT to the recipient’s address.

#### 6. Transfer Confirmation:

- Once the transaction is approved and confirmed by the blockchain, a message will appear stating: **"NFT transferred successfully!"**.
- The NFT will no longer be in your collection and will now be in the recipient’s collection.

---

## 🧑‍💻 Final Thoughts

The NFT Transfer feature introduces new opportunities for users on how to use NFTs securely and conveniently. Whether users want to gift an NFT or navigate different wallets for a collection, this feature brings convenience to the platform.
