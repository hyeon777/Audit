## 공급량 비율 확인
### 설명
addliquidity()에서 공급량 비율을 확인하지 않고 바로 LPToken양을 측정함
```
 function addLiquidity(uint256 tokenXAmount, uint256 tokenYAmount, uint256 minimumLPTokenAmount) external returns (uint256 LPTokenAmount){
        require(tokenXAmount > 0, "tokenXAmount must exceed 0");
        require(tokenYAmount > 0, "tokenYAmount must exceed 0");
        require(ERC20(token_x).allowance(msg.sender, address(this)) >= tokenXAmount,"ERC20: insufficient allowance");
        require(ERC20(token_y).allowance(msg.sender, address(this)) >= tokenYAmount,"ERC20: insufficient allowance");
        require(ERC20(token_x).balanceOf(msg.sender) >= tokenXAmount,"ERC20: transfer amount exceeds balance");
        require(ERC20(token_y).balanceOf(msg.sender) >= tokenYAmount,"ERC20: transfer amount exceeds balance");
        
        // reserve를 항상 최신화
        reserve_x = ERC20(token_x).balanceOf(address(this));
        reserve_y = ERC20(token_y).balanceOf(address(this));
        
        // LP token 발급
        // uniswap v2 => 처음 이후 lp: 넣을 토큰의 개수 * 원래 있었던 lP수 / 토큰 넣기전 리저브
        // 그냥 dex에 transfer시 다음에 유동성 공급하는사람은 lp 지분에 손해를 보는 구조...?????
        uint token_amount;
        if(totalSupply() == 0){
            token_liquidity_L = sqrt(reserve_x * reserve_y);
            token_amount = sqrt((tokenXAmount * tokenYAmount));
        } else{
            token_amount = (tokenXAmount * 10 ** 18 * totalSupply() / reserve_x) / 10 ** 18;
        }
        require(token_amount > minimumLPTokenAmount, "token_amount > minimumLPTokenAmount");

        require(ERC20(token_x).transferFrom(msg.sender, address(this), tokenXAmount));
        require(ERC20(token_y).transferFrom(msg.sender, address(this), tokenYAmount));
        _mint(msg.sender, token_amount);

        return token_amount;
    }
```
### 파급력
High
이유: 풀에 있는 토큰의 비율을 검사하지 않으면 그 비율을 악의적으로 조정할 수 있음

### 해결방안
토큰의 비율을 확인하는 require(reserveX * tokenYAmount == reserveY * tokenXAmount); 조건문 추가

## SWAP함수의 output값
### 설명
swap()에서 스왑하려는 금액이 커질수록 출력되어야하는 output 값과의 차이가 커짐
```
 function swap(uint256 tokenXAmount, uint256 tokenYAmount, uint256 tokenMinimumOutputAmount) external returns (uint256 outputAmount){
        require((tokenXAmount > 0 && tokenYAmount == 0 ) || (tokenYAmount > 0 && tokenXAmount == 0));
        reserve_x = ERC20(token_x).balanceOf(address(this)) - fee_x;
        reserve_y = ERC20(token_y).balanceOf(address(this)) - fee_y;
        
        uint tmp_reserve_y;
        uint tmp_reserve_x;
        // transferFrom에서 allowance, tokenAmount 체크를 해주긴 하는데, 굳이 allowance체크를 해야할까?
        // 스왑 이후의 X'
        // K / X' = Y'
        // 스왑 이후의 Y'
        // Y-Y' => 스왑으로 얻는 y토큰 => 수수료 0.1%
        // 여기서는 자리수 올려서 계산하는게 의미가 없는듯??????
        if(tokenXAmount == 0){
            require(ERC20(token_x).allowance(msg.sender, address(this)) >= tokenXAmount,"ERC20: insufficient allowance");
            
            tmp_reserve_y = reserve_y + tokenYAmount;
            tmp_reserve_x = (reserve_x * reserve_y) / tmp_reserve_y;
            outputAmount = (reserve_x - tmp_reserve_x) * 999 / 1000;
            fee_x += (reserve_x - tmp_reserve_x) / 1000;

            require(outputAmount >= tokenMinimumOutputAmount, "tokenMinimumOutputAmount");
            require(ERC20(token_y).transferFrom(msg.sender, address(this), tokenYAmount));
            require(ERC20(token_x).transfer(msg.sender, outputAmount));
        }else if(tokenYAmount == 0){
            require(ERC20(token_y).allowance(msg.sender, address(this)) >= tokenYAmount,"ERC20: insufficient allowance");
            
            tmp_reserve_x = reserve_x + tokenXAmount;
            tmp_reserve_y = (reserve_x * reserve_y) / tmp_reserve_x;
            outputAmount = (reserve_y - tmp_reserve_y) * 999 / 1000;
            fee_y += (reserve_y - tmp_reserve_y) / 1000;

            require(outputAmount >= tokenMinimumOutputAmount, "tokenMinimumOutputAmount");
            require(ERC20(token_x).transferFrom(msg.sender, address(this), tokenXAmount));
            require(ERC20(token_y).transfer(msg.sender, outputAmount));
        }
    }
```
### 파급력
High
이유: 금액의 오차가 커질수록 풀 내부의 토큰 밸런스가 깨지게 됨
### 해결방안
아직 정확한 원인 분석이 되지 않음
