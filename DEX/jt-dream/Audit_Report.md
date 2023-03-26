## 토큰 비율 확인
### 설명
addliquidity()에서 공급량 비율을 확인하지 않고 바로 LPToken양을 측정함
```
function addLiquidity(uint256 tokenXAmount, uint256 tokenYAmount, uint256 minimumLPTokenAmount) public returns (uint256 LPTokenAmount){
    require(tokenXAmount > 0, "Less TokenA Supply");
    require(tokenYAmount > 0, "Less TokenB Supply");
    require(tokenX.allowance(msg.sender, address(this)) >= tokenXAmount, "ERC20: insufficient allowance");
    require(tokenY.allowance(msg.sender, address(this)) >= tokenYAmount, "ERC20: insufficient allowance");
    uint256 liqX; //liquidity of x
    uint256 liqY; //liquidity of y
    amountX = tokenX.balanceOf(address(this));// token X amount udpate
    amountY = tokenY.balanceOf(address(this));// token Y amount update
    
    if(totalSupply_ ==0 ){ //if first supply 
        LPTokenAmount = _sqrt(tokenXAmount*tokenYAmount);
    }
    else{// calculate over the before
        liqX = _mul(tokenXAmount ,totalSupply_)/amountX;
        liqY = _mul(tokenYAmount ,totalSupply_)/amountY;
        LPTokenAmount = (liqX<liqY) ? liqX:liqY; 
    }
    require(LPTokenAmount >= minimumLPTokenAmount, "Less LP Token Supply");
    transfer_(msg.sender,LPTokenAmount);
    totalSupply_ += LPTokenAmount;
    amountX += tokenXAmount;
    amountY += tokenYAmount;
    tokenX.transferFrom(msg.sender, address(this), tokenXAmount);
    tokenY.transferFrom(msg.sender, address(this), tokenYAmount);
    return LPTokenAmount;
}
```
### 파급력
High
이유: 풀에 있는 토큰의 비율을 검사하지 않으면 그 비율을 악의적으로 조정할 수 있음

### 해결방안
토큰의 비율을 확인하는 require(reserveX * tokenYAmount == reserveY * tokenXAmount); 조건문 추가


## 사용자가 보유한 LPToken 양 확인
### 설명
removeLiquidity()에서 사용자가 보유한 LPToken의 양을 확인하지 않음
```
function removeLiquidity(uint256 LPTokenAmount, uint256 minimumTokenXAmount, uint256 minimumTokenYAmount) public returns (uint256 tokenXAmount_,uint256 tokenYAmount_){
    require(LPTokenAmount > 0, "Less LP Token Supply");
    amountX = tokenX.balanceOf(address(this));// token X amount udpate
    amountY = tokenY.balanceOf(address(this));// token Y amount update
    tokenXAmount_ = _mul(amountX,LPTokenAmount)/totalSupply_;
    tokenYAmount_ = _mul(amountY,LPTokenAmount)/totalSupply_;
    require(tokenXAmount_ >= minimumTokenXAmount, "TokenX amount below minimum");
    require(tokenYAmount_ >= minimumTokenYAmount, "TokenY amount below minimum");
    amountX -= tokenXAmount_;
    amountY -= tokenYAmount_;
    _burn(msg.sender,LPTokenAmount);
    tokenX.transfer(msg.sender,tokenXAmount_);
    tokenY.transfer(msg.sender,tokenYAmount_);

    return (tokenXAmount_,tokenYAmount_);
}
```
### 파급력
Critical
다른 사용자에게 민틴된 LPToken도 모두 사용해 DEX의 토큰을 모두 빼갈 수 있음

### 해결방안
사용자가 보유한 LPToken의 양을 확인하는 조건문 추가
