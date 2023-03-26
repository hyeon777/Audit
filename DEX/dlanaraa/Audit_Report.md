## 사용자의 보유 LPToken 양 확인
### 설명
removeLiquidity()에서 사용자가 보유한 LPToken 양을 확인하지 않음
```
function removeLiquidity(uint256 LPTokenAmount, uint256 minimumTokenXAmount, uint256 minimumTokenYAmount) public returns (uint _receiveX, uint _receiveY) {        
        // require(LPTokenAmount > 0, "less LPToken");
        // require(balanceOf(msg.sender) >= LPTokenAmount, "less LPToken");

        update();

        //return = pool에 있는 토큰 양 * 갖고 있는 LP 양 / total LP 양
        _receiveX = tokenXpool * LPTokenAmount / totalSupply();
        _receiveY = tokenYpool * LPTokenAmount / totalSupply();

        require(minimumTokenXAmount<= _receiveX, "less than minimum");
        require(minimumTokenYAmount<= _receiveY, "less than minimun");

        tokenXpool -= _receiveX;
        tokenYpool -= _receiveY;

        _burn(msg.sender, LPTokenAmount);

        tokenX.transfer(msg.sender, _receiveX);
        tokenY.transfer(msg.sender, _receiveY);
    }
```
### 파급력
Critical
이유: 사용자의 보유 LPToken 양을 확인하지 않으면 다른 사용자의 LPToken양까지 모두 사용해 DEX의 자금을 전부 탈취할 수 있음

### 해결방안
사용자의 LPToken양을 확인하는 require 구문 추가

## SWAP함수의 output값
### 설명
swap()에서 스왑하려는 금액이 커질수록 출력되어야하는 output 값과 실제 값의 오차가 커짐
```
  function swap(uint256 tokenXAmount, uint256 tokenYAmount, uint256 tokenMinimumOutputAmount) external returns (uint256 outputAmount){
        //둘 중 하나는 무조건 0이어야 함
        require(tokenXAmount == 0 || tokenYAmount == 0, "must have 0 amount token");
        require(tokenXpool > 0 && tokenYpool > 0, "can't swap");

        update();

        /* tokenXAmount가 0
        * 사용자 기준 : y를 주고 x를 받아오는 것
        * dex 기준 : y를 받고 x를 돌려주는 것
        * xy = (x-dx)(y+dy) -> dx = (x * dy) / (y + dy)
        */
        if(tokenXAmount == 0) {      
            // 선행 수수료
            outputAmount = (tokenXpool * (tokenYAmount * 999 / 1000)) / (tokenYpool + (tokenYAmount * 999 / 1000));
            
            // 최소값 검증
            require(outputAmount >= tokenMinimumOutputAmount, "less than Minimum");

            // output만큼 빼주고 받아온만큼 더해주기
            tokenXpool -= outputAmount;
            tokenYpool += tokenYAmount;

            // 보내기
            tokenY.transferFrom(msg.sender, address(this), tokenYAmount);
            tokenX.transfer(msg.sender, outputAmount);
        } 
        /* tokenXAmount가 0이 아니면? -> tokenYAmount가 0일 것
        * 사용자 기준 : x를 주고 y를 받아오는 것
        * dex 기준 : x를 받고 y를 돌려주는 것
        * xy = (x+dx)(y-dy) -> (y * dx) / (x + dx)
        */
        else {
            outputAmount = (tokenYpool * (tokenXAmount * 999 / 1000)) / (tokenXpool + (tokenXAmount * 999 / 1000));

            require(outputAmount >= tokenMinimumOutputAmount, "less than Minimum");

            tokenYpool -= outputAmount;
            tokenXpool += tokenXAmount;

            tokenX.transferFrom(msg.sender, address(this), tokenXAmount);
            tokenY.transfer(msg.sender, outputAmount);
        }
}
```
### 파급력
High
이유: 금액의 오차가 커질수록 풀 내부의 토큰 밸런스가 깨지게 됨

### 해결방안
아직 정확한 원인 분석이 되지 않음

## 유동성 공급 문제
### 설명
totalsupply()가 0이 아닌 경우, tokenX에 따라 LPToken의 양이 정해짐.
```
function addLiquidity(uint256 tokenXAmount, uint256 tokenYAmount, uint256 minimumLPTokenAmount) external returns (uint256 LPTokenAmount){
        // 0개 공급은 안됨
        require(tokenXAmount > 0, "tokenXAmount is 0");
        require(tokenYAmount > 0, "tokenYAmount is 0");
        // msg.sender가 dex한테 tokenX와 tokenB에 대한 권한을 줘야함 -> pool에 공급하는 양 만큼!
        require(tokenX.allowance(msg.sender, address(this)) >= tokenXAmount, "ERC20: insufficient allowance");
        require(tokenY.allowance(msg.sender, address(this)) >= tokenYAmount, "ERC20: insufficient allowance");
        // msg.sender의 token 보유량이 공급하려는 양보다 많아야 함
        require(tokenX.balanceOf(msg.sender) >= tokenXAmount, "ERC20: transfer amount exceeds balance");
        require(tokenY.balanceOf(msg.sender) >= tokenYAmount, "ERC20: transfer amount exceeds balance");

        update();

        // 같은 양을 넣더라도 넣는 시점의 상황(수수료 등등)을 고려해서 reward를 해줘야 함 -> totalSupply 값을 이용해서 LPT 계산
        if (totalSupply() == 0) {
            LPTokenAmount = tokenXAmount * tokenYAmount;
        } else {
            LPTokenAmount = tokenXAmount * totalSupply() / tokenXpool;
        }

        // 인자로 받은 LP토큰 최소값보다 작으면 안됨
        require(LPTokenAmount >= minimumLPTokenAmount, "less than minimum");
        // 만족하는 경우 msg.sender한테 LPT 토큰 발행해줌
        _mint(msg.sender, LPTokenAmount);

        // msg.sender가 공급해준만큼 amountX(Y)를 추가해줌
        tokenXpool += tokenXAmount;
        tokenYpool += tokenYAmount;

        //transferFrom으로 msg.sender의 토큰을 DEX로 가져옴
        tokenX.transferFrom(msg.sender, address(this), tokenXAmount);
        tokenY.transferFrom(msg.sender, address(this), tokenYAmount);
    }

```
### 파급력
Medium
이유: tokenX에 따라 LPToken 양을 결정하기 때문에 tokenX를 더 많이 공급하고 이에 따라 LPToken을 많이 가져갈 수 있음

### 해결방안
tokenY에 따른 LPToken의 양과 비교해 더 적은 수량에 따른 LPToken양을 지급

