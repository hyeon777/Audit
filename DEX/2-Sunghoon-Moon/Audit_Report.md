## SWAP함수의 output값
### 설명
swap()에서 스왑하려는 금액이 커질수록 출력되어야하는 output 값과의 차이가 커짐
```
function swap(uint tokenXAmount, uint tokenYAmount, uint tokenMinimumOutputAmount) public returns (uint) {
        console.log("[+] swap()");
        require(tokenXAmount == 0 || tokenYAmount == 0);
        require((tokenXAmount > 0 && tokenYAmount == 0) || (tokenYAmount > 0 || tokenXAmount == 0));


        ERC20 inputToken;
        ERC20 outputToken;

        uint inputReserve;      // pool에 존재하는 입력토큰의 양
        uint outputReserve;     // pool에 존재하는 반환토큰의 양

        uint inputAmount;
        uint outputAmount;
        

        if (tokenXAmount == 0) {
            inputToken = tokenY;
            outputToken = tokenX;
            
            inputAmount = tokenYAmount;
        } else {
            inputToken = tokenX;
            outputToken = tokenY;
            
            inputAmount = tokenXAmount;
        }
        

        inputReserve = inputToken.balanceOf(address(this));
        outputReserve = outputToken.balanceOf(address(this));

        outputAmount = (outputReserve - (inputReserve * outputReserve / (inputReserve + inputAmount))) * 999 / 1000;

        require(outputAmount >= tokenMinimumOutputAmount);
        require(outputToken.balanceOf(address(this)) >= outputAmount);            
        require(inputToken.allowance(msg.sender, address(this)) >= inputAmount); 

        inputToken.transferFrom(msg.sender, address(this), inputAmount);
        outputToken.transfer(msg.sender, outputAmount);


        emit Swap(msg.sender, address(inputToken), address(outputToken), inputAmount, outputAmount);


        return outputAmount;
    }
```
### 파급력
High
이유: 금액의 오차가 커질수록 풀 내부의 토큰 밸런스가 깨지게 됨
### 해결방안
아직 정확한 원인 분석이 되지 않음

