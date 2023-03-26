## transfer() 호출
### 설명
transfer()을 사용자가 호출할 수 있음
```
    function transfer(address to, uint256 lpAmount) public override(ERC20, IDex) returns (bool) {
        _mint(to, lpAmount);
        return true;
    }
```
### 파급력
Critical
이유: 사용자가 원하는만큼 minting 가능

### 해결방안
require(msg.sender == address(this)); 조건문 추가

### LPToken minting
### 설명
addliquidity()에서 민팅을 하지 않음
```
function addLiquidity(uint256 tokenXAmount, uint256 tokenYAmount, uint256 minimumLPTokenAmount) external returns (uint256 LPTokenAmount){
    require(tokenXAmount > 0 && tokenYAmount > 0,"please check tokenXAmount and tokenYAmount");
    uint256 xBalance = balances[address(tokenX)];
    uint256 yBalance = balances[address(tokenY)];
    uint256 liquidityX;
    uint256 liquidityY;
    if(totalLiquidity == 0) {
        LPTokenAmount = Math.sqrt(tokenXAmount * tokenYAmount);
        require(LPTokenAmount >= minimumLPTokenAmount);
        totalLiquidity = LPTokenAmount;
    } 
    else {
        liquidityX = (totalLiquidity*tokenXAmount) / xBalance;
        liquidityY = (totalLiquidity*tokenYAmount) / yBalance;
        LPTokenAmount = (liquidityX < liquidityY) ? liquidityX : liquidityY;
        require(LPTokenAmount >= minimumLPTokenAmount);
        totalLiquidity += LPTokenAmount;
    }
    tokenX.transferFrom(msg.sender, address(this), tokenXAmount);
    tokenY.transferFrom(msg.sender, address(this), tokenYAmount);
    balances[address(tokenX)] += tokenXAmount;
    balances[address(tokenY)] += tokenYAmount;
    LPToken_balances[msg.sender] += LPTokenAmount;
    return LPTokenAmount;
}   
```

### 파급력
Medium
이유: LPToken을 민팅하지 않고 매핑으로만 저장함

### 해결방안
_mint()와 _burn()을 이용해 ERC20기반의 토큰을 LPToken으로 사용