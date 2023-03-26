## LPToken Minting Burning 
### 설명
addLiquidity()와 removeLiquidity()에서 LPToken을 발행하지 않음
```
function addLiquidity(
        uint256 tokenXAmount,
        uint256 tokenYAmount,
        uint256 minimumLPTokenAmount
    ) external returns (uint256 LPTokenAmount) {
        require(tokenXAmount > 0 && tokenYAmount > 0);
        setReserve(tokenXAmount, tokenYAmount);
        uint256 liquidity;
        uint256 optToken = quote(curX, reservedX, reservedY);
        if (optToken > curY) {
            optToken = quote(curY, reservedY, reservedX);
            require(optToken == curX);
            tokenX.transferFrom(msg.sender, address(this), optToken);
            tokenY.transferFrom(msg.sender, address(this), tokenYAmount);
            liquidity = sqrt(optToken * tokenYAmount);
        } else {
            require(optToken == curY);
            tokenX.transferFrom(msg.sender, address(this), tokenXAmount);
            tokenY.transferFrom(msg.sender, address(this), optToken);
            liquidity = sqrt(optToken * tokenXAmount);
        }

        if (optToken > curY) {
            if (sqrt((reservedX * reservedY) / (curX * curY)) == 1) {
                reward = liquidity;
            } else {
                reward =
                    (sqrt((reservedX * reservedY) / (curX * curY)) - 1) *
                    reward;
            }
            setReserve(optToken, tokenYAmount);
        } else {
            if (sqrt((reservedX * reservedY) / (curX * curY)) == 1) {
                reward = liquidity;
            } else {
                reward =
                    (sqrt((reservedX * reservedY) / (curX * curY)) - 1) *
                    reward;
            }
            setReserve(tokenXAmount, optToken);
        }

        preX = tokenX.balanceOf(address(this));
        preY = tokenY.balanceOf(address(this));

        if (minimumLPTokenAmount > reward) revert();
        rewards[msg.sender] += reward;
        totalReward += reward;
        return reward;
    }
```
### 파급력
Medium
LPToken을 발행하지 않고 Mapping에만 저장함

### 해결방안
mint()와 burn()을 이용해 ERC20 기반의 LPToken 사용

## SWAP함수의 output값
### 설명
swap()에서 스왑하려는 금액이 커질수록 출력되어야하는 output 값과의 차이가 커짐
```
function swap(
        uint256 tokenXAmount,
        uint256 tokenYAmount,
        uint256 tokenMinimumOutputAmount
    ) external returns (uint256 outputAmount) {
        require(tokenXAmount == 0 || tokenYAmount == 0);
        uint256 tokenFrom;
        uint256 tokenTo;
        (tokenFrom, tokenTo) = (tokenXAmount == 0)
            ? (tokenYAmount, tokenXAmount)
            : (tokenXAmount, tokenYAmount);

        setReserve();

        if (tokenXAmount == 0) {
            tokenY.transferFrom(msg.sender, address(this), tokenFrom);
            outputAmount = uint256(
                -(int(reservedY * reservedX) / int(reservedY + tokenFrom)) +
                    int(reservedX)
            );
        } else {
            tokenX.transferFrom(msg.sender, address(this), tokenFrom);
            outputAmount = uint256(
                -(int(reservedX * reservedY) / int(reservedX + tokenFrom)) +
                    int(reservedY)
            );
        }

        setReserve();
        outputAmount = (outputAmount * (feeRate - 1)) / feeRate;

        if (tokenXAmount == 0) {
            tokenX.transfer(msg.sender, outputAmount);
        } else {
            tokenY.transfer(msg.sender, outputAmount);
        }
        setReserve();

        require(outputAmount >= tokenMinimumOutputAmount);
    }
```
### 파급력
High
이유: 금액의 오차가 커질수록 풀 내부의 토큰 밸런스가 깨지게 됨
### 해결방안
아직 정확한 원인 분석이 되지 않음
