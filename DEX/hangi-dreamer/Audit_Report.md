## removeLiquidity() 작동 문제
### 설명
removeLiquidity()에서 LPToken의 양에 해당하는 tokenX, tokenY를 사용자에게 전송하지 않고, LPToken 또한 burn하지 않음
```
function removeLiquidity(
        uint256 LPTokenAmount,
        uint256 minimumTokenXAmount,
        uint256 minimumTokenYAmount
    ) external returns (uint rx, uint ry) {
        require(LPTokenAmount > 0);
        require(minimumTokenXAmount >= 0);
        require(minimumTokenYAmount >= 0);
        require(lpt.balanceOf(msg.sender) >= LPTokenAmount);

        (uint balanceOfX, uint balanceOfY) = pairTokenBalance();

        uint lptTotalSupply = lpt.totalSupply();

        rx = balanceOfX * LPTokenAmount / lptTotalSupply;
        ry = balanceOfY * LPTokenAmount / lptTotalSupply;

        require(rx >= minimumTokenXAmount);
        require(rx >= minimumTokenYAmount);
    }
```
### 파급력
Critical
removeLiquidity()가 아예 작동하지 않는다면, 컨트랙트 자체가 정상적으로 수행되지 않음.

### 해결방안
transfer()과 burn() 을 이용해 토큰을 처리하는 과정을 추가

## 단순 오타 문제
### 설명
#L156에서 TokenY의 최소 수량을 확인하지 않음
```
function removeLiquidity(
        uint256 LPTokenAmount,
        uint256 minimumTokenXAmount,
        uint256 minimumTokenYAmount
    ) external returns (uint rx, uint ry) {
        require(LPTokenAmount > 0);
        require(minimumTokenXAmount >= 0);
        require(minimumTokenYAmount >= 0);
        require(lpt.balanceOf(msg.sender) >= LPTokenAmount);

        (uint balanceOfX, uint balanceOfY) = pairTokenBalance();

        uint lptTotalSupply = lpt.totalSupply();

        rx = balanceOfX * LPTokenAmount / lptTotalSupply;
        ry = balanceOfY * LPTokenAmount / lptTotalSupply;

        require(rx >= minimumTokenXAmount);
        require(rx >= minimumTokenYAmount);
    }

```
### 파급력
일반 버그
이유: 사용자가 설정한 토큰의 최소 수량은 반영되지 않지만 컨트랙트의 작동에 직접적인 영향은 주지 않음.

### 해결방안
rx를 ry로 수정



