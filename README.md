# AVAXproof

This Solidity code consists of two integrated smart contracts: an ERC20 token named "Solidity by Example" (symbol: SOLBYEX) and a Vault for managing token deposits and withdrawals. The ERC20 contract provides basic functionalities such as transferring tokens, approving allowances, transferring on behalf of an owner, minting new tokens, and burning tokens, while also emitting events for transfers and approvals. The Vault contract, interacting with the ERC20 token via its address, allows users to deposit tokens in exchange for shares representing their stake. The number of shares minted during deposit or the number of tokens received during withdrawal is calculated proportionally to the Vault's current balance and total supply. Private _mint and _burn functions manage the issuance and redemption of shares, ensuring accurate updates to the Vault's total supply and user balances. The Vault ensures compliance with standard ERC20 functions and events through the IERC20 interface, maintaining seamless and secure token interactions. The integrated design allows users to efficiently manage their token holdings while leveraging the Vault for secure and proportional asset management.

## Getting Started

### Installing

To interact with `token.sol` and `Vault.sol`.

1.**Set up Ubuntu**

- Download Ubuntu's required version.
- Install packages to run the desired command in Ubuntu.

2.**Metamask**

  - Install Metamask for transaction in Code.
  - add local network which you will created on ubuntu.
  - After it add Token

3.**Set up Remix**

   - visit [Remix]
   - Create two files name `Vault.sol` and `token.sol`.
   - insert the code provided in your project.
   - compile and deploy the code .

     ## Contract Code
      **Token**
     
This Solidity smart contract implements an ERC20 token named "Solidity by Example" (symbol: SOLBYEX) with standard functionalities such as transferring tokens, approving allowances, and transferring tokens on behalf of an owner. It also includes minting and burning functions to increase and decrease the total token supply.
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract ERC20 {
    uint public totalSupply;
    mapping(address => uint) public balanceOf;
    mapping(address => mapping(address => uint)) public allowance;
    string public name = "Solidity by Example";
    string public symbol = "SOLBYEX";
    uint8 public decimals = 18;

		event Transfer(address indexed from, address indexed to, uint value);
    event Approval(address indexed owner, address indexed spender, uint value);

    function transfer(address recipient, uint amount) external returns (bool) {
        balanceOf[msg.sender] -= amount;
        balanceOf[recipient] += amount;
        emit Transfer(msg.sender, recipient, amount);
        return true;
    }

    function approve(address spender, uint amount) external returns (bool) {
        allowance[msg.sender][spender] = amount;
        emit Approval(msg.sender, spender, amount);
        return true;
    }

    function transferFrom(
        address sender,
        address recipient,
        uint amount
    ) external returns (bool) {
        allowance[sender][msg.sender] -= amount;
        balanceOf[sender] -= amount;
        balanceOf[recipient] += amount;
        emit Transfer(sender, recipient, amount);
        return true;
    }

    function mint(uint amount) external {
        balanceOf[msg.sender] += amount;
        totalSupply += amount;
        emit Transfer(address(0), msg.sender, amount);
    }

    function burn(uint amount) external {
        balanceOf[msg.sender] -= amount;
        totalSupply -= amount;
        emit Transfer(msg.sender, address(0), amount);
    }
}
```
**Vault**

This Solidity contract, Vault, allows users to deposit and withdraw ERC20 tokens. When depositing, users receive shares proportional to their contribution relative to the vault's total token balance. When withdrawing, users receive an amount of tokens proportional to their shares in the vault. The contract also mints and burns shares to reflect deposits and withdrawals.
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

interface IERC20 {
    function totalSupply() external view returns (uint);

    function balanceOf(address account) external view returns (uint);

    function transfer(address recipient, uint amount) external returns (bool);

    function allowance(address owner, address spender) external view returns (uint);

    function approve(address spender, uint amount) external returns (bool);

    function transferFrom(
        address sender,
        address recipient,
        uint amount
    ) external returns (bool);

    event Transfer(address indexed from, address indexed to, uint value);
    event Approval(address indexed owner, address indexed spender, uint value);
}

contract Vault {
    IERC20 public immutable token;

    uint public totalSupply;
    mapping(address => uint) public balanceOf;

    constructor(address _token) {
        token = IERC20(_token);
    }

    function _mint(address _to, uint _shares) private {
        totalSupply += _shares;
        balanceOf[_to] += _shares;
    }

    function _burn(address _from, uint _shares) private {
        totalSupply -= _shares;
        balanceOf[_from] -= _shares;
    }

    function deposit(uint _amount) external {
        /*
        a = amount
        B = balance of token before deposit
        T = total supply
        s = shares to mint

        (T + s) / T = (a + B) / B 

        s = aT / B
        */
        uint shares;
        if (totalSupply == 0) {
            shares = _amount;
        } else {
            shares = (_amount * totalSupply) / token.balanceOf(address(this));
        }

        _mint(msg.sender, shares);
        token.transferFrom(msg.sender, address(this), _amount);
    }

    function withdraw(uint _shares) external {
        /*
        a = amount
        B = balance of token before withdraw
        T = total supply
        s = shares to burn

        (T - s) / T = (B - a) / B 

        a = sB / T
        */
        uint amount = (_shares * token.balanceOf(address(this))) / totalSupply;
        _burn(msg.sender, _shares);
        token.transfer(msg.sender, amount);
    }
}
```
## Help
For common issues or problems, ensure you have the following:

- Sufficient balance for transfer
- ensure sufficient allowance so user can approved the contract to spend token.
- Ensure your contract  is optimised for gas usage.
-  Ensure the mint and burn functions update totalSupply and balanceOf accurately.
## Author 
Akash Singh

## License
This project is licensed under the MIT License

 
