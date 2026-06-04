###  `PasswordStore::setPassword` lacks access control, allowing any user to overwrite the owner's password

**Description:** The `PasswordStore::setPassword` function is intended to allow only the contract owner to set a new password. However, the function has no access control modifier (such as `onlyOwner`) or manual `msg.sender` check. As a result, any external caller can invoke `setPassword` and overwrite the owner's password.

**Impact:** Any malicious user can overwrite the owner's stored password, completely breaking the protocol's core guarantee of owner-only password management. The contract becomes useless for its intended purpose of private password storage, and the owner loses control over their own stored data.

**Proof of Concept:**
```solidity
function test_anyoneCanSetPassword() public {
    vm.prank(address(1)); // non-owner caller
    passwordStore.setPassword("hacked");

    vm.prank(owner);
    string memory actual = passwordStore.getPassword();
    assertEq(actual, "hacked");
}
```

**Recommended Mitigation:** Add an access control check to restrict `setPassword` to the contract owner only:

```solidity
function setPassword(string memory newPassword) external {
    if (msg.sender != s_owner) {
        revert PasswordStore__NotOwner();
    }
    s_password = newPassword;
    emit SetNetPassword();
}
```