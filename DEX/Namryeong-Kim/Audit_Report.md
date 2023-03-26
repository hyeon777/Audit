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


