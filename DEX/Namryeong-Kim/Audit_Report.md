## SWAP함수의 output값
### 설명
swap()에서 스왑하려는 금액이 커질수록 출력되어야하는 output 값과의 차이가 커짐
```
    function swap(uint256 tokenXAmount, uint256 tokenYAmount, uint256 tokenMinimumOutputAmount) public override returns (uint256 outputTokenAmount) {
        require(tokenXAmount > 0 || tokenYAmount > 0, "Token must be not zero.");
        require(tokenXAmount == 0 || tokenYAmount == 0, "Only one token can be swap.");

        ERC20 inputToken;
        ERC20 outputToken;
        uint256 swapsize;

        if(tokenXAmount > 0) {
            inputToken = tokenX;
            outputToken = tokenY;
            swapsize = tokenXAmount;
        } else {
            inputToken = tokenY;
            outputToken = tokenX;
            swapsize = tokenYAmount;
        }

        uint256 inputReserve = inputToken.balanceOf(address(this));
        uint256 outputReserve = outputToken.balanceOf(address(this));

        uint256 fee = _mul(swapsize, (999));
        //init swap => fee
        uint256 bs = _mul(fee, outputReserve);
        uint256 tr = _add(_mul(inputReserve, 1000), fee);
        outputTokenAmount = (bs / tr);
        require(outputTokenAmount >= tokenMinimumOutputAmount, "Not enough Minimum Output.");

        inputToken.transferFrom(msg.sender, address(this), swapsize);
        outputToken.transfer(msg.sender, outputTokenAmount);

        emit Swap(msg.sender, swapsize, outputTokenAmount);
        return outputTokenAmount;


    }
```
### 파급력
High
이유: 금액의 오차가 커질수록 풀 내부의 토큰 밸런스가 깨지게 됨
### 해결방안
아직 정확한 원인 분석이 되지 않음

## 최소 토큰 수량 검사 
### 설명
removeLiquidity()에서 minimum_token_amount를 검사할때 등호를 포함하지 않음
```
 function removeLiquidity(uint256 _LPTokenAmount, uint256 _minimumTokenXAmount, uint256 _minimumTokenYAmount) external returns(uint256, uint256){
        require(_LPTokenAmount > 0, "INSUFFICIENT_AMOUNT");
        require(balanceOf(msg.sender) >= _LPTokenAmount, "INSUFFICIENT_LPtoken_AMOUNT");
        
        uint256 reserveX;
        uint256 reserveY;
        uint256 amountX;
        uint256 amountY;

        (reserveX, reserveY) = _update();

        amountX =  reserveX * _LPTokenAmount/ totalSupply();
        amountY = reserveY * _LPTokenAmount / totalSupply();
        require(amountX >_minimumTokenXAmount && amountY>_minimumTokenYAmount, "INSUFFICIENT_LIQUIDITY_BURNED");
        
        tokenX_.transfer(msg.sender, amountX);
        tokenY_.transfer(msg.sender, amountY);

        _burn(msg.sender, _LPTokenAmount);

        (reserveX_,reserveY_) = _update();
        return (amountX, amountY);
    }
```
### 파급력
일반 버그
이유: 컨트랙트에 직접적인 악영향은 미치지 않지만, 사용자가 원하는 조건이 성립되어도 함수가 실행되지 않을 수 있음

### 해결방안
비교 구문에 등호 추가
