## 토큰 비율 확인
### 설명
addliquidity()에서 공급량 비율을 확인하지 않고 바로 LPToken양을 측정함
```
function addLiquidity(uint256 tokenXAmount, uint256 tokenYAmount, uint256 minimumLPTokenAmount) external returns (uint256 LPTokenAmount){
        require(tokenXAmount>0 && tokenYAmount>0);
        require(_tokenX.allowance(msg.sender, address(this))>=tokenXAmount,"ERC20: insufficient allowance");
        require(_tokenY.allowance(msg.sender, address(this))>=tokenYAmount,"ERC20: insufficient allowance");
        require(_tokenX.balanceOf(msg.sender)>=tokenXAmount,"ERC20: transfer amount exceeds balance");
        require(_tokenY.balanceOf(msg.sender)>=tokenYAmount,"ERC20: transfer amount exceeds balance");

        uint lpToken; 
        uint X;
        uint Y;

        if (liquiditySum==0){
            lpToken=Math.sqrt(tokenXAmount*tokenYAmount); //initial token amount
        }
        else {
            (X,Y)=update();
            
            // 기존 토큰에 대한 새 토큰의 비율로 계산
            uint liquidityX=liquiditySum*tokenXAmount/X;
            uint liquidityY=liquiditySum*tokenYAmount/Y;
            lpToken=(liquidityX<liquidityY)?liquidityX:liquidityY;
        }

        require(lpToken>=minimumLPTokenAmount);

        liquiditySum+=lpToken;
        liquidityUser[msg.sender]+=lpToken;

        _tokenX.transferFrom(msg.sender, address(this), tokenXAmount);
        _tokenY.transferFrom(msg.sender, address(this), tokenYAmount);
        transfer(msg.sender, lpToken);
        
        update();

        return lpToken;
    }
```
### 파급력
High
이유: 풀에 있는 토큰의 비율을 검사하지 않으면 그 비율을 악의적으로 조정할 수 있음

### 해결방안
토큰의 비율을 확인하는 require(reserveX * tokenYAmount == reserveY * tokenXAmount); 조건문 추가

## transfer() 호출
### 설명
transfer()을 사용자가 호출할 수 있음
```
    function transfer(address to, uint256 lpAmount) override public returns (bool){
        _mint(to, lpAmount);
        return true;
    }
```
### 파급력
Critical
이유: 사용자가 원하는만큼 minting 가능

### 해결방안
require(msg.sender == address(this)); 조건문 추가