# Safemoon-Contract-Review-Documentation

### Understanding Safemoon Reflection Mechanism

MAX = 115792089237316195423570985008687907853269984665640564039457584007913129639935

_tTotal = 1000000000000000000000000

_rTotal = MAX - (MAX % _tTotal)

MAX % _tTotal = 115792089237316195423570985008687907853269984665640564039457584007913129639935 % 1000000000000000000000000

MAX % _tTotal = 39457584007913129639935

_rTotal = 115792089237316195423570985008687907853269984665640564039457584007913129639935 - 39457584007913129639935

_rTotal = 115792089237316195423570985008687907853269984665640564000000000000000000000000

- maxTxAmount = 5000000000000000000000
- numTokensSellToAddToLiquidity = 500000000000000000000

In the contract constructor:
- owners reflection balance is set to the total reflection (_rTotal)

- _rOwned[owner] = _rTotal

_rOwned[owner] = 115792089237316195423570985008687907853269984665640564000000000000000000000000



### Let's get the balanceOf(owner) given the reflection balance of the owner

- to get the balanceOf(owner) we have to look into the balanceOf(account) function

- First the balanceOf(account) function checks if an account is excluded from rewards using _isExcluded[account]
- _isExcluded[account] is true, it uses the _tOwned[account] to return the address balance.
- But if the account is not excluded from rewards which means _isExcluded[account] returns false, then the tokenFromReflection(_rOwned[account]) function is called taking in the _rOwned[account], which is the reflection balance of the account as parameter.
- the reflection amount is the rAmount.

- Looking at the tokenFromReflection(_rOwned[account])
- rAmount = _rOwned[account]
- first it requires that the rAmount is less than or equal to the _rTotal (total reflection)
- then the _getRate() function is called

- Looking inside the _getRate() function ////
- the _getRate() function is calling another function _getCurrentSupply() which returns the rSupply and tSupply of the token

- Looking inside the _getCurrentSupply()
- it is basically returning the reflection supply(rSupply) and the token total supply (tSupply)
- the tSupply is equal to _tTotal (total token supply)

- tSupply = _tTotal
- tSupply = 1000000000000000000000000

- rSupply = _rTotal
- rSupply = 115792089237316195423570985008687907853269984665640564000000000000000000000000

- the _getRate() function returns currentRate
- currentRate is achieved by dividing rSupply / tSupply

- currentRate = rSupply / tSupply

Note: tSupply is constant

- currentRate = 115792089237316195423570985008687907853269984665640564000000000000000000000000 / 1000000000000000000000000

- currentRate = 115792089237316195423570985008687907853269984665640564

- Once the currentRate is gotten
- actual balance of an address that is not excluded from reward will be a division of reflection balance / currentRate

- balanceOf(address) = _rOwned[address] / currentRate

- balanceOf(address) = rAmount / currentRate
- Using the contract owner balance for this, hence:

- balanceOf(owner) = 115792089237316195423570985008687907853269984665640564000000000000000000000000 / 115792089237316195423570985008687907853269984665640564

- balanceOf(owner) = 1000000000000000000000000 (which is equivalent to _tTotal)




### Understanding How Safemoon _transfer() function Works

- Let transfer amount = 100000000000 (100 token) from the owner

The starting point of transferring token is:
1. transfer(address recipient, uint amount) function which calls
2. _transfer(_msgSender(), recipient, amount) function

Let's look at the _transfer() function

- The _transfer() function performs the necessary validation checks
- It gets the contract balance of the token using balanceOf(address(this))
- It checks if the token balance of the contract is >= the maxTxAmount (Maximum Transaction Amount)
- If the condition passes, then it sets the contract balance of the token to the maxTxAmount (Maximum Transaction Amount)
- It uses this check to provide liquidity to the token
- Liquidity is provided by dividing the contractBalance into 2, converts one of the half to ether (ETH) and then use the eth that was converted and the remaining half in token to provide liquidity in Token/Eth pair
So in our case it will be Safemoon/Eth pair.

- After the swapAndLiquify, the logic then proceeds to the _tokenTransfer() function.

- The _tokenTransfer function takes in 4 parameters: the from address, the to address, the amount and the takeFee status which will be true by default but will be set to false if the from address or the to address is excluded from fee. 

- owner is transferring 100000000000 token
- owner is not excluded from receiving rewards which means _isExcluded[sender] will be false
- also recipient is not excluded from receiving rewards which means _isExcluded[recipient] will also be false
- then the _transferStandard(sender, recipient, amount) will be called.
- inside the _transferStandard(sender, recipient, _amount)
- _getValues(tAmount) is called
- this returns the following values:
- 1 rAmount (reflection amount)
- 2 rTransferAmount (reflection amount to transfer)
- 3 rFee (reflection fee)
- 4 tTransferAmount (actual transaction amount to transfer)
- 5 tFee (tax fee)
- 6 tLiquidity

Remember our tAmount = 100000000000
- the _getValues(tAmount) is called
- inside the _getValues(tAmount), the _getTValues(tAmount) is also called (This return the tTransferAmount, tFee, tLiquidity)
- the tTransferAmount = tAmount - (tFee + tLiquidity)

- tFee = tAmount * _taxFee / 100
- tFee = (100000000000 * 5) / 100
- tFee = 5000000000

- tLiquidity = tAmount * _liquidityFee / 100
- tLiquidity = (100000000000 * 5) / 100
- tLiquidity = 5000000000

- tTransferAmount = tAmount - (tFee + tLiquidity)
- tTransferAmount = 100000000000 - (5000000000 + 5000000000)
- tTransferAmount = 90000000000

- After the above values are gotten from the _getTValues(tAmount)
- Then it's now time to get the reflection values by calling _getRValues(tAmount, tFee, tLiquidity, _getRate())
- the _getRate() function returns the current rate which is the division of the rSupply/tSupply;
- _getRate will return value = 115792089237316195423570985008687907853269984665640564

- currentRate = 115792089237316195423570985008687907853269984665640564


- we want to get the reflection amount(rAmount) using the transaction amount(tAmount)
- tAmount is still  = 100000000000

- rAmount = tAmount * currentRate
- rAmount = 100000000000 * 115792089237316195423570985008687907853269984665640564
- rAmount = 11579208923731619542357098500868790785326998466564056400000000000

- rFee = tFee * currentRate
- rFee = 5000000000 * 115792089237316195423570985008687907853269984665640564
- rFee = 578960446186580977117854925043439539266349923328202820000000000

- rLiquidity = tLiquidity * currentRate
- rLiquidity = 5000000000 * 115792089237316195423570985008687907853269984665640564
- rLiquidity = 578960446186580977117854925043439539266349923328202820000000000


- rTransferAmount = rAmount - (rFee + rLiquidity)

- rTransferAmount = 11579208923731619542357098500868790785326998466564056400000000000 - (578960446186580977117854925043439539266349923328202820000000000 + 578960446186580977117854925043439539266349923328202820000000000)

- rTransferAmount = 10421288031358457588121388650781911706794298619907650760000000000

- _rOwned[sender] = _rOwned[sender] - rAmount
- _rOwned[owner] = _rOwned[owner] - rAmount
- _rOwned[owner] = 115792089237316195423570985008687907853269984665640564000000000000000000000000 - 11579208923731619542357098500868790785326998466564056400000000000

- _rOwned[owner] = 115792089237304616214647253389145550754769115874855237001533435943600000000000

- _rOwned[recipient] = _rOwned[recipient] + rTransferAmount
- Asumming recipient had zero balance

- _rOwned[recipient] = 0 + 10421288031358457588121388650781911706794298619907650760000000000

- _rOwned[recipient] = 10421288031358457588121388650781911706794298619907650760000000000

- Let's now see how the reflection happens
- The _takeLiquidity(tLiquidity) is called
- The _reflectFee(rFee, tFee) is also called

- Looking at the _takeLiquidity(tLiquidity) function ////
- The _takeLiquidity() function will get the current rate
- currentRate = 115792089237316195423570985008687907853269984665640564
- it will use the tLiquidity and the current rate to get the rLiquidity
- rLiquidity = tLiquidity * currentRate
- rLiquidity = 5000000000 * 115792089237316195423570985008687907853269984665640564
- rLiquidity = 578960446186580977117854925043439539266349923328202820000000000

- Then add the rLiquidity to the reflection balance of the contract
- _rOwned[addres(this)] = _rOwned[addres(this)] + rLiquidity
- Asumming address(this) has zero balance at first
- _rOwned[addres(this)] = 0 + 578960446186580977117854925043439539266349923328202820000000000
- _rOwned[addres(this)] = 578960446186580977117854925043439539266349923328202820000000000

/// But if addres(this) is excluded ///

- _tOwned[addres(this)] = _tOwned[addres(this)] + tLiquidity
- _tOwned[addres(this)] = 0 + 5000000000
- _tOwned[addres(this)] = 5000000000

- Now looking at reflectFee(rFee, tFee) function
- The reflectFee function subtracts the rFee from the _rTotal
- and also increments the _tFeeTotal by adding the tFee
- Remeber that the currentRate is gotten by dividing the rSupply(rTotal) by tSupply(tTotal)
- And the balance of an address is gotten by diving the rAmount of the address by the currentRate
- This means if the rTotal is reduced, the rSupply will be reduce too
- If the rSupply is reduced, then currentRate is also reduced too because the tSupply is fixed.
- Hence the reduction in rSupply means balance of an address will be increased.

- balanceOf(address) = rAmount / currentRate


- In the _reflectFee(rFee, tFee) function, we have:
- _rTotal = rTotal - rFee
- _rTotal = 115792089237316195423570985008687907853269984665640564000000000000000000000000 - 578960446186580977117854925043439539266349923328202820000000000

- _rTotal = 115792089237315616463124798427710789998344941226101297650076671797180000000000

- _tFeeTotal = tFeeTotal + tFee
- _tFeeTotal = 0 + 5000000000
- _tFeeTotal = 5000000000

This is the flow of safemoon reflection


////////////////
Let's now check the balanceOf the recipient using the new _rTotal value

- currentRate = rSupply / tSupply

- tSupply = _tTotal
- tSupply = 1000000000000000000000000

- rSupply = _rTotal
- rSupply = 115792089237315616463124798427710789998344941226101297650076671797180000000000

- currentRate = 115792089237315616463124798427710789998344941226101297650076671797180000000000 / 1000000000000000000000000 

- balanceOf(address) = _rOwned[address] / currentRate
- balanceOf(recipient) = _rOwned[recipient] / currentRate
- balanceOf(recipient) = 10421288031358457588121388650781911706794298619907650760000000000 / 
