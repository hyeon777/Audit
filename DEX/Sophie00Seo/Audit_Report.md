## 단순 오타
### 설명
#L48에서 tokenY를 tokenX로 swap할 때 tokenY의 amount에 tokenY가 아닌 tokenX의 amount를 추가함
```
function swap(uint256 tokenXAmount, uint256 tokenYAmount, uint256 tokenMinimumOutputAmount) public returns (uint256){
        require(((tokenXAmount == 0) && (tokenYAmount > 0)) || ((tokenXAmount > 0) && (tokenYAmount == 0)), "only one amount should be zero");
        require(_amountX > 0 && _amountY > 0, "no token to swap");

        updateTokenBalance();
        
        uint256 amount_;
        if(tokenXAmount > 0){
            amount_ = _amountY * (tokenXAmount * 999 / 1000) / (_amountX + (tokenXAmount * 999 / 1000));
            
            require(amount_ >= tokenMinimumOutputAmount, "less than minimum swap amount");
            _amountY -= amount_ ;
            _amountX += tokenXAmount;
            _tokenX.transferFrom(msg.sender, address(this), tokenXAmount);
            _tokenY.transfer(msg.sender, amount_ );
        }
        else{
            amount_ = _amountX * (tokenYAmount * 999 / 1000) / (_amountY + (tokenYAmount * 999 / 1000));

            require(amount_ >= tokenMinimumOutputAmount, "less than minimum swap amount");
            _amountX -= amount_;
            _amountY += tokenXAmount;
            _tokenY.transferFrom(msg.sender, address(this), tokenYAmount);
            _tokenX.transfer(msg.sender, amount_);
        }
        return amount_;
    }
```
개발자의 단순실수이지만, 잘못하면 취약점으로 이어질 수 있음
### 파급력
잠재적 문제점
이유: swap을 하고 다른 함수를 진행하기 전까지는 tokenY의 잔액이 정상적으로 반영되지 않음.
그러나, 해당 컨트랙트에서는 모든 함수에서 updateTokenBalance()를 사용하기 때문에 _amountY의 값에 이상이 있어도 다시 원래 잔액으로 덮어짐.

### 해결방안
tokenXAmount를 tokenYAmount로 변경

## 유동성 공급 문제
### 설명
#L71에서 totalsupply()가 0이 아닌 경우, tokenX에 따라 LPToken의 양이 정해짐.
```
function addLiquidity(uint256 tokenXAmount, uint256 tokenYAmount, uint256 minimumLPTokenAmount) public returns (uint256){
        require(tokenXAmount > 0, "token x amount is 0");
        require(tokenYAmount > 0, "token y amount is 0");
        require(_tokenX.allowance(msg.sender, address(this)) >= tokenXAmount, "ERC20: insufficient allowance");
        require(_tokenY.allowance(msg.sender, address(this)) >= tokenYAmount, "ERC20: insufficient allowance");
        require(_tokenX.balanceOf(msg.sender) >= tokenXAmount, "ERC20: transfer amount exceeds balance");
        require(_tokenY.balanceOf(msg.sender) >= tokenYAmount, "ERC20: transfer amount exceeds balance");
        
        updateTokenBalance();

        // what if _amountX/_amountY and tokenXAmount/tokenYAmount is different?
        uint lpAmount;
        if(totalSupply() == 0){
            lpAmount = tokenXAmount * tokenYAmount / _decimal; // is amount best? no overflow?
        }
        else{
            require(_decimal * tokenXAmount / tokenYAmount == _decimal * _amountX / _amountY, "amount breaks the pool ratio");
            lpAmount = totalSupply() * tokenXAmount / _amountX;
        }
        require(lpAmount >= minimumLPTokenAmount, "less than minimum lp token amount");
        _amountX += tokenXAmount;
        _amountY += tokenYAmount;
        _tokenX.transferFrom(msg.sender, address(this), tokenXAmount);
        _tokenY.transferFrom(msg.sender, address(this), tokenYAmount);
        _mint(msg.sender, lpAmount);
        return lpAmount;
    }
```
###파급력
Medium
이유: tokenX에 따라 LPToken 양을 결정하기 때문에 tokenX를 많이 공급해 이에 따라 LPToken을 많이 가져갈 수 있음

###해결방안
tokenY에 따른 LPToken의 양과 비교해 더 적은 수량에 따른 LPToken양을 지급



