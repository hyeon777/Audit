## SWAP함수의 output값
### 설명
swap()에서 스왑하려는 금액이 커질수록 출력되어야하는 output 값과의 차이가 커짐
```
function swap(uint256 tokenXAmount, uint256 tokenYAmount, uint256 tokenMinimumOutputAmount) public returns (uint256 outputAmount){
        require(!(tokenXAmount == 0 && tokenYAmount == 0), "invalid input");
        require(!(tokenXAmount != 0 && tokenYAmount != 0), "invalid input");

        tokenX_in_LP = tokenX.balanceOf(address(this));
        tokenY_in_LP = tokenY.balanceOf(address(this));
        
        if(tokenXAmount != 0){
            outputAmount = tokenY_in_LP * (tokenXAmount * 999 / 1000) / (tokenX_in_LP + (tokenXAmount * 999 / 1000));

            require(outputAmount >= tokenMinimumOutputAmount, "minimum ouput amount check failed");
            tokenY_in_LP -= outputAmount ;
            tokenX_in_LP += tokenXAmount;
            tokenX.transferFrom(msg.sender, address(this), tokenXAmount);
            tokenY.transfer(msg.sender, outputAmount );
        }
        else{
            outputAmount = tokenX_in_LP * (tokenYAmount * 999 / 1000) / (tokenY_in_LP + (tokenYAmount * 999 / 1000));

            require(outputAmount >= tokenMinimumOutputAmount, "minimum ouput amount check failed");
            tokenX_in_LP -= outputAmount;
            tokenY_in_LP += tokenXAmount;
            tokenY.transferFrom(msg.sender, address(this), tokenYAmount);
            tokenX.transfer(msg.sender, outputAmount);
        }
        return outputAmount;
    }
```
### 파급력
High
이유: 금액의 오차가 커질수록 풀 내부의 토큰 밸런스가 깨지게 됨
### 해결방안
아직 정확한 원인 분석이 되지 않음
