BPZ

High


## Always revert due to not defining inequality properly. Eventually it DOS when shortToken address = longToken address. So No user can deposit anymore because of the flawed conditional. 

## Summary

DOS in getAdjustedLongAndShortTokenAmounts() function due to not setting inequality properly. 


## Vulnerability Detail

getAdjustedLongAndShortTokenAmounts() function is always reverted . Reason is diff is always negative with current implementation. 
(diff is zero only if poolLongTokenAmount = poolShortTokenAmount). So it will revert the function.

## Code Snippet

      392      if (poolLongTokenAmount < poolShortTokenAmount) {
      393                 uint256 diff = poolLongTokenAmount - poolShortTokenAmount;
   
      401      } else {
      402      uint256 diff = poolShortTokenAmount - poolLongTokenAmount;
   
   Also here adjustedLongTokenAmount is not assigend properly. (Not assigning here. Just substraction only)

      406.        adjustedLongTokenAmount - longTokenAmount - adjustedShortTokenAmount;
   
   
https://github.com/0x00012345/gmx-synthetics/blob/8028cb8022b85174be861b311f1082b5b76239df/contracts/deposit/ExecuteDepositUtils.sol#L393

https://github.com/0x00012345/gmx-synthetics/blob/8028cb8022b85174be861b311f1082b5b76239df/contracts/deposit/ExecuteDepositUtils.sol#L402

https://github.com/0x00012345/gmx-synthetics/blob/8028cb8022b85174be861b311f1082b5b76239df/contracts/deposit/ExecuteDepositUtils.sol#L406


## Impact
getAdjustedLongAndShortTokenAmounts() function is always reverted . So it will not return adjustedLongTokenAmount and adjustedShortTokenAmount
as expected. Becaouse of this market will not able to receive additional deposits when the respective market.shortToken = market.longToken,
for example the WETH/WETH market.

Here is the impact step by steps :

User calls createDeposit with a market where longToken = shortToken (we can use WETH for example) as a param for the deposit in the
ExchangeRouter contract

![image](https://user-images.githubusercontent.com/118436384/227443003-c926ea16-c32c-4dc6-8645-2a5fa9801081.png)

https://github.com/0x00012345/gmx-synthetics/blob/8028cb8022b85174be861b311f1082b5b76239df/contracts/router/ExchangeRouter.sol#L115

Then orderKeeper will attempt to execute this deposit in the DepositHandler contract

![image](https://user-images.githubusercontent.com/118436384/227443982-2df66b97-99ad-47a3-874c-8c7c364cd65e.png)

https://github.com/0x00012345/gmx-synthetics/blob/8028cb8022b85174be861b311f1082b5b76239df/contracts/exchange/DepositHandler.sol#L92

_executeDeposit function is then invoked which invokes ExecuteDepositUtils.executeDeposit(params):

![image](https://user-images.githubusercontent.com/118436384/227445886-1b35d296-84d0-43ce-ba0a-4ae9da13c29c.png)

https://github.com/0x00012345/gmx-synthetics/blob/8028cb8022b85174be861b311f1082b5b76239df/contracts/exchange/DepositHandler.sol#L174

Within  ExecuteDepositUtils.executeDeposit, this condition will be true because   longToken = shortToken for this deposit and
getAdjustedLongAndShortTokenAmounts will be invoked:

![46547565465464](https://user-images.githubusercontent.com/118436384/227447434-a934d9a6-f72b-418e-a009-3b39c0b84d7a.PNG)



Then, with-in getAdjustedLongAndShortTokenAmounts, if for the selected market (WETH/WETH) poolLongTokenAmount < poolShortTokenAmount, the
transaction will revert due then reason mention above. 

![image](https://user-images.githubusercontent.com/118436384/227448209-aa596cab-59f0-416e-b0f6-6bdacfb7743e.png)

https://github.com/0x00012345/gmx-synthetics/blob/8028cb8022b85174be861b311f1082b5b76239df/contracts/deposit/ExecuteDepositUtils.sol#L393

## Tools Used

Manual auditing 

## Recommended Mitigation Steps

diff should be declared in a other way. 

      393      uint256 diff = poolShortTokenAmount - poolLongTokenAmount ;
 
      402     uint256 diff =  poolLongTokenAmount - poolShortTokenAmount ;



