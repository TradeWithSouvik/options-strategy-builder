# Options Strategy Builder

### We support Nifty and Banknifty Options intraday backtesting from Jan 2019 in 5 minute time frame.
---
## Quickstart

**1. Install**
```shell
    npm install options-strategy-builder
```
**2. Require module**

```javascript
    const builder = require("options-strategy-builder")
```

**3. Run the module**

`await builder(daysToExpiry,script,tradeFunction,intermediateCallbackFunction)`

Where

  * **daysToExpiry** is the days remaining for expiry. 0 would mean expiry day. and 1 would mean a day before expiry
  * **script** is the script you want to backtest on. For now we support "NIFTY" and "BANKNIFTY" as valid script value
  * **tradeFunction** is the function which will define all the trade decisions taken intraday
  * **intermediateCallbackFunction** is the function which update you on all the test results as the backtesting is being performed

**4. Define tradeFunction**

```
const tradeFunction = async(currentTradeData, ocData, context)=>{ 
    let orders=[]
    //WRITE YOUR OWN LOGIC// 
    return {orders,context}
}
```

Where

  * **currentTradeData** is the currentTradeData with pnl and leg position.
  * **ocData** is the option chain data of that particular timestamp
  * **context** is the data which can be used as wished. It is a optional recurrent data which is maintained during a trade session.

## tradeFunction Inputs
**currentTradeData**
```   
    { 
         bookedPnl:number,
         pnl:number,
         legs:{
             <strikePrice>:{
                 ltp:number,
                 quantity:number,
                 opType:<BUY/SELL>
             }
         }
     }
```
**ocData**
```
    {
         currentTimestamp:number,
         expiryTimestamp:number,
         daysToExpiry:number,
         volatility:number,
         lotSize:number,
         futurePrice:number,
         spotPrice:number,
         strikeAtm:number,
         strikeDiff:number,
         optionChainData:{
                 <strikePrice>:{
                             call :{
                                 ltp:number, 
                                 strike:number, 
                                 delta:number, 
                                 vega:number, 
                                 iv:number
                             },
                             put :{
                                 ltp:number, 
                                 strike:number, 
                                 delta:number, 
                                 vega:number, 
                                 iv:number

                             }
                     }
             }
     }
```
**context**
```
    {
        // The context is reset in every trade. Data in the context will only be modified by tradeFunction.
    }
```

## tradeFunction Output
**orders**
```
    [{
        legType:<PUT/CALL>,
        opType:<BUY/SELL>,
        strike:<strike to be executed>,
        quantity:<number of lots>
    },...]
```
**context**
```
    {
        // Put whatever contextual data you wish to use in a trade session. The context is reset in every trade.
    }
```
## Example

```javascript
const builder = require("options-strategy-builder")

const tradeFunction = async(currentTradeData, ocData, context)=>{

    //Example of BANKNIFTY Straddle at 9:20 am with full price sl on both legs
    let orders=[]
    const currentDate=new Date(ocData.currentTimestamp)
    if(!context.isEnd){
        if(currentDate.getHours()==9&&currentDate.getMinutes()==20){
            orders=[{legType:"PUT",opType:"SELL",strike:ocData.strikeAtm,quantity:1},{legType:"CALL",opType:"SELL",strike:ocData.strikeAtm,quantity:1}]
            context.strikeAtm=ocData.strikeAtm
            context.sl=ocData.optionChainData[ocData.strikeAtm].call.ltp+ocData.optionChainData[ocData.strikeAtm].put.ltp
        }
        else if(context.strikeAtm&&currentTradeData.legs[context.strikeAtm]&&currentTradeData.legs[context.strikeAtm].put.pnl<-context.sl){
            orders=[{legType:"PUT",opType:"BUY",strike:context.strikeAtm,quantity:1}]
            context.isEnd=true
        }
        else if(context.strikeAtm&&currentTradeData.legs[context.strikeAtm]&&currentTradeData.legs[context.strikeAtm].call.pnl<-context.sl){
            orders=[{legType:"CALL",opType:"BUY",strike:context.strikeAtm,quantity:1}]
            context.strikeAtm=strikeAtm
            context.isEnd=true
        }
    }
    return {orders,context}
}


;(async()=>{
    try{
        const result=await builder(0,"BANKNIFTY",tradeFunction,(updateData)=>{
            console.log(updateData)
        })
        console.log(result)
    }
    catch(e){
        console.log(e)
    }
    
})()
```


