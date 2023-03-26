### Issues Template
| Letter | Name | Description |
|:--:|:-------:|:-------:|
| L  | Low risk | Potential risk |
| NC |  Non-critical | Non risky findings |
| R  | Refactor | Code changes |
| O | Ordinary | Commonly found issues |

| Total Found Issues | 20 |
|:--:|:--:|

### Low Risk Template
| Count | Title | Instances |
|:--:|:-------|:--:|
| [L-01] | Critical address changes should use 2 step procedure | 4 |
| [L-02] | Remove unnecessary `receive() external payable`| 4 |

| Total Low Risk Issues | 2 |
|:--:|:--:|

### Non-Critical Template
| Count | Title | Instances |
|:--:|:-------|:--:|
| [N-01] | Implement checks for input validation for extra safety in setters | 7 |
| [N-02] | Missing event for critical parameters init and change | 3 |
| [N-03] | Omission of important parameters in events emitted | 5 |
| [N-04] | Missing zero-address checks for `initialize` functions | 4 |
| [N-05] |Upgradeable contract is missing a `__gap[50]` storage variable to allow for new storage variables in later versions | 4 |
| [N-06] | Insufficient Coverage | 3 |

| Total Non-Critical Issues | 6 |
|:--:|:--:|

### Refactor Issues Template
| Count | Title | Instances |
|:--:|:-------|:--:|
| [R-01] | Upgradeable contract is missing a `__gap[50]` storage variable to allow for new storage variables in later versions | 1 |
| [R-02] | Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18) | 23 |
| [R-03] | Use `string.concat() ` instead of `abi.encodePacked()` | 6 |
| [R-04] | String constants used in multiple places should be defined as constants | 6 |
| [R-05] | Use ternary operators to shorten if/else statements | 2 |
| [R-06] | Indent logic in for loops in code blocks {} correctly | 2 |
| [R-07] | Should not allow `_weight` parameter to be set as zero | 2 |

| Total Refactor Issues | 7 |
|:--:|:--:|

### Ordinary Issues 
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [O-01] | Unlocked Pragma | 4 |
| [O-02] | For mordern and more readable code, update import usages | 31 |
| [O-03] | Add missing NatSpec comments such as return and parame | 20 |
| [O-04] | Order of function does not comply with the solidity style guide | 9 |
| [O-05] | Function naming suggestion of internal and private functions starts with _ | 4 |

| Total Ordinary Issues | 5 |
|:--:|:--:|


## [L-01] Critical address changes should use 2 step procedure
```solidity
4 results - 4 files

/WstEth.sol
33:    function initialize(address _owner) external initializer {
34:        _transferOwnership(_owner);
35:        maxSlippage = (1 * 10 ** 16); // 1%
36:    }

/SfrxEth.sol
36:    function initialize(address _owner) external initializer {
37:        _transferOwnership(_owner);
38:        maxSlippage = (1 * 10 ** 16); // 1%
39:    }

/SafEth.sol
48:    function initialize(
49:        string memory _tokenName,
50:        string memory _tokenSymbol
51:    ) external initializer {
52:        ERC20Upgradeable.__ERC20_init(_tokenName, _tokenSymbol);
53:        _transferOwnership(msg.sender);
54:        minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum
55:        maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum
56:    }

/Reth.sol
42:    function initialize(address _owner) external initializer {
43:        _transferOwnership(_owner);
44:        maxSlippage = (1 * 10 ** 16); // 1%
45:    }
```

Description:
Lack of two-step procedure for critical operations leaves them error-prone. Consider adding a two-step pattern on critical changes to avoid mistakenly transferring ownership of roles or critical functionalities to the wrong address. This allows us to recover from mistakes in first step since second step can never be done if incorrect address is used in first step

Recommendation:
Move away from single step change and instead use a two step change, 
1.  First steps grants/approve a new address as being the changed address
2.  Second step is a tx from the new address that claims itself as being the new address

## [L-02] Remove unnecessary `receive() external payable`
```solidity
4 results - 4 files

/WstEth.sol
97: receive() external payable {}

/SfrxEth.sol
126: receive() external payable {}

/SafEth.sol
246: receive() external payable {}

/Reth.sol
245: receive() external payable {}
```

Description:
All the functionalities of the contracts such as the main entry and exit point has been met and as such the additional `receive() external payable` function is not required as an entry point. This saves gas on deployment and also prevents users from unknowingly sending ETH leading to unnecessary extra funds received by owner of this contracts 

In `/WstEth.sol`, `/SfrxEth.sol` and `/Reth.sol`:
Entry: `deposit()`
Exit: `withdraw()`

In `/SafEth.sol`
Entry: `stake()`
Exit: `unstake()`

Recommendation:
Remove `receive() external payable` functions


## [N-01] Implement checks for input validation for extra safety in setters
```solidity
7 results - 4 files

/WstEth.sol
48:    function setMaxSlippage(uint256 _slippage) external onlyOwner {
49:        maxSlippage = _slippage;
50:    }

/SfrxEth.sol
51:    function setMaxSlippage(uint256 _slippage) external onlyOwner {
52:        maxSlippage = _slippage;
53:    }

/SafEth.sol
214:    function setMinAmount(uint256 _minAmount) external onlyOwner {
215:        minAmount = _minAmount;
216:        emit ChangeMinAmount(minAmount);
217:    }

223:    function setMaxAmount(uint256 _maxAmount) external onlyOwner {
224:        maxAmount = _maxAmount;
225:        emit ChangeMaxAmount(maxAmount);
226:    }

232:    function setPauseStaking(bool _pause) external onlyOwner {
233:        pauseStaking = _pause;
234:        emit StakingPaused(pauseStaking);
235:    }

241:    function setPauseUnstaking(bool _pause) external onlyOwner {
242:        pauseUnstaking = _pause;
243:        emit UnstakingPaused(pauseUnstaking);
244:    }

/Reth.sol
58:    function setMaxSlippage(uint256 _slippage) external onlyOwner {
59:        maxSlippage = _slippage;
60:    }
```

Description:
In the functions, checking whether the current value and the new value are the same could be added.

## [N-02] Missing event for critical parameters init and change
```solidity
3 results - 3 files

/WstEth.sol
48:    function setMaxSlippage(uint256 _slippage) external onlyOwner {
49:        maxSlippage = _slippage;
50:    }

/SfrxEth.sol
51:    function setMaxSlippage(uint256 _slippage) external onlyOwner {
52:        maxSlippage = _slippage;
53:    }

/Reth.sol
58:    function setMaxSlippage(uint256 _slippage) external onlyOwner {
59:        maxSlippage = _slippage;
60:    }
```

Description:
Events help non-contract tools to track changes, and events prevent users from being surprised by changes

Recommendation:
Add Event-Emit

## [N-03] Omission of important parameters in events emitted
```solidity
5 results - 1 file

/SafEth.sol
202:    function setMaxSlippage(
203:        uint _derivativeIndex,
204:        uint _slippage
205:    ) external onlyOwner {
206:        derivatives[_derivativeIndex].setMaxSlippage(_slippage);
207:        emit SetMaxSlippage(_derivativeIndex, _slippage);
208:    }

214:    function setMinAmount(uint256 _minAmount) external onlyOwner {
215:        minAmount = _minAmount;
216:        emit ChangeMinAmount(minAmount);
217:    }

223:    function setMaxAmount(uint256 _maxAmount) external onlyOwner {
224:        maxAmount = _maxAmount;
225:        emit ChangeMaxAmount(maxAmount);
226:    }

232:    function setPauseStaking(bool _pause) external onlyOwner {
233:        pauseStaking = _pause;
234:        emit StakingPaused(pauseStaking);
235:    }

241:    function setPauseUnstaking(bool _pause) external onlyOwner {
242:        pauseUnstaking = _pause;
243:        emit UnstakingPaused(pauseUnstaking);
244:    }
```

Description
Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters.

Recommendation:
The events should include the new value and old value where possible

## [N-04] Missing zero-address checks for `initialize` functions
```solidity
4 results - 4 files

/WstEth.sol
33:    function initialize(address _owner) external initializer {
34:        _transferOwnership(_owner);
35:        maxSlippage = (1 * 10 ** 16); // 1%
36:    }

/SfrxEth.sol
36:    function initialize(address _owner) external initializer {
37:        _transferOwnership(_owner);
38:        maxSlippage = (1 * 10 ** 16); // 1%
39:    }

/SafEth.sol
48:    function initialize(
49:        string memory _tokenName,
50:        string memory _tokenSymbol
51:    ) external initializer {
52:        ERC20Upgradeable.__ERC20_init(_tokenName, _tokenSymbol);
53:        _transferOwnership(msg.sender);
54:        minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum
55:        maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum
56:    }

/Reth.sol
42:    function initialize(address _owner) external initializer {
43:        _transferOwnership(_owner);
44:        maxSlippage = (1 * 10 ** 16); // 1%
45:    }
```
Description:
Address zero check can be added in the functions to ensure the new owner's addresses are not mistakenly transferred to address(0).

## [N-05] Upgradeable contract is missing a `__gap[50]` storage variable to allow for new storage variables in later versions
```solidity
4 results - 4 files

/WstEth.sol
12: contract WstEth is IDerivative, Initializable, OwnableUpgradeable 

/SfrxEth.sol
13: contract SfrxEth is IDerivative, Initializable, OwnableUpgradeable

/SafEth.sol
15: contract SafEth is
16:    Initializable,
17:    ERC20Upgradeable,
18:    OwnableUpgradeable,
19:    SafEthStorage

/Reth.sol
19: contract Reth is IDerivative, Initializable, OwnableUpgradeable
```

See this [link](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps) for a description of this storage variable. While some contracts may not currently be sub-classed, adding the variable now protects against forgetting to add it in the future.

## [N-06] Insufficient Coverage
The test coverage of the project is 98.06%. 

Due to its capacity, test coverage is expected to be 100%. Ensure the contracts `WstEth.sol`, `SfrxEth.sol` and `Reth.sol` have 100% coverage


## [R-01] Overly complex calculation
1 result - 1 file

```solidity
212:    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
213:        if (poolCanDeposit(_amount))
214:            return
215:                RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);
216:        else return (poolPrice() * 10 ** 18) / (10 ** 18); 
217:    }
```

The above instance can be refactored to:

```solidity
212:    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
213:        if (poolCanDeposit(_amount))
214:            return
215:                RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);
216:        else return poolPrice() 
217:    }
```

## [R-02] Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)
```solidity
23 results - 4 files

/WstEth.sol
33:    function initialize(address _owner) external initializer
35:        maxSlippage = (1 * 10 ** 16); // 1%

56:    function withdraw(uint256 _amount) external onlyOwner
60:        uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;

86:    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
87:        return IWStETH(WST_ETH).getStETHByWstETH(10 ** 18);

/SfrxEth.sol
36:    function initialize(address _owner) external initializer 
38:        maxSlippage = (1 * 10 ** 16); // 1%

60:    function withdraw(uint256 _amount) external onlyOwner 
74:        uint256 minOut = (((ethPerDerivative(_amount) * _amount) / 10 ** 18) *
75:            (10 ** 18 - maxSlippage)) / 10 ** 18;

111:    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
112:        uint256 frxAmount = IsFrxEth(SFRX_ETH_ADDRESS).convertToAssets(
113:            10 ** 18
114:        );
115:        return ((10 ** 18 * frxAmount) /

/SafEth.sol
48:    function initialize
54:        minAmount = 5 * 10 ** 17; // initializing with .5 ETH as minimum
55:        maxAmount = 200 * 10 ** 18; // initializing with 200 ETH as maximum

63:    function stake() external payable
73:                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
74:                    derivatives[i].balance()) /
75:                10 ** 18;

80:            preDepositPrice = 10 ** 18; // initializes with a price of 1
81:        else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;

92:            uint derivativeReceivedEthValue = (derivative.ethPerDerivative(
93:                depositAmount
94:            ) * depositAmount) / 10 ** 18;

98:        uint256 mintAmount = (totalStakeValueEth * 10 ** 18) / preDepositPrice;

/Reth.sol
42:    function initialize(address _owner) external initializer
44:        maxSlippage = (1 * 10 ** 16); // 1%

156:    function deposit() external payable onlyOwner returns (uint256)
172:            uint rethPerEth = (10 ** 36) / poolPrice();
173:            
174:            uint256 minOut = ((((rethPerEth * msg.value) / 10 ** 18) *
175:                ((10 ** 18 - maxSlippage))) / 10 ** 18);

212:    function ethPerDerivative(uint256 _amount) public view returns (uint256)
215:                RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);
```

Description:
While the compiler knows how to optimize away the exponentiation, it is still better coding practice to use idioms that do not require compiler optimization, if they exist.

Reccomendation:
Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)

## [R-03] Use `string.concat() ` instead of `abi.encodePacked()`
```solidity
6 results - 1 file

/Reth.sol
66:    function rethAddress() private view returns (address) {
67:        return
68:            RocketStorageInterface(ROCKET_STORAGE_ADDRESS).getAddress(
69:                keccak256(
70:                    abi.encodePacked("contract.address", "rocketTokenRETH")
71:                )
72:            );
73:    }

120:    function poolCanDeposit(uint256 _amount) private view returns (bool) {
121:        address rocketDepositPoolAddress = RocketStorageInterface(
122:            ROCKET_STORAGE_ADDRESS
123:        ).getAddress(
124:                keccak256(
125:                    abi.encodePacked("contract.address", "rocketDepositPool")
126:                )
127:            );
128:        RocketDepositPoolInterface rocketDepositPool = RocketDepositPoolInterface(

132:        address rocketProtocolSettingsAddress = RocketStorageInterface(
133:            ROCKET_STORAGE_ADDRESS
134:        ).getAddress(
135:                keccak256(
136:                    abi.encodePacked(
137:                        "contract.address",
138:                        "rocketDAOProtocolSettingsDeposit"
139:                    )
140:                )
141:            );


156:    function deposit() external payable onlyOwner returns (uint256) {
157:        // Per RocketPool Docs query addresses each time it is used
158:        address rocketDepositPoolAddress = RocketStorageInterface(
159:            ROCKET_STORAGE_ADDRESS
160:        ).getAddress(
161:                keccak256(
162:                    abi.encodePacked("contract.address", "rocketDepositPool")
163:                )
164:            );

188:            address rocketTokenRETHAddress = RocketStorageInterface(
189:                ROCKET_STORAGE_ADDRESS
190:            ).getAddress(
191:                    keccak256(
192:                        abi.encodePacked("contract.address", "rocketTokenRETH")
193:                    )
194:                );

229:    function poolPrice() private view returns (uint256) {
230:        address rocketTokenRETHAddress = RocketStorageInterface(
231:            ROCKET_STORAGE_ADDRESS
232:        ).getAddress(
233:                keccak256(
234:                    abi.encodePacked("contract.address", "rocketTokenRETH")
235:                )
236:            );
```

Since a solidity version of at least 0.8.13 is used within contracts, string.concat() is enabled and can be used instead of abi.encodePacked(<str>,<str>)

## [R-04] String constants used in multiple places should be defined as constants
```solidity
6 results - 1 file

/Reth.sol
70: abi.encodePacked("contract.address", "rocketTokenRETH")
125: abi.encodePacked("contract.address", "rocketDepositPool")

136: abi.encodePacked(
137:     "contract.address",
138:     "rocketDAOProtocolSettingsDeposit"
139: )

162: abi.encodePacked("contract.address", "rocketDepositPool")
192: abi.encodePacked("contract.address", "rocketTokenRETH")
234: abi.encodePacked("contract.address", "rocketTokenRETH")
```
Description:
String constants used in multiple places such as `contract.address`, `rocketTokenRETH` and `rocketDepositPool` should be defined as constants

## [R-05] Use ternary operators to shorten if/else statements
Description:
Use ternary operators to shorten if/else statements to improve readability and shorten SLOC

The following can be refactored
```solidity
1 result - 1 file

/SafEth.sol
63:    function stake() external payable
79:        if (totalSupply == 0)
80:            preDepositPrice = 10 ** 18; // initializes with a price of 1
81:        else preDepositPrice = (10 ** 18 * underlyingValue) / totalSupply;
```

to:

```solidity
/SafEth.sol
preDepositPrice = (totalSupply == 0) ? 1e18 : (1e18 * underlyingValue) / totalSupply
```

The following can be refactored
```solidity
1 result - 1 file

212:    function ethPerDerivative(uint256 _amount) public view returns (uint256) {
213:        if (poolCanDeposit(_amount))
214:            return
215:                RocketTokenRETHInterface(rethAddress()).getEthValue(10 ** 18);
216:        else return (poolPrice() * 10 ** 18) / (10 ** 18); 
217:    }
```
to:

```solidity
function ethPerDerivative(uint256 _amount) public view returns (uint256) {
    return (poolCanDeposit(_amount)) ?  RocketTokenRETHInterface(rethAddress()).getEthValue(1e18) : poolPrice();
}
```

## [R-06] Indent logic in for loops in code blocks {} correctly
```solidity
2 results - 1 file

/SafEth.sol
63:    function stake() external payable
71:        for (uint i = 0; i < derivativeCount; i++)
72:            underlyingValue +=
73:                (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
74:                    derivatives[i].balance()) /
75:                10 ** 18;

165:    function adjustWeight
171:        for (uint256 i = 0; i < derivativeCount; i++)
172:            localTotalWeight += weights[i];
```

Consider properly including for loop logic within curly braces {} to improve readability of code and prevent accidental execution of logic not intended within a for loop

## [R-07] Should not allow `_weight` parameter to be set as zero
```solidity
2 results - 1 file

/SafEth.sol
63:    function stake() external payable
84:        for (uint i = 0; i < derivativeCount; i++) {
85:            uint256 weight = weights[i];
86:            IDerivative derivative = derivatives[i];
87:            if (weight == 0) continue;

138:    function rebalanceToWeights() external onlyOwner
147:       for (uint i = 0; i < derivativeCount; i++) {
148:            if (weights[i] == 0 || ethAmountToRebalance == 0) continue;
149:            uint256 ethAmount = (ethAmountToRebalance * weights[i]) /
150:                totalWeight;
151:            // Price will change due to slippage
152:            derivatives[i].deposit{value: ethAmount}();
153:        }
```

Description:
In the instances shown above that relates to checks within for loops, weights that have a value of zero are always excluded. This check can be removed by simply adding a check to not allow the `_weight` parameter to be set to zero in `adjustWeight` and `addDerivative`. 

Recommendation:
Add a check using custom errors or a simple require statement like `require(_weight > 0, "weight set to zero")` for `adjustWeight` and `addDerivative` functions

```solidity
/SafEth.sol
function adjustWeight(
    uint256 _derivativeIndex,
    uint256 _weight
) external onlyOwner {
    require(_weight > 0, "weight set to zero");
    weights[_derivativeIndex] = _weight;
    uint256 localTotalWeight = 0;
    for (uint256 i = 0; i < derivativeCount; i++)
        localTotalWeight += weights[i];
    totalWeight = localTotalWeight;
    emit WeightChange(_derivativeIndex, _weight);
}

function addDerivative(
    address _contractAddress,
    uint256 _weight
) external onlyOwner {
    require(_weight > 0, "weight set to zero");    
    derivatives[derivativeCount] = IDerivative(_contractAddress);
    weights[derivativeCount] = _weight;
    derivativeCount++;

    uint256 localTotalWeight = 0;
    for (uint256 i = 0; i < derivativeCount; i++)
        localTotalWeight += weights[i];
    totalWeight = localTotalWeight;
    emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
}
```

## [O-1] Unlocked Pragma
```solidity
4 results - 4 files

/WstEth.sol
2: pragma solidity ^0.8.13;

/SfrxEth.sol
2: pragma solidity ^0.8.13;

/SafEth.sol
2: pragma solidity ^0.8.13;

/Reth.sol
2: pragma solidity ^0.8.13;
```
Recommendation:
Locking pragma helps ensure that contracts do not accidentally get deployed using a different compiler version with which they have been tested the most. Hence, it is reccommended to lock pragmas to a specific Solidity version.
Solidity compiler bugs: [Known solidity bugs](https://github.com/ethereum/solidity/blob/develop/docs/bugs_by_version.json)
Solidity new features: [Solidity new features](https://github.com/ethereum/solidity/blob/develop/Changelog.md)


## [O-2] For mordern and more readable code, update import usages
```solidity
31 results - 4 files

/WstEth.sol
4: import "../../interfaces/IDerivative.sol";
5: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
6: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
7: import "../../interfaces/curve/IStEthEthPool.sol";
8: import "../../interfaces/lido/IWStETH.sol";

/SfrxEth.sol
4: import "../../interfaces/IDerivative.sol";
5: import "../../interfaces/frax/IsFrxEth.sol";
6: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
7: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
8: import "../../interfaces/curve/IFrxEthEthPool.sol";
9: import "../../interfaces/frax/IFrxETHMinter.sol";

/SafEth.sol
 4: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
 5: import "../interfaces/IWETH.sol";
 6: import "../interfaces/uniswap/ISwapRouter.sol";
 7: import "../interfaces/lido/IWStETH.sol";
 8: import "../interfaces/lido/IstETH.sol";
 9: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
10: import "./SafEthStorage.sol";
11: import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";

/Reth.sol
 4: import "../../interfaces/IDerivative.sol";
 5: import "../../interfaces/frax/IsFrxEth.sol";
 6: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
 7: import "../../interfaces/rocketpool/RocketStorageInterface.sol";
 8: import "../../interfaces/rocketpool/RocketTokenRETHInterface.sol";
 9: import "../../interfaces/rocketpool/RocketDepositPoolInterface.sol";
10: import "../../interfaces/rocketpool/RocketDAOProtocolSettingsDepositInterface.sol";
11: import "../../interfaces/IWETH.sol";
12: import "../../interfaces/uniswap/ISwapRouter.sol";
13: import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
14: import "../../interfaces/uniswap/IUniswapV3Factory.sol";
15: import "../../interfaces/uniswap/IUniswapV3Pool.sol";

```

Description:
Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.
This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.
https://betterprogramming.pub/solidity-tutorial-all-about-imports-c65110e41f3a

Recommendation:
 `import {contract1 , contract2} from "filename.sol"`; 

## [O-3] Add missing NatSpec comments such as return and param
```solidity
20 results - 3 files

/WstEth.sol
41: function name() public pure returns (string memory)
48: function setMaxSlippage(uint256 _slippage) external onlyOwner
56: function withdraw(uint256 _amount) external onlyOwner
73: function deposit() external payable onlyOwner returns (uint256)
86: function ethPerDerivative(uint256 _amount) public view returns (uint256) 
93: function balance() public view returns (uint256)

/SfrxEth.sol
44: function name() public pure returns (string memory) 
51: function setMaxSlippage(uint256 _slippage) external onlyOwner
94: function deposit() external payable onlyOwner returns (uint256) 
111: function ethPerDerivative(uint256 _amount) public view returns (uint256) 
122: function balance() public view returns (uint256)

/Reth.sol
50: function name() public pure returns (string memory)
66: function rethAddress() private view returns (address)
83: function swapExactInputSingleHop
107: function withdraw(uint256 amount) external onlyOwner
120: function poolCanDeposit(uint256 _amount) private view returns (bool)
156: function deposit() external payable onlyOwner returns (uint256)
212: function ethPerDerivative(uint256 _amount) public view returns (uint256)
222: function balance() public view returns (uint256)
229: function poolPrice() private view returns (uint256)
```

Description:
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.

In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

Recommendation:
NatSpec comments should be increased in contracts.

## [O-4] Order of function does not comply with the solidity style guide

- constructor
- receive function (if exists)
- fallback function (if exists)
- external
- public
- internal
- private

```solidity
9 results - 4 files

/WstEth.sol
 41: function name() public pure returns (string memory) 
 97: receive() external payable {}

/SfrxEth.sol
 44: function name() public pure returns (string memory)

/SafEth.sol
246: receive() external payable {}

/Reth.sol
 50: function name() public pure returns (string memory)
 66: function rethAddress() private view returns (address)
 83: function swapExactInputSingleHop
120: function poolCanDeposit(uint256 _amount) private view returns (bool)
245: receive() external payable {}
```

## [O-5] Function naming suggestion of internal and private functions starts with _
```solidity
4 results - 1 file

/Reth.sol
 66: function rethAddress() private view returns (address)
 83: function swapExactInputSingleHop
120: function poolCanDeposit(uint256 _amount) private view returns (bool)
229: function poolPrice() private view returns (uint256)
```

Proper use of _ as a function name prefix and a common pattern is to prefix internal and private function names with _. 