## SWAP함수의 output값
### 설명
swap()에서 스왑하려는 금액이 커질수록 출력되어야하는 output 값과의 차이가 커짐
```
function swap(uint tokenXAmount, uint tokenYAmount, uint tokenMinimumOutputAmount) external override nonReentrant returns (uint) {
        require(tokenXAmount >= 0 || tokenYAmount >= 0, "Amounts must be greater than zero."); 
        require(tokenXAmount == 0 || tokenYAmount == 0, "Only one token can be swapped at a time."); 
        
    
        uint inputAmount;
        uint outputAmount;
        ERC20 inputToken;
        ERC20 outputToken;
        // x*fee->y
        if (tokenXAmount > 0) {
            inputAmount = tokenXAmount;
            inputToken = tokenX;
            outputToken = tokenY;
        // y*fee->x
        } else {
            inputAmount = tokenYAmount;
            inputToken = tokenY;
            outputToken = tokenX;
        }
        // 
        uint inputReserve = inputToken.balanceOf(address(this));
        uint outputReserve = outputToken.balanceOf(address(this));

        // Protocol Fee: 0.1% check (Pi = 0.1% ( = 10 bp) = ø = 0.999 = 99.9% )
        uint amountInMulFee = _mul(inputAmount, (999));
        uint nm = _mul(amountInMulFee, (outputReserve));
        uint dm = _add(_mul(inputReserve, 1000), amountInMulFee);

        outputAmount = _div(nm, dm);


        require(outputAmount >= tokenMinimumOutputAmount, "Minimum output amount not met");

        inputToken.transferFrom(msg.sender, address(this), inputAmount);
        outputToken.transfer(msg.sender, outputAmount);

        emit Swap(msg.sender, inputAmount, outputAmount);
        return outputAmount;
    }
```
### 파급력
High
이유: 금액의 오차가 커질수록 풀 내부의 토큰 밸런스가 깨지게 됨
### 해결방안
아직 정확한 원인 분석이 되지 않음

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


