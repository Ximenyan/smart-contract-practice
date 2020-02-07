# 数值验证

## 事件经过
主网合约[contract.chainknights](http://cocos-terminal.com/#/contract/contract.chainknights)。

## 漏洞分析
如下部分合约代码,未校验玩家转账的金额是否为0.
```
     -- 购买
     local function buy(buyer, item_type, sell_id, seller, item_guid, price, count, qty)
     
         local now_time = chainhelper:time()
     
         
         if item_type == ITEM_TYPE then 
             
             assert(math.floor(price * count) == math.floor(qty), "##erroCode##20024")
             
             read_list = {public_data={_item_buys=true,_global=true}}
             chainhelper:read_chain()
     
             _check_all_switch()
     
             local u = public_data._item_buys[buyer];
     
             if u ~= nil then
                 -- assert((now_time > u.created_at) and ((now_time - u.created_at) <= COOL_TIME_INTERVAL), "time was not cooldown")
             else
                 assert(keepTableSpace(public_data._item_buys) == true)
             end
             u = {}
             u.id = next_item_id();
             u.buyer = buyer
             u.sell_id = sell_id
             u.seller = seller
             u.item_guid = item_guid
             u.price = string.format("%.4f", price / COCOS_ACCURACY) .. " COCOS"
             u.count = count
             u.created_at = now_time
             public_data._item_buys[buyer] = u
             
             write_list = {public_data={_item_buys=true,_global=true}}
             chainhelper:write_chain()
     
         elseif item_type == HERO_TYPE then 
     
             assert(math.floor(price * count) == math.floor(qty), "hero payment is not satisfied")
             read_list = {public_data={_hero_buys=true,_global=true}}
             chainhelper:read_chain()
     
             _check_all_switch()
     
             local u = public_data._hero_buys[buyer]
     
             if u ~= nil then
                 -- assert((now_time > u.created_at) and ((now_time - u.created_at) <= COOL_TIME_INTERVAL), "time was not cooldown")
             else
                 assert(keepTableSpace(public_data._hero_buys) == true)
             end
             u = {}
             u.id = next_hero_id()
             u.buyer = buyer
             u.sell_id = sell_id
             u.seller = seller
             u.item_guid = item_guid
             u.price = string.format("%.4f", price / COCOS_ACCURACY) .. " COCOS"
             u.count = count
             u.created_at = now_time
             public_data._hero_buys[buyer] = u
     
             write_list = {public_data={_hero_buys=true,_global=true}}
             chainhelper:write_chain()
         else 
             assert(false, "buy invalid item_type")
         end
     
     end

```
攻击者就可以利用这个漏洞进行攻击,不停的用0金额去创建订单：
具体的攻击手段在[这里](http://cocos-terminal.com/#/block-trx/5f509d055fa89d8e554aa5ed1d46e168efa8f9f3d161bd5064e920dfe7c35e7c)


## 防御方法

### 增加严格的数值校验,不可相信玩家传入的数据