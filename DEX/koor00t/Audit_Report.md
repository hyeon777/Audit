## 유동성 공급 문제
### 설명
removeLiquidity()에서 사용자의 보유 LPToken 양을 확인하지 않음
```
function removeLiquidity(uint256 LPTokenAmount, uint256 minimumTokenXAmount, uint256 minimumTokenYAmount) public override returns (uint256, uint256){
        require(LPTokenAmount > 0, "Token must be not zero.");
        reserveX = tokenX.balanceOf(address(this));
        reserveY = tokenY.balanceOf(address(this));
        uint256 _totalSupply = totalSupply();
        uint256 tokenXAmount = (_mul(LPTokenAmount, tokenX.balanceOf(address(this))) / _totalSupply);
        uint256 tokenYAmount = (_mul(LPTokenAmount, tokenY.balanceOf(address(this))) / _totalSupply);
        require(tokenXAmount >= minimumTokenXAmount && tokenYAmount >= minimumTokenYAmount, "Minimum liquidity.");

        reserveX -= tokenXAmount;
        reserveY -= tokenYAmount;

        tokenX.transfer(msg.sender, tokenXAmount);
        tokenY.transfer(msg.sender, tokenYAmount);

        _burn(msg.sender, LPTokenAmount);
        emit RemoveLiquidity(msg.sender, tokenXAmount, tokenYAmount);
        return (tokenXAmount, tokenYAmount);
        
    }
```
### 파급력
Critical
이유: 사용자의 보유 LPToken 양을 확인하지 않으면 다른 사용자의 LPToken양까지 모두 사용해 DEX의 자금을 전부 탈취할 수 있음

### 해결방안
사용자의 LPToken양을 확인하는 require 구문 추가

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
