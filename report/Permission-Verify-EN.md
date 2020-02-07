# 权限验证

## 事件经过
主网合约[contract.chainknights](http://cocos-terminal.com/#/contract/contract.chainknights)。

## 漏洞分析
如下部分合约代码,from由用户传入,合约中未加权限验证.
```
function ontransfer(from, amount, memo)

    local tMemo = split(memo)
    assert(#tMemo >= 1, "transfer memo=> transaction type | transaction content")
    local trxType = tonumber(tMemo[1])
    if trxType == TRX_TYPE_DELEGATE then

        assert(#tMemo == 4, "transfer memo=> transaction type | reffer | delegateType | item_id")

        read_list = {public_data={_hero_deles=true,_global=true}}
        chainhelper:read_chain()

        _check_all_switch()

        local reffer = tMemo[2]
        local delegateType = tonumber(tMemo[3])
        local item_id = tMemo[4] 
        reffer = TEAM

        assert(delegateType == DELEGATE_TYPE_HERO, "invalid delegateType")

        u = {}
        u.id = next_hero_dele_id()
        u.player = from
        u.item_id = item_id
        u.stake_amt = amount / 10
        public_data._hero_deles[from] = u

        write_list = {public_data={_hero_deles=true}}
        chainhelper:write_chain()

    elseif trxType == TRX_TYPE_BUY then

        local item_type = tonumber(tMemo[2])
        local sell_id = tMemo[3]
        local seller = tMemo[4]
        local item_guid = tMemo[5]
        local price = tonumber(tMemo[6])
        local count = tonumber(tMemo[7])

        buy(from, item_type, sell_id, seller, item_guid, price, count, amount)

    elseif trxType == TRX_TYPE_USER_UPGRADE then
        user_upgrade(from, amount)
    elseif trxType == TRX_TYPE_SYSTEM_BUY then
        local mystery_shop_id = tonumber(tMemo[2])
        system_buy(from, mystery_shop_id, amount)
    else
        assert(false, "trxType was not supported")
    end

    chainhelper:transfer_from_caller(contract_base_info.owner, amount, IN_TOKEN_SYMBOL, true)

end

```
攻击者就可以利用这个漏洞进行攻击,覆盖其他玩家的订单。基本的攻击手法如下：
1. 监听其他玩家的合约调用。
2. 在其他玩家创建订单之后,攻击者在调用合约修改订单,导致玩家交易失败

具体的攻击手段在[这里](http://cocos-terminal.com/#/block-trx/094267ec73b76cf3e1f9716e4a7684dea18889b4df4d2e9683b76aeef66dc2f5)


## 防御方法

### 风险操作增加权限校验