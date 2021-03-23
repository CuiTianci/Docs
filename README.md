# 订阅笔记
参考文档：https://developer.android.com/google/play/billing/subscriptions

## 客户端与Google Developer Api能够获取到的主要效数据对比

### BillingClient.queryPurchase()


| 状态        |  是否返回？  |   isAutoRenewing  |
| :----:      |    :----:   |   :---:           |
| 订阅有效    | 是          | true               |
| 已取消      | 是          | false              |
| 在宽限期内  | 是          | true               |
| 账号保留    | 否          | 无                 |
| 已暂停      | 否          | 无                 |
| 已到期      | 否          | 无                 |

### Purchases.subscriptions:get

| 状态        |是否返回？|	expiryTimeMillis|	paymentState|	autoRenewing|
| :----:      |    :----:   |   :----:   | :----: | :----:|
有效|	是|	未来|	1（已收到付款）|	True
已取消	|	是	|未来|	1（已收到付款）|	False
在宽限期内	|	是 |	未来 |	0（付款待处理）|	True
暂时保留	|	是 |	过去 |	0（付款待处理）|	True
已暂停	|	是	| 过去 |	1（已收到付款）|	True
已到期	|	是 |	过去 |	1（已收到付款）|	False

### 宽限期
启用宽限期后，如果在结算周期结束时存在付款问题，订阅就会进入宽限期。在此期间，用户应仍有权访问订阅，同时 Google Play 会尝试续订订阅。您可以从 Google Play 管理中心的应用内商品设置中指定宽限期的长度。

### 帐号保留
帐号保留是一种订阅状态，当通过用户的付款方式扣款失败且任何关联的宽限期都已结束而没有付款解决方案时，订阅开始进入这种状态。当订阅进入帐号保留状态后，您应阻止用户访问您的内容或服务。帐号保留期最多持续 30 天。

### 暂停订阅
在您启用暂停功能后，用户可以选择暂停订阅一段时间（介于一周到三个月之间），具体取决于循环周期。启用暂停选项后，它将同时显示在订阅中心和取消流程中。请注意，按年订阅无法暂停，并且一周和三个月的暂停限制随时可能更改。
只有在当前结算周期结束后，订阅暂停才会生效。订阅暂停后，用户将无法访问订阅。在暂停期结束时，订阅将恢复，并且 Google 会尝试续订订阅。如果恢复成功，订阅将再次变为活动状态。如果由于付款问题导致恢复失败，用户将进入帐号保留状态。

### 恢复订阅
已取消的订阅会在到期日期之前继续显示在 Play 商店应用中。用户可以通过在 Google Play 商店应用的订阅部分中点击重新订阅（以前称为恢复），在已取消的订阅到期之前恢复订阅。

## 客户端与Google Developer Api能够获取到的详细效数据对比

### Purchase基本成员变量：
```json
          {
            "orderId":"GPA.3322-5820-0817-24722",
            "packageName":"messages.chat.free.text.messaging.sms",
            "productId":"amessage_one_year_sub",
            "purchaseTime":1615451238680,
            "purchaseState":0,
            "purchaseToken":"khnhmabhgckfnmoimdhjoide.AO-J1OwXT1bTO4rrJII6jZkucYYc9VfzU4jauFK4AZPgWPDbiW7XNsNU86006RJKQp5DlDuZXNV00trt5Kn49mluezXIHxMJCwqd9K39X3EYA8DnjJ2DLRPDJyF_vRbBdc4TLwe1vvKI",
            "autoRenewing":true,
            "acknowledged":false
           }
```

### skuDetails成员变量
```json
{
            "productId":"amessage_one_month_sub",
            "type":"subs",
            "price":"$7.99",
            "price_amount_micros":7990000,
            "price_currency_code":"USD",
            "subscriptionPeriod":"P1M",//订阅期限，1month
            "freeTrialPeriod":"P3D",//免费试用期限，3Days
            "title":"One Month- aMessage (Messages: free texting messages chat app)",
            "description":"One Month- aMessage",
            "skuDetailsToken":"AEuhp4LohMWFjmyeEX2vHwD-Dw_uI3sFbrkv8p682xsts98Pgk0-qPrtmAqGwqhn-Zzi"
        }
```

### Google Developer API
详情见接口文档：https://developers.google.com/android-publisher/api-ref/rest/v3/purchases.subscriptions/get  

### 实时开发者通知

如果您的应用将订阅状态存储在安全的后端服务器上，则应使用实时开发者通知监听状态变化，以确保状态保持同步。系统会针对影响订阅状态的事件（如续订和取消）发送 SubscriptionNotification。您在收到实时开发者通知后需要调用 Developer API(Purchases.subscriptions:get) 来获取完整的状态信息，并更新您自己的后端状态。此类通知只会告知您订阅状态发生了变化，而不会为您提供有关总体订阅状态的完整信息。  
配置链接：https://developer.android.com/google/play/billing/getting-ready#configure-rtdn  


## 升降级补差价策略（官方称之为：按比例计费）
注意特殊情况：IMMEDIATE_AND_CHARGE_PRORATED_PRICE仅适用于单位价格较低向单位价格较高的订阅项的转化，否则无法进行购买。错误信息：Error retrieving information from server.DF-DFERH-01。

| 按比例计费模式	| 说明       |
| :----:      |    :----:   |
| IMMEDIATE_WITH_TIME_PRORATION	| 订阅会立即升级或降级。任何剩余时间都会根据差价进行调整，并通过向前推下一个结算日期，将余额记入新订阅。这是默认行为。 |
| IMMEDIATE_AND_CHARGE_PRORATED_PRICE |	订阅会立即升级，结算周期保持不变。用户随后需要补足剩余订阅期的差价。 |
| IMMEDIATE_WITHOUT_PRORATION |	订阅会立即升级或降级，在订阅续订时将按新价格收取费用。结算周期保持不变。 |
| DEFERRED	| 只有在订阅续订时，订阅才会升级或降级。 |

### 代码示例
```js
val flowParams = BillingFlowParams.newBuilder()
        .setOldSku(previousSku, purchaseTokenOfOriginalSubscription)
        .setReplaceSkusProrationMode(BillingFlowParams.ProrationMode.DEFERRED)
        .setSkuDetails(upgradeOrDowngradeSkuDetails)
        .build();
val responseCode = billingClient.launchBillingFlow(activity, flowParams)
```

### 免费试订或初次体验价优惠期间升级
- 如果对于您的应用中提供的所有订阅，用户只能获得一次免费试订机会，则用户要改用的方案将没有免费试订或初次体验价。（默认）
- 如果您针对每种订阅产品提供一次免费试订机会，则用户要改用的方案可以有免费试订或初次体验价。

### 当新方案和旧方案都有免费试订且用户在免费试订期间升级时各种按比例计费模式的行为

| 按比例计费模式 |  在每个应用中免费试订一次	| 每种订阅产品免费试订一次 |
| :----: | :----: | :----: |
|IMMEDIATE_WITH_TIME_PRORATION	|用户会立即失去免费试订期会根据差价转换为新层级的等效免费期。|	用户会失去级的层的免费试订。
IMMEDIATE_AND_CHARGE_PRORATED_PRICE	|用户会用户随后需要补足剩余订阅期的差价。下一个结算日期保持不变。  注意：此选提高的订阅升级。|同|
IMMEDIATE_WITHOUT_PRORATION |	用户会立即升级到新层级。用户可以一直免费试订新层级的内容，直到上一个结算周期结束。|同|
DEFERRED	| 用户可以一直免费试订旧订阅内容，直到下一个结算日期。| 同|

## 按比例计费建议

|场景|	建议的按比例计费模式|	结果|
| :----: | :----: | :----: |
升级到费用更高的层级 |	IMMEDIATE_AND_CHARGE_PRORATED_PRICE|	用户会立即获得访问权限，而结算周期保持不变。
降级到费用更低的层级|	DEFERRED	|用户已经为费用更高的层级付款，因此他们将保有访问权限，直到下一个结算日期。
更改同一层级的循环周期（例如，从按月订阅更改为按年订阅）|	DEFERRED|	用户将在下一个结算日期按新的周期性价格支付。
在免费试订期间升级 - 保留对免费试订的访问权限|	IMMEDIATE_WITHOUT_PRORATION |	用户会保有对免费试订的访问权限，但试订期的剩余时间会升级到更高的层级。
在免费试订期间升级 - 终止对免费试订的访问权限|	IMMEDIATE_AND_CHARGE_PRORATED_PRICE|	用户会立即获得对新层级的访问权限，但不能再免费试订。

## 关于区分试用与付费
如果购买的订阅提供免费试订，则从 Google Play Developer API 返回的订阅具有 paymentState = 2 属性（免费试订）。如果订阅成功续订，则 paymentState 会改为 1（付款已接收）。

## 后台配置订阅价格发生改变时，应该注意以下问题
- 价格变动一旦确认，便无法撤消。
- Google Play 从确认后七天开始通知用户价格变动。
- 如果订阅价格上调，用户必须在 30 天内批准价格变动，否则他们的订阅会自动取消。
- 如果订阅价格下调，对于现有订阅者，会在下一个续订日期自动应用这一价格变动。对于在价格下调后订阅的所有用户，会自动实行下调后的价格。

启动价格变动确认流程，来提示用户对价格变动进行确认：
```js
val priceChangeFlowParams = PriceChangeFlowParams.newBuilder()
        .setSkuDetails(changedPriceSubscriptionSkuDetails)
        .build()

billingClient.launchPriceChangeConfirmationFlow(activity,
        priceChangeFlowParams,
        object : PriceChangeConfirmationListener() {
            override fun onPriceChangeConfirmationResult(responseCode: Int) {
                if (responseCode == BillingResponseCode.OK) {
                    // User has confirmed the price change.
                } else if (responseCode == BillingResponseCode.USER_CANCELED) {
                    // User hasn't confirmed the price change and should retain
                    // access until the end of the current billing cycle.
                }
            }
        })
```

## 常见问题
### PurchaseToken会在哪些情况下发生变化？
当用户升级、降级或重新订阅时，旧订阅会变为无效，并且系统会使用一个新的购买令牌创建新订阅。
此外，从 Google Play Developer API 返回的订阅资源将包含一个 linkedPurchaseToken，用来指示用户升级、降级或重新订阅时所基于的旧购买交易。您可以使用 linkedPurchaseToken 查询旧订阅并识别现有用户帐号，以便将新购买交易与同一帐号关联。

### OrderId在什么情况下会发生变化。
对于订阅，首次购买交易会创建一个OrderId,升级、降级、替换和重新订阅都会创建新的OrderId。  
订阅续订的订单号包含一个额外的整数，它表示具体是第几次续订。例如，初始订阅的OrderId可能是GPA.1234-5678-9012-34567，后续OrderId是GPA.1234-5678-9012-34567..0（第一次续订）、GPA.1234-5678-9012-34567..1（第二次续订），依此类推。

### 对订阅进行确认（Acknowledge）
订阅完成后（已将订阅状态绑定到用户后）务必对订阅进行确认。如果在三天内未确认购买交易，用户会自动收到退款，并且 Google Play 会撤消该购买交易。

### 测试环境中的订阅周期
注意：测试订阅最多可续订六次。

生产订阅期	| 测试订阅续订
 :----: | :----:
1 周	| 5 分钟
1 个月	| 5 分钟
3 个月 |	10 分钟
6 个月 |	15 分钟
1 年 |	30 分钟

### 订阅升降级是否存在次数限制？
答：不存在，官方文档未提及该限制+测试结论。

### 是否存在产生多个订阅的可能？
答：有可能，Android的订阅没有“组”的概念，每个订阅项都是一个独立的商品。在进行订阅时，BillingFlowParams.Builder.setOldSku(sku,purchaseToken)为可选项，若未调用该方法，则认定当前订阅为一个全新的订阅。若调用该方法，并将已存在的订阅相关参数传入，则会进行订阅的升降级流程。  
因此，当已存某个订阅，在购买另外一个订阅项时，未调用setOldSku方法，则会产生两个不相干的订阅。

||订阅a、订阅A属于同一类服务，级别不同 | 订阅X、订阅Y属于不同服务 |
 :---: | :---: |:---:|
|例子| a、A均为去广告服务，a订阅期1个月，A订阅期1年。	| X为去广告服务，Y为扩展功能服务。
是否可共存？ | 否  | 是
需要升降级？ | 是	| 否
存在订阅，购买另外一个订阅项过程 | 调用setOldSku方法 | 不调用setOldSku方法
存在订阅，购买另外一个订阅项结果 | 订阅发生升降级 |	同时存在X、Y个订阅 |

## 两个暂未确认的问题
以下两个问题在官方文档中未提及，为个人测试结果，仅供参考。会持续对各种情况进行测试，并查阅更多相关文档，并对此进行完善。

### 关于订阅缓存
个人测试过程中，订阅缓存并不明显，客户端订阅失效（即查询不到）时间与订阅到期时间基本一致。

### 多账号问题
个人测试过程中，其中一个账户存在订阅时，客户端即可查询到。暂未测试多个账户都存在有效订阅的情况。

