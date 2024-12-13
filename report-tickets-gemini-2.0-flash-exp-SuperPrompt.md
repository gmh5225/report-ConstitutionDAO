# Smart Contract Security Analysis Report


## About

The `Tickets` contract is a simple ERC20 token contract with minting and burning capabilities controlled by the contract owner. It inherits from `ERC20`, `ERC20Permit`, and `Ownable` from the OpenZeppelin library. The contract allows the owner to mint new tokens to an account and burn existing tokens from an account. It implements the `ITickets` interface, which defines the `print` (mint) and `redeem` (burn) functions.

## Findings Severity breakdown
- Critical: 0
- High: 0
- Medium: 0
- Low: 2
- Gas: 1

---

### Missing Event Emissions for Token Operations
- **Title:** Missing Event Emissions for Token Operations
- **Severity:** Low
- **Description:** The `print` and `redeem` functions perform token minting and burning, respectively, but do not emit any events specific to these actions. While the inherited ERC20 logic emits `Transfer` events on mint and burn, custom events for these owner-controlled operations would improve the transparency and ease of indexing of token events.
- **Impact:** Difficult to trace specific owner-controlled minting and burning operations.
- **Location:** Tickets.sol:11 & 19
- **Recommendation:** Emit custom events within the `print` and `redeem` functions, such as `Minted(address indexed to, uint256 amount)` and `Burned(address indexed from, uint256 amount)`, before returning. This ensures detailed tracking of these operations.

---

### Function Visibility for Mint and Burn
- **Title:** Function Visibility for Mint and Burn
- **Severity:** Low
- **Description:** The `print` (mint) and `redeem` (burn) functions are declared as `external`, which is technically correct since they are intended to be called externally. However, because they are restricted to `onlyOwner`, marking them `public` would not pose any security issue and would actually save a small amount of gas on external call.
- **Impact:** Minimal, but it's better practice to use the most restrictive modifier while not having any effect on security.
- **Location:** Tickets.sol:11 & 19
- **Recommendation:** Change the visibility of `print` and `redeem` from `external` to `public`, since the `onlyOwner` modifier already controls their accessibility. This minor adjustment can slightly reduce gas consumption.

---
### Unnecessary `return` keyword
- **Title:** Unnecessary `return` keyword
- **Severity:** Gas
- **Description:** The `return` keyword is used before calling `_mint` and `_burn` functions. The `_mint` and `_burn` functions are `internal` functions and does not return any value so the `return` is not necessary.
- **Impact:** A very small amount of gas is wasted on each call because of the unnecessary `return` keyword.
- **Location:** Tickets.sol:14 & 22
- **Recommendation:** Remove the `return` keyword from `print` and `redeem` functions to save gas.

---

## Detailed Analysis

### Architecture
The `Tickets` contract uses inheritance from `ERC20`, `ERC20Permit`, and `Ownable` contracts provided by OpenZeppelin. This approach leverages established, secure implementations for core functionalities such as token transfers, approvals, permits, and owner management. The contract provides a constructor to set the token name and symbol. It then introduces custom functionality for owner-controlled token minting (`print`) and burning (`redeem`).

### Code Quality
The contract adheres to basic best practices, such as using OpenZeppelin for standard token functionalities. It uses the `onlyOwner` modifier to restrict the minting and burning functions. However, the `print` and `redeem` functions could include additional event emissions. The overall readability and structure of the code is good and it is easy to understand.

### Centralization Risks
The contract's minting and burning functions are entirely controlled by the contract owner, which is a significant point of centralization. There is no mechanism for the community to control the total supply of token other than the single owner. The contract owner holds full authority to increase and decrease supply, thus holding a lot of power over the economic aspect of the token.

### Systemic Risks
The contract has minimal external dependencies, relying mostly on standard OpenZeppelin contracts. It does not interact with any external protocols, so composability risks are low. However, as a standard ERC20, the contract is susceptible to common ERC20 vulnerabilities if implemented incorrectly. This contract has a basic implementation, so that risk is low.

### Testing & Verification
The contract code is minimal, making it easier to verify its correctness. It can be tested with a suite of test cases covering the owner-controlled minting and burning functionalities, and standard ERC20 functions. Unit tests should be written to verify functionality.

## Final Recommendations

1. **Emit Custom Events:** Implement `Minted` and `Burned` events in the `print` and `redeem` functions respectively, before returning.
2. **Function Visibility:** Change function visibility of `print` and `redeem` from `external` to `public`.
3. **Remove Unnecessary return:** Remove the unnecessary `return` keyword in the `print` and `redeem` functions.

## Improved Code with Security Comments
```solidity
// File: Tickets.sol
// SPDX-License-Identifier: MIT
pragma solidity 0.8.6;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

interface ITickets {
    function print(address _account, uint256 _amount) external;
    function redeem(address _account, uint256 _amount) external;
}

contract Tickets is ERC20, ERC20Permit, Ownable, ITickets {
    // Custom event for minting tokens.
    event Minted(address indexed to, uint256 amount);
    // Custom event for burning tokens.
    event Burned(address indexed from, uint256 amount);

    constructor(string memory _name, string memory _symbol)
        ERC20(_name, _symbol)
        ERC20Permit(_name)
    {}

    // Changed visibility to public for slight gas optimization.
    function print(address _account, uint256 _amount)
        public
        override
        onlyOwner
    {
        // Minting tokens to the specified account.
        _mint(_account, _amount);
        // Emit a custom event for minting action for better tracking.
        emit Minted(_account, _amount);
    }

   // Changed visibility to public for slight gas optimization.
    function redeem(address _account, uint256 _amount)
        public
        override
        onlyOwner
    {
         // Burning tokens from the specified account.
        _burn(_account, _amount);
        // Emit a custom event for burning action for better tracking.
        emit Burned(_account, _amount);
    }
}
```
