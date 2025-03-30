WAR API



--本插件是依靠Tinkr和魔兽API指令编写的，可以使用这两者的所有指令，
--魔兽插件不识别任何中文函数和中文变量，脚本中不能出现任何中文函数和中文变量，输出的函数参数可以为中文
--魔兽API英文地址
--https://wowpedia.fandom.com/wiki/Global_functions/Classic
--魔兽API中文地址
--https://wow.battlenet.top/api/view/api
--Tinkr API地址
-- https://docs.tinkr.site/Lua/
--Tinkr的代码运行机制不同于EWT和minibot，所有使用GL内置的指令都要在指令前添加 BF.
--======延时============================================================================================================================================
{

    --等待延时 sec为毫秒，
    function BF.WaitTime(sec)   end
    --延时等待，等待秒数
    --second等待秒数
    function BF.Delay(second)	end
    --取启动间隔，如果上次触发这个事件的时间间隔秒数大于参数2，返回真，否则返回假
    --参数1 触发标签
    --参数2 秒数
    function BF.TimeDelayTrue(sign,second) end
    --如果一个信息输出很频繁，可以使用此函数来控制输出频率
    --如果 print("每隔5秒输出")是每帧输出，这样设置之后就变为每5秒输出一次
    if BF.TimeDelayTrue("调控输出",5) then
        print("每隔5秒输出")
    end


}
--======移动============================================================================================================================================
{
    --停止移动
    function BF.StopMoving() 
    --使用导航寻路到x,y,z
    --range 停止距离 默认3,可省略
    --callback 回调函数 可省略
    function BF.NavTo(x,y,z,range,callback)
    
    --例子，
    --直线移动到某个坐标,2D坐标符合,r为停止距离
    function  BF.Move2D(x,y,z,r)
    --直线移动到某个坐标,3D坐标符合
    function  BF.Move3D(x,y,z,r)
    --基础指令向某个坐标移动    
    function  MovTo(x,y,z)
    --如果寻路不到，可以按指定路径移动，这个指令就是按指定表格设定的路径寻路移动，
    --参数1-3 坐标,
    --参数4,表，按指定坐标链接移动,除了添加前3个坐标外，还可以添加回调函数 
    --参数4也可以是文本型，去掉后缀.txt,文本放置在tinkr目录/scripts/GL/GLWayPoint 里
    function BF.ListMove(x,y,z,list)
    -- 例子，这样就会安装指定坐标链，移动到-487.37, -1990.92, 94.31 这个坐标
        BF.ListMove(-487.37, -1990.92, 94.31,{
            { X=-223.42893981934, Y=-2173.7546386719, Z=91.666732788086 },
            { X=-164.22750854492, Y=-2115.501953125, Z=91.667961120605 },
            { X=-116.28896331787, Y=-2093.1020507813, Z=93.472412109375 },
            { X=-161.43339538574, Y=-2050.7419433594, Z=94.689636230469 },
            { X=-299.87240600586, Y=-1834.9588623047, Z=94.45728302002 },
            { X=-288.55242919922, Y=-1794.9470214844, Z=92.543502807617 },
            { X=-337.22882080078, Y=-1775.3216552734, Z=92.099708557129 },
            { X=-478.80490112305, Y=-1759.7982177734, Z=91.803001403809 },
            { X=-531.47955322266, Y=-1790.4780273438, Z=92.660980224609 },
            { X=-469.97775268555, Y=-1836.6502685547, Z=94.65503692627 },
            { X=-484.87924194336, Y=-1935.7020263672, Z=92.193954467773 },
            { X=-487.37191772461, Y=-1990.9291992188, Z=94.318382263184 },
            { X=-460.26556396484, Y=-2027.4305419922, Z=93.31413269043 },
            { X=-465.38900756836, Y=-2059.1713867188, Z=91.666931152344 },
            { X=-398.56198120117, Y=-2063.0234375, Z=92.478950500488 },
        },)
    --按指定表格设定的路径移动，直线移动不寻路，参数4表格添加Nav=true，代表可以寻路移动
    --参数1-3 坐标,
    --参数4,表，按指定坐标链接移动,除了添加前3个坐标外，
    --参数4也可以是文本型，去掉后缀.txt,文本放置在tinkr目录/scripts/GL/GLWayPoint 里
    function BF.ListMove(x,y,z,list)  
    --例子  
        BF.ListMoveS(1621.3694, -4402.5122, 12.3082,
        {
            { X = 1525.6708, Y = -4212.0366, Z = 41.0641,Nav=true},
            { X = 1605.1382, Y = -4265.6895, Z = 47.4190,Nav=true},
            { X = 1620.3325, Y = -4305.8208, Z = 21.5845 },
            { X = 1608.6796, Y = -4333.2847, Z = 1.4929 },
            { X = 1604.7950, Y = -4385.4365, Z = 10.0006 },
            { X = 1608.4728, Y = -4391.8159, Z = 10.0625 },
        })    
}
--======技能释放==============================================================================================================================
{
    --单独开启自建循环
    GL.OpenSelfRotation=true
    --运行宏，和魔兽宏一样的效果
    function RunMacroText(Macro)
    --释放指定ID技能,id为数字
    function CastSpellByID(ID)
    --释放指定名字技能,name为技能名称，参数为文字
    function CastSpellByName(name)
    --点击地面坐标，用于释放暴风雪
    function Click(x,y,z)
      --操作技能循环开关  
    function  BF.SetRotation(name, value)   
    --关闭猎人印记
    BF.SetRotation('猎人印记', false) 
    --对象buff或者debuff剩余时间
    --unit 对象 player或者target
    --idOrName buff Id 或者名字
    function BF.AuraRemain(unit,idOrName)     气氛，氛围；
}
--======角色操作类==============================================================================================================================
{
    --角色面向,可以面向角度，物体，坐标
    --如面向目标 BF.SetFace("target")
    --如面向坐标 BF.SetFace(-487.37, -1990.92)
    --如面向角度 BF.SetFace(2.4185206890106)
    function BF.SetFace(a,b,c) 
    

--跳跃
    function JumpOrAscendStart()
    --重置副本
    function ResetInstances()
    --采集使用物品，如门，传送门等,参数为数字
    function BF.Collect(物品id)
    --使用炉石
    function BF.UseHearthStone()
    --战斗中自动使用药水,饰品。在恢复设置。
    function BF.UsePotionTrinket()
    --单独使用饰品函数为：
    function BF.UseTrinket(13) 
    function BF.UseTrinket(14)
    --取出公会银行所有物品
    --参数1 单独取出物品名称 可空
    --参数2 物品品质 可空
    --参数3 取到剩余多少背包空格，默认为3 可空
    function BF.GetGuildBankItem(item,Rarity,freebags)
    --提取所有物品
    BF.GetGuildBankItem()
    --提取单个物品
    BF.GetGuildBankItem("北地皮碎片")
    --提取多个物品
    BF.GetGuildBankItem("水之结晶,空气结晶")
    --按品质提取物品
    BF.GetGuildBankItem(nil,3)
    --参数1：雕文id或者名称
    --参数2：slot
    --大型雕文 1,4,6
    --小型雕文 2,3,5
    function BF.EquipGlyph(NameOrID,slot) end
    

    --按界面公会存储设置存储物品
    function BF.GuildBankDeposit()
    --获取物品采集次数
    --物品id，可省略，不填写获取总采集次数，填写获取指定物品采集次数
    function BF.GetCollectCount(objid)
      --学习技能 参数1 技能ID或技能名字  参数2  要学技能等级 如果不传入参数 则所有学习所有等级，需要打开学习技能NPC
    function BF.LearnSpell(id,nlevel) 
    function BF.EquipBag()
        
    --分解装备，LimitRarity为物品品质，breakbind为是否分解不绑定装备
    --BreakDownEquipment(4,true)
    function BF.BreakDownEquipment(LimitRarity,breakbind)
     
    --获取在那个界面天赋加点最多
    --返回 1,2,3
    function BF.GetTalentTabIndex() 
     --设置pawn换装天赋，只设置了圣骑士,其他职业需要自己设置
    function BF.SetPawn_Talent()
        --GL默认选择属性权重，Pawn本职业天赋从上到下为1到N
            BF.Pawn_Talent={
                ["MAGE"] = 1,--法师 冰霜
                ["SHAMAN"] = 1,--萨满 元素
                ["PRIEST"] = 2,--牧师 暗影
                ["WARLOCK"] = 1,--术士 恶魔学识
                ["HUNTER"] = 3,--猎人 野兽控制
                ["PALADIN"] = 1,--圣骑士 惩戒
                ["ROGUE"] =2,--潜行者 奇袭
                ["DRUID"] =3,--德鲁伊 野性伤害
                ["WARRIOR"] =1,--战士 武器
                ["DEATHKNIGHT"] =3,--死亡骑士 鲜血
            }
            local index=BF.GetTalentTabIndex()
            if BF.Me.Class=="PALADIN" then
                if index==3 then
                    BF.Pawn_Talent.PALADIN=1
                elseif index==1 then
                    BF.Pawn_Talent.PALADIN=2
                elseif index==2 then
                    BF.Pawn_Talent.PALADIN=3	
                end
            end
     end   
    --判断装备是否比身上或者背包中的好
    --参数1:物品链接
    function BF.IsEquipmentBetter(link)  
    --自动装备更好的装备，请安装pawn插件
    function BF.AutoEquipment()
    --上马
    function BF.Mount.MountUp()
     --下马
    function BF.Mount.Dismount()
     --重置副本
    function ResetInstances()  
    --判断队友是否在身边
    --参数1：队友名称，可空，不填写判断所有队友是否在身边，填写判断指定队友是否在身边
    --参数2：范围，可空，默认60
    function BF.IsTeammateNearby(teammateName,range)
}
--======选中怪物，怪物判断，对象管理=========================================================================================================================
{
    --函数作用，选中指定坐标点范围内指定ID的怪物，并且决定是否面向
    --x, y, z 坐标 数字
    --range范围 数字
    --unitId 怪物ID，数字
    --faceNow 是否面向，true false
    function SelectUnitByLocationAndId(x, y, z, range, unitId, faceNow)
        local target =
            BF.ObjectManager.GetWoWUnit(
            function(unit)
                return unit.ObjectID == unitId and unit:DistanceTo(x, y, z) <= range and not unit.Dead and
                    UnitCanAttack("player", unit.Pointer) and
                    unit.GUID ~= BF.Me.GUID
            end,
            function(a, b)
                if a ~= nil and b ~= nil then
                    return (a.Distance < b.Distance)
                end
            end
        )
        if target then
            --BF.Log.Write('选择并面向目标 = %s', target.Name)
            target:TargetUnit()
            if faceNow then
                target:Face()
            end
            return target
        end
        return false
    end
    --函数作用，选中指定坐标点范围内的怪物，并且决定是否面向
    --x, y, z 坐标 数字
    --range范围 数字
    --faceNow 是否面向，true false
    function SelectUnitAtLocation(x, y, z, range, faceNow)
        local target =
            BF.ObjectManager.GetWoWUnit(
            function(unit)
                -- print(unit:DistanceTo(x,y,z), UnitCanAttack("player", unit.Pointer))
                return unit:DistanceTo(x, y, z) <= range and not unit.Dead and UnitCanAttack("player", unit.Pointer) and
                    unit.GUID ~= BF.Me.GUID
            end,
			function(a, b)
                if a ~= nil and b ~= nil then
                    return (a.Distance < b.Distance)
                end
            end
        )
        if target then
            -- BF.Log.Write('选择并面向目标 = %s', target.Name)
            target:TargetUnit()
            if faceNow then
                target:Face()
            end
            return target
        end
        return false
    end
    --获取指定坐标范围的怪物数量
    --x, y, z 坐标
    --range 范围
    function  BF.ObjectManager.GetMonsterNumber(x, y, z, range)

    --获取对象和单位函数
    --根据指定条件获取指定多个NPC
    --参数1条件，
    --参数2 排序函数
    function BF.ObjectManager.GetWoWUnits(condition, orderBy)
    --根据指定条件获取指定单个NPC
    --参数1条件，
    --参数2 排序函数
    function BF.ObjectManager.GetWoWUnit(condition, orderBy)	
    
    --选中指定位置，指定范围的指定ID的怪物，是否面对
    --x,y,z 坐标
    --range 范围
    --unitid 怪物id
    --facenow 是否面对怪物
    function BF.SelectUnitByLocationAndId(x, y, z, range, unitId, faceNow)
        local target =
            BF.ObjectManager.GetWoWUnit(
            function(unit)
                return unit.ObjectID == unitId and unit:DistanceTo(x, y, z) <= range and not unit.Dead and
                    UnitCanAttack("player", unit.Pointer) and
                    unit.GUID ~= BF.Me.GUID
            end,
            function(a, b)
                if a ~= nil and b ~= nil then
                    return (a.Distance < b.Distance)
                end
            end
        )
        if target then
            --BF.Log.Write('选择并面向目标 = %s', target.Name)
            target:TargetUnit()
            if faceNow then
                target:Face()
            end
            return target
        end
        return false
    end
    --获取所有存活的可攻击和攻击自己的单位数量
    function getLiveUnitNum()
        local enemies100y =
            BF.ObjectManager.GetWoWUnits(
            function(unit)
                return not unit.Dead and UnitCanAttack('PLAYER', unit.Pointer) and UnitAffectingCombat(unit.Pointer) 
            end,
            function(x, y)
                return x.Distance < y.Distance
            end
        )
        return #enemies100y
    end
    --获取自身属性各项值
    function TableToString(BF.Me)
    --遍历周围单位,怪物等。
    function GetUnitsInfo()
        for _, Unit in pairs(BF.Units) do
           BF.TableToString(Unit)
        end
    end
    --遍历周围物体,采集类
    function GetGameObjectInfo()
        for _, Object in pairs(BF.GameObjects) do
           BF.TableToString(Object)
        end
    end
    --如主程序造成帧数下降，可以关闭以下遍历()
    --关闭技能循环
    BF.AutoFight(true)
    --关闭主程序遍历npc
    BF.StopOM_Npcs=true
    --关闭主程序遍历玩家
    BF.StopOM_Players=true
    --关闭主程序遍历周围游戏物体
    BF.StopOM_GameObjects=true
    --关闭全部遍历,不建议全部关闭
    BF.StopOM=true
} 
--======消息输出===========================================================================================================================================
{
    --魔兽自带打印消息
    function print(msg)
    --GL输出绿色文字 
    function BF.pmsg(msg)
    --屏幕输出文字
    --msg:文字信息
    --size:文字大小
    --在屏幕中间输出字符
     --文字颜色变量
    --'|cFF'+颜色16进制+msg 例子
    --BF.Log.Frame("|cFF8B0016 密语报警", 200)
    function BF.Log.Frame(msg,size)
    --屏幕上方输出文字
    --参数1,文字1
    --参数2,文字2
    --参数3，播放声音,数字型,1-7
    --参数4,文字消失时间，默认2.5秒
    function BF.Alert(message1,message2,sound,fadetime)    
    --以下皆是聊天框输出各种文字类信息，在综合设置-综合设置-聊天框文字输出级别可以选择那些文字不输出
    function BF.Log.UIEr(msg) 
    function BF.Log.Write(msg, ...)  
    function BF.Log.Debug(msg, ...)
    function BF.Log.Warning(msg, ...)
    function BF.Log.Path(msg, ...)
    --在主程序界面下方输出msg信息
    function BF.Log.Status(msg)
    function BFKern.Core.Info(msg)
      
}
--======距离判断=======================================================================================================================================
{
    --计算两个单位直接的距离,如计算自己和目标之间的距离
    --local targetDistance=BF.GetDistanceBetweenObjects("player","target")
    BF.GetDistanceBetweenObjects= function(obj1,obj2) 
    --计算两个坐标直接的3D距离
    BF.GetDistanceBetweenPositions = function(X1, Y1, Z1, X2, Y2, Z2)  
    --计算两个坐标直接的2D距离
    BF.GetPlaneDistanceBetweenPositions= function(X1, Y1, X2, Y2) 
}
--======NPC交互==============================================================================================================================
{
    --对话NPC,参数为NPCID或NPC名称
    function BF.OpenNpc(id)
    --商店购买物品，需要打开商店，用BF.OpenNPC(idOrName)
    --参数1：参数为物品ID或物品名称
    --参数2：参数为数量
    --参数3：是否强制购买，不判断背包数量
    function BF.ShopBuy(NameOrId,count,force)
     --从NPC处购买装备，
    --参数1 npc的名称或者id
    --参数2 荣誉值超过多少可以购买，可省略
    function BF.BuyEquipmentFromNpc(idOrName,Honor)   
    --获取装备的pawn分数
    --参数1 装备链接
    --参数2 ”player“,可省略，如果插件pawn不返回装备分数,设置为player返回装备等级
    function BF.GetEquipmentScore(link,sign)  
    --遍历从npc处购买装备
    function BF.BuyEquipment()  
    --获取背包中有的指定部位的装备分数，没有返回0
    --参数 position 装备位置1到19
    function BF.GetBagEquipmentScore(position)
        
    --打开邮箱后按综合设置里的配置邮寄
    function BF.AutoVendor.AutoMail()
    --打开NPC后，按主程序设置修理售卖物品
    function BF.AutoVendor.DoRepair()
    --交通飞行('8018',"暴风城，艾尔文森林",对话选择项)
    function BF.TakeTaxi(NpcId,FlyNodeName)
    --收取所有邮件，在邮箱附近使用，参数为剩余多少背包空格停止收取，默认为1
    function BF.TakeMail(FreeBagSlots)
}
--======背包物品操作==============================================================================================================================
{
    --换背包耐久度更高的装备
    --参数1：物品在身上的位置1到19，默认17：盾牌
    --参数2：耐久度1到100，默认10
    function BF.Ex_HigherDurabilityItem(slot,durabilityPercent)
    --获取背包空格
    function BF.GetFreeBagSlots()
    --获取背包物品数量
    --nameorid参数为：物品名或者物品ID，
    function GetItemCount(NameorId)
    --使用背包物品,参数为数字id
    function BF.UseItem(物品id) 
    --销毁背包物品,参数为物品id 
    function BF.Bag.DeleteItem(id)
    --获取物品冷却时间，参数为物品id   如获取炉石冷却时间
    function GetItemCooldown(6948) 
}  
--======条件判断,脚本切换==============================================================================================================================
{
    --获取当前地图名称
    function GetZoneText()
    --获取角色声望等级 如中立，后续返回其他参数 BF.GetReputation(‘加基森’)
    function BF.GetReputation(szName)
    --无参数，获取今天星期几，返回数字，周一为1，依次类推， 
    --有参数，为年月日判断某年某月某日为星期几
    function BF.GetWeekDay(y,m,d)
    --获取今天在线游戏时间，返回秒
    function BF.GetTodayGameTime()
    --获取此时电脑时间，返回时分，如7点5分为，0705
    function BF.GetComputerTime()
    --获取当前脚本名称，如：GL-Tinkr/脚本编写说明/脚本切换例子初始脚本.lua
    function BF.GetScriptName()
    --获取当前脚本运行时间：精确到秒，卡位重启重新计算，重新运行重新计算
    function BF.GetScriptTime()

    --切换脚本添加参数，
    --参数1，脚本名称
    --参数2，是否从上次记录任务序号开始，默认否
    --参数3，任务序号，默认从插件保存开始，填序号从指定序号开始
    function BF.ExchangeScript(scriptName,fromor,index)
    --切换指定脚本，参数为脚本路径名
    --BF.ExchangeScript('GL-Tinkr/被GM传送后要执行的脚本/TeleAction.lua')
    --如果想卡位多次后改为切换脚本，添加这个函数到脚本,或者切换脚本代码处
    function BF.RestartChangeScript()
        BF.ExchangeScript("GL-Tinkr/部落诺森德任务/部落灰熊丘陵73-75.ol")
    end
}
--======生活技能交易行操作==============================================================================================================================
{
    --参数1 人物名字
    --参数2 交易物品名字
    --参数3 交易物品数量
    --参数4 交易超时时间 
    --参数5 交易金钱，交易金钱暂时不行
    function BF.TradePlayer(name,stname,ammount,limittime,money)
     --取消所有交易行自己的的非最低价物品
    function  BF.AuctionCancelHigherPrice()
    --取消所有交易行拍卖物品，需要打开交易行
    function BF.CancelAuction() 

    --生活技能练习附魔用
    --spellid,生活技能id
    --name,附魔名称或者技能ID，可以通过编写调试-技能概览获取生活技能ID
    --equipid,要附魔的装备位置1到19
    function BF.DoTradeFM(spellid,name,equipid)
    ----参数1，生活技能ID
    --参数2，配方名称或者id可以通过编写调试-技能概览获取生活技能ID
    --参数3，重复次数 默认1,注意在脚本中，此次数填写大于1时，你后续有操作时，你需要加延时。
    function BF.DoTrade(spellid,name,times)
    --交易行购买物品
    --tname,要购买的物品名称或者id
    --amount,购买数量,默认1,
    --rolename,指定购买人,默认无
    --keepgold,保留铜币,默认99金，99*10000
    --protectprice,保护价格铜，默认99金,超过这个价格不购买，99*10000
    --mustbuygold,必定购买价格,价格低于此值必定购买，铜
    --wait,购买前等待时间,秒,默认1
    --sort,排序函数 例子
        -- function()
        --     BF.Log.D("默认最低价排序,GL default sort")
        --     SortAuctionClearSort('list')
        --     SortAuctionSetSort('list', 'unitprice', false)
        -- end
    function BF.AuctionBuy(tname,amount,rolename,keepgold,protectprice,mustbuygold,wait,sort)
    
      --交易行拍卖物品
    --szname,要购买的物品名称或者id
    --limitprice,限制价格,拍卖行比较价格低于此价格,将以此价格出售，否则以拍卖行比较价格出售,不设置默认最低
    --waittime,拍卖前等待时间,默认为1秒
    --index 拍卖行比较价格的序号，默认最低价格,取值1到11，设置为2,将以拍卖行第二低价格和limitprice比较
    --amount,拍卖数量,默认全部,
    --force,强制使用限制价格挂交易行，默认否
    --ignoreself，是自己的不出售，默认否
    --useset,采用拍卖脚本的部分设置，默认否
    function BF.AuctionSell(szname,limitprice,waittime,index,amount,force,ignoreself,useset)
    
    --交易行获取物品价格
    --tname,要查询的物品名称或id
    --wait,查询后等待时间,秒，数字，可省略
    --index,查询第几个的价格，数字，小于50，可省略
    --sort,可省略，默认排序，可以是排序函数 默认排序是 function() SortAuctionClearSort('list') SortAuctionSetSort('list', 'unitprice', true) end 	
    function BF.GetAuctionPrice(tname,wait,index,sort)
    
    --取消交易行拍卖物品，需要打开交易行界面，在副本中运行
    function BF.CancelAuction()
    
    --获取所有生活技能佩服名称和ID
    function  BF.GetTradeSkillSpellID()
}
--======文件读写==============================================================================================================================
{
    --写角色配置，路径在 解锁器文件夹/configs/服务器-角色名
    --参数1：配置项
    --参数2：值
    function BF.WriteRoleConfig(configure, value)
    --参数1：配置项 返回你写入的配置项的内容
    function BF.ReadRoleConfig(configure)
}
--======其他函数==============================================================================================================================
{
    --指定坐标画半径为多少的圈，几秒后消除
    --参数都为数字
    --r 范围
    --time 消除时间
    function BF.Draw(x,y,z,r,time)
    --保护魔兽禁止API,避免游戏崩溃。
    --参数1，函数名
    --参数2，不定参数，填入你调用的函数的参数，用逗号分隔。
    --例子：BF.ProtectFun("CastSpellByID",6603)
    function BF.ProtectFun=function(str,...)
    --打印表格信息
    function BF.TableToString( tbl , level, filteDefault)
    --防止爆本指令，如果距离倒数第4次重置副本不足3630秒则等待，上次重置副本时间是BF插件自动记录的，则会等待到时间，避免1小时刷本超过5次。
    --sec 爆本限制时间 默认3630秒
    --判断倒数第几次的时间
    --fbcount 副本次数大于第几次时开始判断
    function BF.AvoidBoom(sec,jgcount,fbcount)
    -- 该函数的作用是平滑路线，使录制的路线不会形成尖锐的转向。
    -- points: your waypath，路径点
    --angel: Defualt is 160,1-170°，角度小于这个值，就形成新的用于平滑路径的点,
    --radius:Defualt is 3, number，形成距离原来点的距离，请注意要小于原来点2到点3的距离
    function BF.SmoothPoints(points, angel, radius)


}
--======怪物属性判断==============================================================================================================================
--如果选中了一个怪物，就可以用 BF.Me.Target.Name ,获取怪物名称
BF.Me.Target=
{

    GUID = Creature-0-4891-1-1591-5453-000205A2B2
    ObjectID = 5453
    Pointer = Creature-0-4891-1-1591-5453-000205A2B2
    HealthMax = 2673
    IsTargetingPartyMember = false
    PosX = -9110.4169921875
    Attackable = true
    Quest = false
    Dead = false
    CombatReach = 1.5
    PosY = -4134.9306640625
    Distance = 95.973621216209
    Health = 2673
    --是否在视野中
    LoS = true
    Friend = false
    TargetIsInMyParty = false
    IsTargetingMeOrMyPet = false
    IsTargetingMyPet = false
    IsTargetingMe = false
    HasTarget = false
    Target = false
    TargetIsMe = false
    IsTargetingMeOrMyPetOrPartyMember = false
    TargetIsMyPet = false
    Distance2D = 90.376158513629
    IsVisible = true
    Trackable = false
    Facing = false
    ValidEnemy = false
    Level = 49
    Moving = true
    RealHealth = true
    NextUpdate = 306117.0415
    Player = false
    Name = 哈扎里掘洞蝎
    Position = table: 0x29d42ca90
     {
      Y = -4134.9306640625
      X = -9110.4169921875
      IsUnit = false
      Z = 12.003898620605
     }
    CreatureType = 10
    PosZ = 12.003898620605
    HP = 100
    InCombat = false
}
--======角色登录类==============================================================================================================================
{
    --在脚本中如何控制自动登录：
    --退出游戏
    function	BF.Logout() end
    --设置登录账号为换号账号
    function BF.WriteConfig("AutoRelogIndex", 'WoWAccount1')
    end
    --本次开启自动登录,重登后失效,重登后检查 自动登录设置-自动重登设置
    function	BF.WriteConfig("BFAutoLogin",'1') 
    --本次掉线关闭自动登录,重登后失效,重登后检查 自动登录设置-自动重登设置
    function	BF.WriteConfig("BFAutoLogin",'0')
    --控制自动登录的时间，当前360秒后登录
    functionBF.WriteConfig(BFLoginTime,tostring(GetTime()+360))
    --切换游戏角色序号 index为角色序号
    function  BF.ChangWoWAccountIndex(index) end


​    
}
--======自身属性判断==============================================================================================================================
{
​    --以下各项值为判断自身各种属性，如 print(BF.Me.Class,BF.Me.ZY) 就可以获取自己的职业。
​    BF.Me={
​        --Hp比例
​        HP = 99.324668705403
​        --阵营
​        FactionGroup = Horde
​        --种族
​        Race = Tauren
​        --小区域地图
​        SubZone = 灌木谷
​        --职业
​        ZY = 德鲁伊
​        --是否战斗中
​        IsCombat = true
​        --Z坐标
​        PosZ = 9.0650234222412
​        --能量比例
​        PowerPct = 100
​        --背包空格
​        FreeBagSlots = 49
​        CombatStartTime = 305747.72
​        --最大生命值
​        HealthMax = 7848
​        PosX = -8943.197265625
​        ComboPoints = 0
​        --地图ID
​        MapId = 1446
​        --地图名称
​        Zone = 塔纳利斯
​        CombatDistance = 4
​        NoControl = false
​        InGroup = false
​        Casting = false
​        --面向角度
​        Angel = 3.8432590961456
​        DurabilityPercent = 98
​        Class = DRUID
​        SingleCombatTime = 4.170999999973
​        FrameRate = 59
​        --最大能量
​        PowerMax = 4955
​        Position = table: 0x2c9097010
​         {
​          Y = -2289.3046875
​          X = -8943.197265625
​          IsUnit = false
​          Z = 9.0650234222412
​         }
​        IPLocal = 省
​       
​        InCombat = true
​        GUID = Player-4707-044B94AF
​        Combat = 305747.72
​        Pointer = Player-4707-044B94AF
​        IsMoving = true
​        SwingMH = 0
​        Looting = false
​        CombatReach = 4.0500001907349
​        PosY = -2289.3046875
​        Power = 4955
​        Distance = 0
​        Health = 7795
​        PetActive = false
​        --运行脚本
​        ScriptName = /Z脚本源码/专业技能练习源码/1-300-部落-采矿.lua
​        CollectCount = 218
​        SwingOH = false
​        ComboMax = 5
​        PowerRegen = 22.687700271606
​        FBCount = 0
​        ComboDeficit = 5
​        EID = false
​        Moving = true
​        IsMelee = false
​        --Ip
​        IP = 123.1
​        CombatTime = 4.170999999973
​        --等级
​        Level = 70
​        DeadCount = 1
​        SingleCombat = 305747.72
​        NetStats = 29
​        NowOnLineTime = 01:27:22
​        InRaid = false
​        OnLineTime = 09:08:30
​        Instance = none
​        CombatLeft = false
​        BattleCount = 0
​        OpenClientTime = 15:13:42
​        
     }
}
--======GL主程序界面各项配置说明，==============================================================================================================================
{
    --设置这些值就可以控制主程序的功能
    --如勾选使用飞行坐骑
    --在野外脚本中使用
    { RunLua = 'BF.Settings.profile.UseFlyMount=true' },	
    --副本脚本中使用，
    BF.Settings.profile.UseFlyMount=true

    可以使用  BF.TableToString(BF.UI_Options.args) 来获取所有的设置面板。
    会在scripts/GL文件夹下生成 调试信息.txt 可以按面板提示查找所有增加的设置。
    
    如查找 玩家密语后自动回复，找到此项：BF.Settings.profile.WhisperSendBack 即为设置项。
    WhisperSendBack = table: 0x110a62f60
             type = toggle
             name = 玩家密语后自动回复
             set = function: 0x2ba199e30
             order = 3.2
             get = function: 0x125654740
             width = normal
             desc = 
    --前面为主界面设置值，括号为注释，最后为设置值，最下方有参考设置。
    白色（存储白色物品）值为：BF.Settings.profile.StorageWhite
    紫色（存储紫色物品）值为：BF.Settings.profile.StoragePurple
    绿色（）值为：BF.Settings.profile.ResolvedGreen
    保留类型,多种类型用|分割,留空为全部类型都分解（例如：其他|魔杖|法杖,留空为全部类型都分解,这个判断为物品大类,用背包物品获取.）值为： BF.Settings.profile.ResolvedKeepType 
    存储物品名称,|分隔（格式为：物品1|物品2|物品3）值为： BF.Settings.profile.StorageItemName
    蓝色（）值为：BF.Settings.profile.ResolvedBlue
    保留金币（保留多少金币,不存储请设置最大）值为： BF.Settings.profile.StorageKeepGold
    分解保留名称,一行一个,支持模糊匹配（格式为：保留关键字:物品小类1|物品小类2|物品小类3;例如：雄鹰:布甲|其它|法杖  代表保留关键字为雄鹰的法杖,布甲和其他类型(戒指项链)装备,同样支持精确匹配）值为： BF.Settings.profile.ResolvedKeepName 
    蓝色（存储蓝色物品）值为：BF.Settings.profile.StorageBlue
    绿色（存储绿色物品）值为：BF.Settings.profile.StorageGreen
    灰色（存储灰色物品）值为：BF.Settings.profile.StorageGray
    卖店价格低于(铜)（物品卖店价格低于多少铜全部分解，注意设定物品品质）值为： BF.Settings.profile.ResolvedItemSellPrice
    紫色（）值为：BF.Settings.profile.ResolvedPurple
    任务序号（该序号为最后执行的while指令,可以通过调整任务序号来调整要执行的任务值,请注意必须设定执行的任务脚本名称才能生效）值为： BF.Settings.profile.ExTaskScriptIndex
    执行的任务脚本名称（保存任务脚本名称）值为： BF.Settings.profile.ExTaskScript
    调试显示任务执行内容（用于任务调试，显示任务内容执行到那一步）值为： BF.Settings.profile.DisTaskValue
    使用指定条件换号（是否使用下方指定的条件,进行换号）值为： BF.Settings.profile.ChangeGameAccountOpen
    使用多段工作时间,不在工作时间段内会登录换号账号（是否使用多时间段定时上下线,如果勾选了此选项,会优先判断此工作时间,不再判断定时下线。）值为： BF.Settings.profile.MultiTimeLogOut
    使用以下条件下线,大于（采集次数大于以下数字）值为： BF.Settings.profile.TriggerOffline
    多工作时间段（例子:00:00to12:00|14:00to18:00|20:00to21:00|23:00to23:59）值为： BF.Settings.profile.MultiTimeWork
    切换账号条件（下方的函数条件符合就会换号: BF.GetComputerTime()==2202 or BF.GetComputerTime()==1402 填写例子代表在电脑时间14点02分和22点02分,符合换号条件。）值为： BF.Settings.profile.ChangeGameAccountCondition
    噩梦藤个数（）值为： BF.Settings.profile.NightmareVineCollectCount
    下线后多久上线(秒)（如果你7点下线，想8点上线，设置3600秒后上线,注意这个间隔要大于你想下线的时间秒数）值为： BF.Settings.profile.RelogAfterTime
    副本中不下线（副本中不下线）值为： BF.Settings.profile.FBNoLogOut
    指定游戏物体个数（）值为： BF.Settings.profile.CollectIDCount
    指定游戏物体ID（）值为： BF.Settings.profile.CollectID
    泰罗果个数（）值为： BF.Settings.profile.TeroconeCollectCount
    24小时采集次数（）值为： BF.Settings.profile.TotalCollectCount
    定时上线（是否开启定时上线,下线时会在登录页面等待到登录时间）值为： BF.Settings.profile.TimeLogIn
    智能登录（根据你设置的工作时间段,不在工作时间段后自动设定下线后多久上线）值为： BF.Settings.profile.SmartTimeLogIn
    使用累积在线时间下线（是否使用累积在线时间下线）值为： BF.Settings.profile.OLTimeReachLogOut
    工作时间段1（例子: 0:00to4:00,代表0点到4点在上号时间,如果只设置一个时间段,工作时间段1和工作时间段2设置为一样）值为： BF.Settings.profile.TimeWork1
    在线累积多少小时下线（今日累计在线多少小时下线）值为： BF.Settings.profile.OLTimeReachDrop
    工作时间段2（例子: 8:00to24:00,代表8点到24点在上号时间）值为： BF.Settings.profile.TimeWork2
    切换账号频率(秒)（多久切换账号一次）值为： BF.Settings.profile.TimeChangeGameAccountRate
    定时切换账号,本次在线时间达到就会登录换号账号,掉线重载时间重新计算（是否开启定时切换账号,在自动登录账号里设置要切换的账号信息，原理同副本换号）值为： BF.Settings.profile.TimeChangeGameAccount
    定时下线（是否开启定时下线）值为： BF.Settings.profile.TimeLogOut
    换号也等定时上线（使用换号后时同样等待定时上线设置）值为： BF.Settings.profile.ChangeGameAccountWaitTimeLogin
    换号前清空副本次数（换号前清空本号保存的BF副本次数）值为： BF.Settings.profile.LogOutClearFBCount
    指定次数炉石回城（是否刷本指定次数炉石回城）值为： BF.Settings.profile.FBCountUseHearthStone
    死亡重置副本（）值为： BF.Settings.profile.DeadResetInstances
    爆本退出魔兽（有时候内置统计会出错，避免出错退出魔兽）值为： BF.Settings.profile.BoomLogOut
    刷本指定次数（刷本多少次退出魔兽或者换号）值为： BF.Settings.profile.FBCount
    指定次数退出魔兽（是否刷本指定次数后直接退出魔兽）值为： BF.Settings.profile.FB30CountStop
    指定次数或者爆本换号,无换号信息在登录界面等待（换号功能需要在自动登录里设置好要换号的信息）值为： BF.Settings.profile.FB30CountChangeAccount
    爆本停止脚本（有时候内置统计会出错，避免出错停止脚本）值为： BF.Settings.profile.BoomStop
    强制邮寄物品列表（每行一个）值为： BF.Settings.profile.ForceMailList
    邮寄起始金币（金币大于这个值开始邮寄金币）值为： BF.Settings.profile.MailStartMoney
    开启邮寄（）值为： BF.Settings.profile.UseMail
    不邮寄物品列表（每行一个）值为： BF.Settings.profile.DoNotMailList
    邮寄紫色（邮寄紫色）值为： BF.Settings.profile.MailPurple
    邮寄灰色（邮寄灰色）值为： BF.Settings.profile.MailGray
    上线重新添加邮寄好友（如果出现收件人是好友,也出现邮寄确认框的情况,请勾选此项,会每次上线重新添加好友）值为： BF.Settings.profile.ReAddMailFriend 
    发送邮件时采用付费邮件（邮件需要付费随机1金到10金才能收取,邮件时效3天）值为： BF.Settings.profile.UseGoldMail
    邮寄蓝色（邮寄蓝色）值为： BF.Settings.profile.MailBlue
    主题（主题）值为： BF.Settings.profile.MailSubject
    邮寄绿色（邮寄绿色）值为： BF.Settings.profile.MailGreen
    收件人（收件人）值为： BF.Settings.profile.MailRecipient
    邮寄白色（邮寄白色）值为： BF.Settings.profile.MailWhite
    保留金币（邮寄后保留的金币）值为： BF.Settings.profile.MailRemoney
    使用治疗药水%（使用治疗药水最低血量百分比）值为： BF.Settings.profile.HPPercent / 100
    使用食物（使用食物）值为： BF.Settings.profile.UseFood
    使用饮料（使用饮料）值为： BF.Settings.profile.UseDrink
    到%（Drink Percent）值为： BF.Settings.profile.DrinkMaxPercent / 100
    恢复生命值从%（Food Percent）值为： BF.Settings.profile.FoodPercent / 100
    使用药剂（是否根据自身buff使用药剂）值为：BF.Settings.profile.UsePotions
    食物名称（食物名称）值为： BF.Settings.profile.FoodName
    自身使用药剂（填写名称,多个药剂用,分隔，要和前面的buff名称对应,分隔符为英文逗号,最多4个）值为：BF.Settings.profile.MeUsePotions
    自身无BUFF（判断自身有无buff名称,多个buff用,分隔，要和后面的药剂对应,分隔符为英文逗号,最多4个）值为：BF.Settings.profile.MeNoBuff
    武器附魔（是否进行武器附魔）值为：BF.Settings.profile.WeaponEnchant
    武器附魔物品名称（进行物品附魔的物品名称）值为：BF.Settings.profile.WeaponEnchantName
    使用饰品装备（）值为： BF.Settings.profile.UseTrinket
    使用法力药水%（使用法力药水最低蓝量百分比）值为： BF.Settings.profile.MPPercent / 100
    到%（Food Percent）值为： BF.Settings.profile.FoodMaxPercent / 100
    使用治疗药水（）值为： BF.Settings.profile.UseHP
    使用法力药水（）值为： BF.Settings.profile.UseMP
    恢复法力值从%（Drink Percent）值为： BF.Settings.profile.DrinkPercent / 100
    饮料名称（饮料名称）值为： BF.Settings.profile.DrinkName
    指定密聊好友名称（是否指定随机密聊的好友名称）值为： BF.Settings.profile.WhisperFriendName
    随机密聊挑选（随机挑选下面的一句话密聊好友）值为： BF.Settings.profile.WhisperFriendRandomMessage
    开启随机密聊好友（是否开启随机密聊在线好友,频率约为1小时2次）值为： BF.Settings.profile.WhisperFriendToggle
    水中不使用导航（水中不使用导航）值为： BF.Settings.profile.NoNavInWater
    使用飞行导航（是否使用内置的飞行导航路径）值为： BF.Settings.profile.UseFlyNavigation
    卡位多次炉石回城（）值为： BF.Settings.profile.StuckUseHearthStone
    随机跳跃几率（寻路导航途中随机跳跃的几率，默认0.2%）值为：BF.Settings.profile.NavRandomJumpChance / 100
    内置导航（使用解锁器陆地导航）值为： BF.Settings.profile.UseEWTNavigation
    处理水中卡位（当在水中卡位时也进行卡位处理）值为： BF.Settings.profile.NavigationCoreectInWater
    调试导航信息用（是否显示具体的导航信息,调试导航用）值为： BF.Settings.profile.NavBugShow
    导航坐标随机偏差（导航中路径随机偏差,终点偏移：终点随机偏移，每点偏移:寻路中每点随机偏移，可能造成卡位,不要在需要精确寻路的脚本中使用）值为： BF.Settings.profile.NavRandomPoint
    |cFFFF0000不使用点击移动（|cFFFF0000慎用，会造成很多脚本移动出问题，移动时不使用鼠标点击,使用转向移动。仅陆地移动生效）值为： BF.Settings.profile.NoUseCTM
    无路径时随机移动（寻路导航没有返回路点时,随机移动到附近一点再寻路）值为： BF.Settings.profile.NoWayToNext
    导航中随机跳（寻路导航中随机跳一跳,更加拟人化,自己设置跳跃几率,副本中或者容易卡位的路径不要开启,容易造成卡位.）值为： BF.Settings.profile.NavRandomJump
    直接执行中控指令（直接执行中控台发出的指令,请确保你的指令正确）值为： BF.Settings.profile.UseConsoleDirect
    使用中控台（是否开启GL中控制台,请网盘下载控制台程序）值为： BF.Settings.profile.UseConsole
    通讯频率(秒)（多久和控制台通讯一次,默认1分钟）值为： BF.Settings.profile.ConsoleSeconds
    控制台指令执行（）值为： BF.Settings.profile.ConsoleScriptCode
    端口（控制台端口,默认5001,公网IP请放通端口）值为： BF.Settings.profile.ConsolePort
    IP（填写局域网IP或者公网IP）值为： BF.Settings.profile.ConsoleIP
    （反馈的为GL还是RT的问题,请选择）值为： BF.Settings.profile.BugVer
    回复内容（当作者看到您到问题后，会后台回复你到问题，请耐心等待）值为： BF.Settings.profile.BugAnswer
    反馈BUG和建议（）值为： BF.Settings.profile.BugAndAdvice
    邮箱位置,邮箱附近点击下方按钮,可不填（到邮箱附近,点击生成邮箱位置）值为： BF.Settings.profile.CSMailBox
    修理NPC,选中修理NPC,点击下方按钮（生成修理,选中修理NPC,点击下方按钮）值为： BF.Settings.profile.CSRepair
    采集打怪路点,每到一个采集打怪点,点击下方按钮（鼠标激活此框,到一个采集打怪点,点击生成打怪采集路点）值为： BF.Settings.profile.CSHotspots
    食物NPC,选中食物NPC,点击下方按钮（生成购买食物位置,选中食物NPC,点击下方按钮）值为： BF.Settings.profile.CSFood
    黑名单坐标点,到一个黑名单点,点击下方按钮,可不填（鼠标激活此框,到一个采集打怪点,点击生成黑名单点,此点20尺范围内怪物不打）值为： BF.Settings.profile.CSBlackspots
    手动输入你生成的脚本名称（输入脚本名称才会生成脚本,生成后运行此脚本即可。）值为： BF.Settings.profile.CSName
    弹药NPC,选中弹药NPC,点击下方按钮（生成弹药,选中弹药NPC,点击下方按钮）值为： BF.Settings.profile.CSAmmo
    选中怪物,点击下方按钮（选中怪物,点击下方按钮）值为： BF.Settings.profile.CSMobIds
    修理路线,可不填（如果修理回城时卡位较多，可以自己录制修理路线,包括你自己回城时的修理补给点）值为： BF.Settings.profile.CSVendorPath
    采集名称,会智能判断,不必填（勾选拾取采集里的采集选项,就会自动判断采集物品,不用填）值为： BF.Settings.profile.CSGatherIds
    跑尸路线,可不填（如果默认寻路跑尸路线到达不了,请自己录制跑尸路线）值为： BF.Settings.profile.CSCorpspots
    德鲁伊使用旅行形态（德鲁伊使用旅行形态代替使用坐骑）值为： BF.Settings.profile.DRUIDNavUseTravelForm
    德鲁伊使用飞行形态（德鲁伊使用飞行形态代替使用飞行坐骑,需要勾选使用飞行坐骑）值为： BF.Settings.profile.DRUIDNavUseFlyForm
    德鲁伊飞行形态打怪（德鲁伊飞行状态下也进行攻击）值为： BF.Settings.profile.DRUIDFlyFormAttack
    坐骑名称（不填写则自动判断背包坐骑）值为： BF.Settings.profile.MountName
    使用坐骑（使用坐骑）值为： BF.Settings.profile.UseMount
    检测上坐骑间隔(秒)（脱战后多少秒以内上坐骑,寻路时间隔多久检测一次上坐骑）值为： BF.Settings.profile.MountCheckTime
    使用飞行坐骑Z坐标提升（使用飞行坐骑Z坐标提升）值为： BF.Settings.profile.UseFlyMountUpZ
    使用飞行坐骑距离（距离目标点多少距离,使用飞行坐骑）值为： BF.Settings.profile.UseFlyMountDistance
    坐骑距离（坐骑距离）值为： BF.Settings.profile.MountDistance
    使用飞行坐骑（使用飞行坐骑）值为： BF.Settings.profile.UseFlyMount
    坐骑时不反击（坐骑时不反击）值为： BF.Settings.profile.NotAttackWhenMounted
    自动学天赋（是否自动学天赋,现在只设置了法师猎人学天赋,其他职业不要勾选,在插件目录/自定义设置.lua里设置。）值为： BF.Settings.profile.AutoLearnTalent
    自动接受组队（自动接受组队）值为： BF.Settings.profile.AutoAcceptGroup
    打怪采集时从离自己最近的点开始（打怪采集时从离自己最近的点开始,而不是从第一点开始）值为： BF.Settings.profile.GetGrindNear
    拒绝组队（自动拒绝组队）值为： BF.Settings.profile.AutoDeclineGroup
    Debug（bug信息）值为： BF.Settings.profile.Debug
    聊天框文字输出级别（数字越大,聊天框输出的信息越少,越小,输出的信息越多,默认5,设置5到15即可,每加1就会少一项输出,15以上的级别还没有采用,到100,BF的信息输出会没有,但是调用魔兽print()的输出信息还会存在）值为： BF.Settings.profile.MsgLv
    复活超时(秒)（多长时间寻找不到尸体,虚弱复活）值为： BF.Settings.profile.GotoCorpseLimitTime
    尸体中心点复活（野外脚本复活时在尸体中心点复活）值为： BF.Settings.profile.RepopMeInCenter
    其他职业攻击距离（当角色距离怪物多少码以内开始释放技能，近战职业在此设置）值为： BF.Settings.profile.ClassCombatDistance
    水中等待自然回复（水中是否等待自动回复,取消则水中不进行回复）值为： BF.Settings.profile.NoRestInWater
    自动换更好的背包（是否打到大背包换小背包，如果造成问题,请取消。）值为： BF.Settings.profile.AutoEquipBetterBag
    更改路点循环方式（倒序:野外路径打怪到最后一点后倒序走回来，原本次序1-2-3-1-2-3,勾选后变为1-2-3-2-1-2-3,随机:有1,2,3点,每次随机选一点打怪）值为： BF.Settings.profile.ReverseGrinding
    装备耐久0虚弱复活（有装备耐久为0时会灵魂医者复活,会有虚弱复活DEBUFF）值为： BF.Settings.profile.NeedRepairRetrieve
    虚弱复活等待（等待虚弱复活DEBUFF消失）值为： BF.Settings.profile.WeakWaiting
    RT-Server（RT最新脚本在线路1，备用脚本在线路2，New RT Script is in line 1,Reserver Script is in 2）值为： BF.Settings.profile.RTline
    快速寻怪（用于快速寻找怪物，只判断怪物ID和是否死亡）值为： BF.Settings.profile.QuikA
    统计角色数据（）值为： BF.Settings.profile.GetDetailedroledata
    副本中战斗不拾取（副本中战斗进入战斗时不拾取和不剥皮）值为： BF.Settings.profile.InCombatNoLoot
    主动移动击杀玩家（玩家出现在设定攻击范围内，主动移动到玩家去击杀）值为： BF.Settings.profile.ActivAttackPlayer
    法师开始攻击距离（当远程职业距离怪物多少码以内开始释放技能）值为： BF.Settings.profile.MageCombatDistance
    攻击玩家（是否反击玩家或主动攻击玩家）值为： BF.Settings.profile.AttackPlayer
    自动防暂离（自动防暂离功能）值为： BF.Settings.profile.AutoResetAfk
    拒绝决斗（自动拒绝决斗）值为： BF.Settings.profile.AutoCancelDuel
    安全复活（当野外复活周围有怪物时,会面向怪物后退复活）值为： BF.Settings.profile.SafeRetrieve
    始终使用修理路线（野外打怪时如果有修理路线,脚本开始时一直使用）值为： BF.Settings.profile.AlwaysUseVendorPath
    使用加密的技能循环（使用用户分享的技能循环，暂时只有治疗牧师）值为： BF.Settings.profile.SecretRotation
    自动换装备（是否自动穿更好的装备,在45级以前售卖时会进行自动换装）值为： BF.Settings.profile.AutoEquip
    死亡自动释放灵魂（需要自动释放灵魂不要取消,取消则不释放灵魂,）值为： BF.Settings.profile.AutoRepopMe
    开启销毁/使用(拾取物品后进行销毁/使用)（会在拾取后进行销毁/使用）值为：BF.Settings.profile.DestroyWhenLoot
    使用物品,用|分割,填物品名称或者ID都可以（例如：狼肉|12356|石头,用编写调试-背包物品获取物品ID）值为： BF.Settings.profile.UseItemList  
    销毁低于此价格的物品(铜)（价格标准为铜）值为： BF.Settings.profile.DesroyItemLessPrice
    模糊销毁物品,用|分割,填物品名称关键字即可（例如：矿石|草,就会销毁所有名称中带矿和草的物品）值为： BF.Settings.profile.VagueDestroyItemList  
    合成微粒（合成土之,火焰等微粒）值为：BF.Settings.profile.Composparti
    根据价格进行销毁（开启根据价格进行销毁）值为：BF.Settings.profile.DesroyAccordingPrice
    销毁物品,用|分割,填物品名称或者ID都可以（例如：狼肉|12356|石头,用编写调试-背包物品获取物品ID）值为： BF.Settings.profile.DestroyItemList  
    灰色（）值为：BF.Settings.profile.DestroyGray
    自动剥皮（自动剥皮）值为： BF.Settings.profile.AutoSkinning
    采集草药（采集草药）值为： BF.Settings.profile.AutoGatherHerb
    只剥自己（只剥皮自己杀的怪）值为： BF.Settings.profile.JustSkinMyself
    水中采集（水中采集）值为： BF.Settings.profile.CollectItemsInWater
    不采集灰色物品（采矿和采药等级和采集物品相差多少级不采集,默认75）值为： BF.Settings.profile.NoCollectGray
    采集箱子（采集野外的箱子,蘑菇等可以采集的物品）值为： BF.Settings.profile.AutoGatherBox
    遇怪不反击（采集途中遇怪不打,即使不在坐骑上,请确保你的采集路线不会被轻易打死）值为： BF.Settings.profile.CollectNoAttack
    怪物超过多少个（判断拾取尸体周围多少范围内的怪物数量超过多少个,就暂时不拾取这个尸体）值为： BF.Settings.profile.LootPassMonsterNum
    自动拾取（自动拾取）值为： BF.Settings.profile.AutoLoot
    采集黑点坐标（表格形式，存在X,Y,Z即可,Range代表拉黑范围.例子:{ Name = '精金矿脉', Entry = 181556, X = 2500.0637, Y = 3366.5195, Z = 123.9153, Range = 50 },）值为： BF.Settings.profile.CollectBlackPoint
    自动清除采集黑名单（每隔5分钟自动清除一次自动添加采集黑名单,并不是上方的黑名单，是移动超时添加的黑名单）值为： BF.Settings.profile.AutoClearCollectBlackList
    采集矿物（采集矿物）值为： BF.Settings.profile.AutoGatherOre
    拾取过滤（开启拾取尸体过滤,当尸体范围内怪物超过以下标准,暂缓拾取）值为： BF.Settings.profile.LootPassVielMonster
    任意采集物品每日采集次数大于（任意一个采集物品的每日采集次数大于下面的值,就不再采集）值为： BF.Settings.profile.BlackAnyCollectNameCount
    采集黑名单表,填名称用|分隔（例如：铜矿|锡矿|银叶草）值为： BF.Settings.profile.CollectBlackList
    过滤精英怪（采集时过滤等级差8级内的精英,如70级采集物周围有63级的精英就不再采集,过滤范围采用打怪-判断怪物|矿草周围多少范围内码数。）值为： BF.Settings.profile.PassElite 
    指定游戏物品个数3（）值为： BF.Settings.profile.BlackCollectNameCount3
    指定游戏物品名称2（）值为： BF.Settings.profile.BlackCollectName2
    按背包物品数量添加采集黑名单（按背包物品数量添加采集黑名单）值为： BF.Settings.profile.TriggerBlackList
    直线移动到采集点距离（如果采集时,出现来回移动的情况,把这个范围尽量调小,这个控制直接移动到采集拾取点的距离）值为： BF.Settings.profile.HMInWaterSearchRadius
    限制双采等级差（采集等级和物品需求等级差大于此值不采集,开启不采集灰色物品生效）值为： BF.Settings.profile.CollectLevAbs
    指定游戏物品名称3（）值为： BF.Settings.profile.BlackCollectName3
    不和玩家抢采集物品（超怂，采集物周围有玩家就放弃，即使是自己先发现的物品）值为： BF.Settings.profile.Dodgeplayer 
    指定游戏物品个数1（）值为： BF.Settings.profile.BlackCollectNameCount1
    采集/拾取超时(秒)（采集/拾取超过时间,放弃采集/拾取,添加黑名单）值为： BF.Settings.profile.CollectLimitTime
    判断尸体周围多少范围内(码)（判断尸体周围多少范围内的怪物数量超过多少个,就暂缓拾取）值为： BF.Settings.profile.LootPassMonsterRange
    指定游戏物品个数2（）值为： BF.Settings.profile.BlackCollectNameCount2
    采集物周围有玩家不采集（当采集时,采集物品周围有玩家不采集这个物品）值为： BF.Settings.profile.PassPlayer 
    指定游戏物品名称1（）值为： BF.Settings.profile.BlackCollectName1
    吸取微粒（需要工程305,有气阀微粒提取器）值为： BF.Settings.profile.AutoGatherParti
    采集范围(码)（搜索多少码内的矿草,解锁器一般只能识别到120码范围的采集物品）值为： BF.Settings.profile.HMSearchRadius
    采集尸体（采集一些可以采集的非剥皮类尸体,请注意采药或者采矿技能需要符合）值为： BF.Settings.profile.AutoGatherElecorpse
    紫色（）值为： BF.Settings.profile.SellPurple
    蓝色（）值为： BF.Settings.profile.SellBlue
    不检测回城（不检测回城）值为： BF.Settings.profile.NotGoToTown
    食物数量（回城购买多少食物）值为： BF.Settings.profile.FoodAmount
    弹药数量（当弹药数量少于多少时触发回城）值为： BF.Settings.profile.AmmoAmountToGoToTown
    包满回城（是否触发包满回城）值为： BF.Settings.profile.FullBagToGoToTown
    炉石后暂停N秒（）值为： BF.Settings.profile.StopSecondAfterUseHearthStone
    强制出售物品列表（强制出售物品列表）值为： BF.Settings.profile.ForceSellList
    统计所有背包空格（勾选此项后将计算所有的背包空格,包括草药袋，箭袋，矿石袋）值为： BF.Settings.profile.GetAllFreeBagSlots
    不出售物品列表(一行一个),可以写物品ID（一行一个，可以写关键字比如 石头）值为： BF.Settings.profile.DoNotSellList
    绿色（）值为： BF.Settings.profile.SellGreen
    按需购买弹药数量（猎人不再购买指定数量的弹药,按背包空格数量*设定数量购买弹药，勾选后指定弹药数量自动设置）值为： BF.Settings.profile.AmmoAmountBuyPercentToGoToTown
    白色（）值为： BF.Settings.profile.SellWhite
    必须使用炉石回城（炉石不CD不回城）值为： BF.Settings.profile.MustUseHearthStone
    灰色（）值为： BF.Settings.profile.SellGray
    回城不反击（回城不反击）值为： BF.Settings.profile.NotAttackWhenGoToTown
    指定弹药数量（回城购买多少弹药,勾选按需购买弹药数量后,此项设置无效）值为： BF.Settings.profile.AmmoAmount
    背包空格*设定数量购买（需要勾选按需购买弹药数量,回城购买多少弹药）值为： BF.Settings.profile.BagxAmmoAmount
    金币数量（当金币数量大于多少时触发回城）值为： BF.Settings.profile.MoeneyToGoToTown
    饮料数量（回城购买多少饮料）值为： BF.Settings.profile.DrinkAmount
    使用炉石（是否使用炉石回城）值为： BF.Settings.profile.UseHearthStone
    装备耐久（当装备耐久度小于等于%多少时触发回城）值为： BF.Settings.profile.MinDurabilityPercent
    背包空格（当背包空格小于等于多少时触发回城）值为： BF.Settings.profile.MinFreeBagSlotsToGoToTown
    战场预估排队最大时间（战场预估排队时间超过此时间,增加战场次数1）值为： BF.Settings.profile.MaxBattlefieldEstimatedWaitTime
    战场排队最大时间（战场排队时间超过此时间,增加战场次数1）值为： BF.Settings.profile.MaxBattlefieldTimeWaited
    竞技场排队（请选择排那一个竞技场）值为： BF.Settings.profile.ArenaQueue
    刷战场次数（指定刷战场多少次）值为： BF.Settings.profile.MaxBattleCount
    切换到脚本4条件（请打开脚本:GL-Tinkr/脚本编写说明/脚本切换例子初始脚本.lua,参考各项指令的说明）值为： BF.Settings.profile.ChangeScript4Condition
    切换到脚本2条件（请打开脚本:GL-Tinkr/脚本编写说明/脚本切换例子初始脚本.lua,参考各项指令的说明）值为： BF.Settings.profile.ChangeScript2Condition
    切换到脚本3条件（请打开脚本:GL-Tinkr/脚本编写说明/脚本切换例子初始脚本.lua,参考各项指令的说明）值为： BF.Settings.profile.ChangeScript3Condition
    切换脚本代码（在脚本编写说明/通过脚本代码切换设置.lua 查看编写说明）值为： BF.Settings.profile.ChangeScriptCode
    切换到脚本1条件（请打开脚本:GL-Tinkr/脚本编写说明/脚本切换例子初始脚本.lua,参考各项指令的说明）值为： BF.Settings.profile.ChangeScript1Condition
    脚本1名称（填写脚本路径,如:GL-Tinkr/脚本编写说明/脚本切换例子初始脚本.lua）值为： BF.Settings.profile.Script1Name
    脚本4名称（填写脚本路径,如:GL-Tinkr/脚本编写说明/脚本切换例子初始脚本.lua）值为： BF.Settings.profile.Script4Name
    脚本3名称（填写脚本路径,如:GL-Tinkr/脚本编写说明/脚本切换例子初始脚本.lua）值为： BF.Settings.profile.Script3Name
    回城切换脚本名称（切换该脚本有1个小时CD）值为： BF.Settings.profile.GoToTownChangeScriptName
    脚本2名称（填写脚本路径,如:GL-Tinkr/脚本编写说明/脚本切换例子初始脚本.lua）值为： BF.Settings.profile.Script2Name
    开启回城切换脚本（是否开启回城切换指定脚本,内置1个小时CD）值为： BF.Settings.profile.GoToTownChangeScriptToggle
    开启按条件切换脚本（是否开启按指定条件切换脚本）值为： BF.Settings.profile.ChangeScriptToggle
    开启包满切换脚本（是否开启包满切换指定脚本,内置1个小时CD）值为： BF.Settings.profile.FullBagChangeScript
    登录运行延时(秒)（登录后延时几秒运行脚本）值为： BF.Settings.profile.AutoRunSecond
    脚本停止自动重启（脚本报错自动重启）值为： BF.Settings.profile.Restart
    登录自动运行脚本（进入游戏自动运行脚本）值为： BF.Settings.profile.AutoRun
    换号账号信息（格式: 账号|密码|子账号|服务器|角序|安全令序列号|还原码,不想自动登录,设置为空即可。）值为： BF.Settings.profile.WoWAccount2
    登录账号序号（把指定账号序号的信息保存在解锁器文件夹/scripts/GL/自动登录账号设置.ini）值为： BF.Settings.profile.AutoRelogIndex
    重登账号信息（格式: 账号|密码|子账号|服务器|角序|安全令序列号|还原码,不想自动登录,设置为空即可。）值为： BF.Settings.profile.WoWAccount1
    自动重登（）值为： BF.Settings.profile.AutoRelog
    不在坐骑上距离(短距离)（国服会传送2码以内的距离，建议设置为2，同时可以调整判断传送间隔）值为： BF.Settings.profile.ShortUnMountTeleportDistance
    传送后炉石回城（被GM传送后是否炉石回城）值为： BF.Settings.profile.AfterTeleportGoToTown
    被GM传送后想执行的脚本路径（被GM传送后你想执行的脚本路径,例如：GL-Tinkr/被GM传送后执行脚本/TeleAction.lua）值为： BF.Settings.profile.TeleportCode
    GM传送警报（）值为： BF.Settings.profile.TeleportAlert
    被传送后执行指定脚本（是否开启被GM传送后执行指定脚本）值为： BF.Settings.profile.AfterTeleportExScript
    坐骑上距离（在坐骑上瞬间移动距离大于此距离报警）值为： BF.Settings.profile.TeleportDistance
    不在坐骑上距离（国服会传送2码以内的距离，建议设置为2，同时可以调整判断传送间隔）值为： BF.Settings.profile.UnMountTeleportDistance
    异常转向报警（角色异常面向报警）值为： BF.Settings.profile.AbnormalRotationAlert
    Z坐标提升报警（当X,Y坐标微变,Z坐标提升时进行报警。）值为： BF.Settings.profile.ZUpAlert
    玩家密语后自动回复（）值为： BF.Settings.profile.WhisperSendBack
    判断传送间隔(毫秒)（）值为： BF.Settings.profile.JudgmentInterval
    GM密语报警（）值为： BF.Settings.profile.WhisperAlert
    传送报警始终执行（脚本停止时也进行传送检测）值为： BF.Settings.profile.ImmerAlert
    配置密语回复规则（填写配置密语回复规则路径,例子：GL-Tinkr/被GM传送后执行脚本/密语回复配置.txt,类似于QQ机器人的智能回复,不能触发会随机挑选一句玩家密聊回复）值为： BF.Settings.profile.SmartWhisperPath
    异常转向值（异常转向值越小越敏感,越大越不敏感）值为： BF.Settings.profile.RDiffSum
    灵魂状态传送报警（）值为： BF.Settings.profile.TeleGhostAlert


    --以下为各项值如何参考设置
        BF.Settings.profile.AutoLearnTalent=true
        BF.Settings.profile.RTline=1
        BF.Settings.profile.TeleportCode='GL-Tinkr/被GM传送后执行脚本/TeleAction.lua'
        BF.Settings.profile.AlwaysUseVendorPath=false
        BF.Settings.profile.WoWAccount1=''
        BF.Settings.profile.Restart=false
        BF.Settings.profile.HunterCollectNoAttack=true
        BF.Settings.profile.FlyTeleportAlert=false
        BF.Settings.profile.JudgmentInterval=50
        BF.Settings.profile.AlwaysRevivePet=true
        BF.Settings.profile.HMInWaterSearchRadius=5
        BF.Settings.profile.AutoLoot=false
        BF.Settings.profile.UseHP=true
        BF.Settings.profile.UseTrinket=true
        BF.Settings.profile.LuaErr=true
        BF.Settings.profile.MinDurabilityPercent=19
        BF.Settings.profile.ChangeGameAccountWaitTimeLogin=false
        BF.Settings.profile.LootPassVielMonster=false
        BF.Settings.profile.UseMP=true
        BF.Settings.profile.DRUIDNavUseFlyForm=false
        BF.Settings.profile.MultiTimeWork='00:00to4:00|6:00to12:00|14:00to18:00|20:00to21:00|23:00to23:59'
        BF.Settings.profile.UseFlyMountDistance=10
        BF.Settings.profile.Composparti=true
        BF.Settings.profile.SecretRotation=false
        BF.Settings.profile.CSFood='{ Name = '泰罗巨蛾', Entry = 18468, X = -2057.4419, Y = 4498.6035, Z = 9.4202, Distance = 35.619173225855, Radius = 10 }'
        BF.Settings.profile.ShortTeleportDistance=15
        BF.Settings.profile.FrameRate=30
        BF.Settings.profile.CollectLevAbs=85
        BF.Settings.profile.WeaponEnchant=false
        BF.Settings.profile.CSVendorPath=''
        BF.Settings.profile.BlackCollectNameCount2=400
        BF.Settings.profile.TeleComputer='编写2'
        BF.Settings.profile.BFBotNum=1
        BF.Settings.profile.TimeWork1='0:00to3:00'
        BF.Settings.profile.TotalCollectCount=800
        BF.Settings.profile.NoNavInWater=true
        BF.Settings.profile.MultiTimeLogOut=false
        BF.Settings.profile.TriggerOffline=true
        BF.Settings.profile.GetGrindNear=true
        BF.Settings.profile.AutoGatherHerb=false
        BF.Settings.profile.LootPassMonsterRange=25
        BF.Settings.profile.FoodPercent=80
        BF.Settings.profile.BugAndAdvice=''
        BF.Settings.profile.WhisperReplyMessage='(*￣︶￣)'
        BF.Settings.profile.AfterTeleportExScript=false
        BF.Settings.profile.AutoRepopMe=true
        BF.Settings.profile.CollectNoAttack=false
        BF.Settings.profile.CollectFeigndeath=true
        BF.Settings.profile.LogoutTargetByPlayer=true
        BF.Settings.profile.NoWayToNext=false
        BF.Settings.profile.BFAccountPassword=''
        BF.Settings.profile.NavBugShow=false
        BF.Settings.profile.TeroconeCollectCount=400
        BF.Settings.profile.MainWindw_Top=623
        BF.Settings.profile.ChangeScriptToggle=false
        BF.Settings.profile.AutoEquipBetterBag=true
        BF.Settings.profile.ChangeScript2Condition=''
        BF.Settings.profile.BFKardKey=''
        BF.Settings.profile.BFLoginAutoUpdate=false
        BF.Settings.profile.AttackLimitTime=1000
        BF.Settings.profile.DoNotSellQuality='灰色'
        BF.Settings.profile.WoWAccount2=''
        BF.Settings.profile.FB30CountChangeAccount=false
        BF.Settings.profile.MailPurple=true
        BF.Settings.profile.CloseAlertMultiTime='00:00to8:00'
        BF.Settings.profile.Debug=true
        BF.Settings.profile.ShortTeleStopSecond=200
        BF.Settings.profile.InCombatNoLoot=true
        BF.Settings.profile.CollectUseIce=true
        BF.Settings.profile.NavRandomJump=false
        BF.Settings.profile.AutoResetAfk=true
        BF.Settings.profile.UseHearthStone=false
        BF.Settings.profile.ForceMailList=''
        BF.Settings.profile.LogOutClearFBCount=false
        BF.Settings.profile.NotGoToTown=false
        BF.Settings.profile.AfterTeleportGoToTown=false
        BF.Settings.profile.BagxAmmoAmount=30
        BF.Settings.profile.SellGreen=false
        BF.Settings.profile.NightmareVineCollectCount=400
        BF.Settings.profile.FBCount=120
        BF.Settings.profile.GetAllFreeBagSlots=false
        BF.Settings.profile.AutoGatherElecorpse=false
        BF.Settings.profile.ConsoleIP='192.168.0.104'
        BF.Settings.profile.MustUseHearthStone=false
        BF.Settings.profile.ChangeScript1Condition=''
        BF.Settings.profile.WhisperAlert=true
        BF.Settings.profile.BlackCollectNameCount1=400
        BF.Settings.profile.WeaponEnchantName=''
        BF.Settings.profile.CloseAlertToggle=false
        BF.Settings.profile.SmartWhisperPath='GL-Tinkr/被GM传送后执行脚本/密语回复配置.txt'
        BF.Settings.profile.GotoCorpseLimitTime=200
        BF.Settings.profile.MaxBattlefieldTimeWaited=3600
        BF.Settings.profile.AttackGreyMob=true
        BF.Settings.profile.UseFlyNavigation=true
        BF.Settings.profile.ReverseGrinding=1
        BF.Settings.profile.AutoUpdateRate=24
        BF.Settings.profile.NeedRepairRetrieve=true
        BF.Settings.profile.StopSecondAfterUseHearthStone=0
        BF.Settings.profile.TeleGhostAlert=false
        BF.Settings.profile.UseTrinketNoCombat=false
        BF.Settings.profile.MaxBattlefieldEstimatedWaitTime=7200
        BF.Settings.profile.TimeChangeGameAccount=false
        BF.Settings.profile.CSBlackspots=''
        BF.Settings.profile.StorageKeepGold=1000000
        BF.Settings.profile.UseBFRotation=true
        BF.Settings.profile.MeUsePotions=''
        BF.Settings.profile.TimeLogIn=true
        BF.Settings.profile.SmartTimeLogIn=true
        BF.Settings.profile.NoAttackPlayer=false
        BF.Settings.profile.AfterSecondUseHearthStone=8
        BF.Settings.profile.ChangeGameAccountOpen=false
        BF.Settings.profile.AutoAcceptGroup=false
        BF.Settings.profile.HunterScan=false
        BF.Settings.profile.AttackOthersMonster=false
        BF.Settings.profile.NoCollectGray=true
        BF.Settings.profile.DRUIDFlyFormAttack=false
        BF.Settings.profile.HMSearchRadius=120
        BF.Settings.profile.ResolvedKeepName=''
        BF.Settings.profile.ForceSellList=''
        BF.Settings.profile.ZUpAlert=false
        BF.Settings.profile.LootPassMonsterNum=1
        BF.Settings.profile.RepopMeInCenter=false
        BF.Settings.profile.RandomMoveCount=5
        BF.Settings.profile.RepopCloseBT=false
        BF.Settings.profile.FoodMaxPercent=100
        BF.Settings.profile.ConsolePort='5001'
        BF.Settings.profile.FoodAmount=0
        BF.Settings.profile.StuckUseHearthStone=false
        BF.Settings.profile.BFAccountName='ccoude19'
        BF.Settings.profile.BFGameAccountBind=''
        BF.Settings.profile.BFAutoUpdateTime=0
        BF.Settings.profile.PassElite=false
        BF.Settings.profile.DrinkAmount=0
        BF.Settings.profile.CSAmmo='{ Name = '泰罗巨蛾', Entry = 18468, X = -2057.4419, Y = 4498.6035, Z = 9.4202, Distance = 35.619173225855, Radius = 10 }'
        BF.Settings.profile.UseEWTNavigation=true
        BF.Settings.profile.SellWhite=false
        BF.Settings.profile.NavRandomPoint=1
        BF.Settings.profile.PassMonsterRange=25
        BF.Settings.profile.BlackCollectName3=''
        BF.Settings.profile.ChangeScript3Condition=''
        BF.Settings.profile.FullBagToGoToTown=true
        BF.Settings.profile.UseMount=true
        BF.Settings.profile.CSHotspots='
        { X=-2075.883789, Y=4468.335938, Z=5.885282,},
        { X=-2137.006836, Y=4433.165527, Z=-1.075469,},
        { X=-2181.984863, Y=4474.823242, Z=2.713873,},'
        BF.Settings.profile.GoToTownChangeScriptToggle=false
        BF.Settings.profile.OutGMStation=true
        BF.Settings.profile.StorageBlue=false
        BF.Settings.profile.Dodgeplayer=false
        BF.Settings.profile.ShortTeleStop=false
        BF.Settings.profile.DoNotMailList=''
        BF.Settings.profile.WoWAccount3=''
        BF.Settings.profile.ResolvedItemSellPrice=1000000
        BF.Settings.profile.ZUpDis=6
        BF.Settings.profile.UseItemList=''
        BF.Settings.profile.WhisperFriendRandomMessage=':-D'
        BF.Settings.profile.AutoUpdateDelay=30
        BF.Settings.profile.StoragePurple=false
        BF.Settings.profile.ConsoleSeconds=30
        BF.Settings.profile.MinFreeBagSlotsToGoToTown=9
        BF.Settings.profile.WallDistance=2
        BF.Settings.profile.Script4Name=''
        BF.Settings.profile.UseConsoleDirect=true
        BF.Settings.profile.FoodName=''
        BF.Settings.profile.NotAttackWhenGoToTown=false
        BF.Settings.profile.MailGray=false
        BF.Settings.profile.MainWindw_Left=1420
        BF.Settings.profile.AutoStart=false
        BF.Settings.profile.MailWhite=false
        BF.Settings.profile.ShortUnMountTeleportDistance=3
        BF.Settings.profile.MailStartMoney='1'
        BF.Settings.profile.AutoSkinning=false
        BF.Settings.profile.AutoCancelDuel=true
        BF.Settings.profile.ResolvedKeepType=''
        BF.Settings.profile.DestroyItemList='OOX-17/TN定位器|一本破旧的历史书|小蚌壳|瓦希塔帕恩的羽毛|奥瓦坦卡的尾刺|拉克塔曼尼的蹄子|被撕破的日记|西部荒野地契'
        BF.Settings.profile.TeleportAlert=true
        BF.Settings.profile.StorageGray=false
        BF.Settings.profile.AbnormalRotationAlert=false
        BF.Settings.profile.ScriptMode=1
        BF.Settings.profile.UseGoldMail=false
        BF.Settings.profile.ClassCombatDistance=4
        BF.Settings.profile.CollectID=''
        BF.Settings.profile.GetDetailedroledata=true
        BF.Settings.profile.NoRestInWater=true
        BF.Settings.profile.DisTaskValue=false
        BF.Settings.profile.DeadResetInstances=false
        BF.Settings.profile.CSCorpspots=''
        BF.Settings.profile.ArenaQueue=4
        BF.Settings.profile.NoAttackInSwim=false
        BF.Settings.profile.AlterSoundTime=1
        BF.Settings.profile.PetFoodAmountToGoToTown=1
        BF.Settings.profile.Script3Name=''
        BF.Settings.profile.CSMobIds='18468,18476,18466'
        BF.Settings.profile.ScriptVersion=1
        BF.Settings.profile.CollectBlackList=''
        BF.Settings.profile.UseFlyMountUpZ=35
        BF.Settings.profile.ignoreFlags=0
        BF.Settings.profile.HaveConsult=false
        BF.Settings.profile.MountCheckTime=5
        BF.Settings.profile.AfterSecondSendMessage=3
        BF.Settings.profile.PassVielMonster=false
        BF.Settings.profile.PassMonsterNum=2
        BF.Settings.profile.CollectIDCount=400
        BF.Settings.profile.CombatNoRevivePet=false
        BF.Settings.profile.NoNavRandomMove=false
        BF.Settings.profile.UsePotions=false
        BF.Settings.profile.AutoGatherBox=false
        BF.Settings.profile.UseConsole=false
        BF.Settings.profile.NavRandomJumpChance=0.2
        BF.Settings.profile.BlackCollectName2=''
        BF.Settings.profile.PassPlayer=false
        BF.Settings.profile.CollectLimitTime=150
        BF.Settings.profile.TimeWork2='10:00to23:00'
        BF.Settings.profile.BoomWait=3
        BF.Settings.profile.MailSubject='邮寄物品'
        BF.Settings.profile.TurnAfterSent=185
        BF.Settings.profile.TimeChangeGameAccountRate=1800
        BF.Settings.profile.CollectUsePolymorph=false
        BF.Settings.profile.NewVersionTip=true
        BF.Settings.profile.MeNoBuff=''
        BF.Settings.profile.StorageWhite=false
        BF.Settings.profile.BeforeRevivePetUseFeigndeath=true
        BF.Settings.profile.HTTPDelay=1
        BF.Settings.profile.QuikA=false
        BF.Settings.profile.UseFood=true
        BF.Settings.profile.PetHPMaxPercent=65
        BF.Settings.profile.JustSkinMyself=false
        BF.Settings.profile.CollectItemsInWater=true
        BF.Settings.profile.AttackPlayer=true
        BF.Settings.profile.MailGreen=true
        BF.Settings.profile.NavigationCoreectInWater=true
        BF.Settings.profile.WhisperFriendToggle=false
        BF.Settings.profile.ActivAttackPlayer=false
        BF.Settings.profile.IgnoreServerRoadsWater=false
        BF.Settings.profile.MsgLv=5
        BF.Settings.profile.DesroyItemLessPrice=20
        BF.Settings.profile.UnMountTeleportDistance=3
        BF.Settings.profile.NoUseCTM=false
        BF.Settings.profile.ExTaskScript=''
        BF.Settings.profile.TeleportMessage='???'
        BF.Settings.profile.ObjUpdateInter=true
        BF.Settings.profile.MageCombatDistance=30
        BF.Settings.profile.SearchRadius=300
        BF.Settings.profile.NavgaionMapName='Classic_tbc'
        BF.Settings.profile.DestroyWhenLoot=true
        BF.Settings.profile.GMWhisperReply=true
        BF.Settings.profile.HunterCollectFinishAttack=false
        BF.Settings.profile.DrinkPercent=60
        BF.Settings.profile.DrinkName=''
        BF.Settings.profile.ChangeScript4Condition=''
        BF.Settings.profile.SellBlue=false
        BF.Settings.profile.HunterRevivePetlv10=false
        BF.Settings.profile.MainFrame_point='BOTTOMRIGHT'
        BF.Settings.profile.GoToTownChangeScriptName='GL-Tinkr/拍卖脚本/自定义上架拍卖加密.lua'
        BF.Settings.profile.MainWindow_Left=630
        BF.Settings.profile.HunterCombatDistance=34
        BF.Settings.profile.AttackBlackList=' '
        BF.Settings.profile.KeepFrame=true
        BF.Settings.profile.TimeLogOut=false
        BF.Settings.profile.PlayerWhisperAlert=false
        BF.Settings.profile.BFLoginFailAutoRestart=true
        BF.Settings.profile.SellGray=true
        BF.Settings.profile.ResolvedPurple=false
        BF.Settings.profile.PetHPPercent=50
        BF.Settings.profile.CSName='泰罗卡测试.lua'
        BF.Settings.profile.DestroyGray=false
        BF.Settings.profile.AmmoAmountBuyPercentToGoToTown=true
        BF.Settings.profile.BFNavXYZ=''
        BF.Settings.profile.ShortTeleOnlyAlert=false
        BF.Settings.profile.TeleportSmartRate=0.6
        BF.Settings.profile.AutoRun=true
        BF.Settings.profile.FullBagChangeScript=false
        BF.Settings.profile.MailBlue=true
        BF.Settings.profile.MoeneyToGoToTown=1000000
        BF.Settings.profile.WhisperAlertStop=false
        BF.Settings.profile.TwoPeopleInInstanceAlert=false
        BF.Settings.profile.BlackCollectName1=''
        BF.Settings.profile.AutoRelog=false
        BF.Settings.profile.ExTaskScriptIndex=49
        BF.Settings.profile.AutoRunSecond=5
        BF.Settings.profile.AlwaysCallPet=true
        BF.Settings.profile.DRUIDNavUseTravelForm=false
        BF.Settings.profile.HPPercent=10
        BF.Settings.profile.PetFoodName=''
        BF.Settings.profile.AmmoAmountToGoToTown=0
        BF.Settings.profile.HunterDistanceAttack=true
        BF.Settings.profile.StorageGreen=false
        BF.Settings.profile.WhisperFriendName=''
        BF.Settings.profile.SetPitch0=true
        BF.Settings.profile.CSRepair='{ Name = '泰罗巨蛾', Entry = 18468, X = -2057.4419, Y = 4498.6035, Z = 9.4202, Distance = 35.619173225855, Radius = 10 }'
        BF.Settings.profile.DestroyGreen=false
        BF.Settings.profile.StorageItemName=''
        BF.Settings.profile.OLScriptName='本地脚本'
        BF.Settings.profile.SayAfterTele=true
        BF.Settings.profile.UseStraightPath=false
        BF.Settings.profile.CollectBlackPoint=''
        BF.Settings.profile.ResolvedGreen=true
        BF.Settings.profile.ResolvedBlue=true
        BF.Settings.profile.UseMail=false
        BF.Settings.profile.DestroyWhite=false
        BF.Settings.profile.SendMessageRate=30
        BF.Settings.profile.UseDrink=true
        BF.Settings.profile.MountName=''
        BF.Settings.profile.VagueDestroyItemList=''
        BF.Settings.profile.SmartJudgeTeleportDistance=false
        BF.Settings.profile.NoAttackRedMob=true
        BF.Settings.profile.DesroyAccordingPrice=false
        BF.Settings.profile.MainWindow_Top=220
        BF.Settings.profile.TeleportDistance=15
        BF.Settings.profile.CreateCSMobIds=''
        BF.Settings.profile.AutoGatherOre=true
        BF.Settings.profile.SellPurple=false
        BF.Settings.profile.BlackAnyCollectNameCount=150
        BF.Settings.profile.BugVer=1
        BF.Settings.profile.RDiffSum=800
        BF.Settings.profile.BlackCollectNameCount3=400
        BF.Settings.profile.MountDistance=68
        BF.Settings.profile.UseFlyMount=false
        BF.Settings.profile.ChangeGameAccountCondition='BF.GetComputerTime()==2202 or BF.GetComputerTime()==1402'
        BF.Settings.profile.AutoGatherParti=false
        BF.Settings.profile.DrinkMaxPercent=100
        BF.Settings.profile.RelogAfterTime=2100
        BF.Settings.profile.TriggerBlackList=false
        BF.Settings.profile.WhisperSendBack=true
        BF.Settings.profile.NotAttackWhenMounted=true
        BF.Settings.profile.WeakWaiting=false
        BF.Settings.profile.ignoreFlagstotal=2
        BF.Settings.profile.SafeRetrieve=true
        BF.Settings.profile.Script1Name=''
        BF.Settings.profile.AmmoAmount=0
        BF.Settings.profile.ScriptName='scripts/GL/Profile/Z脚本源码/专业技能练习源码/1-300-部落-采矿.lua'
        BF.Settings.profile.AutoClearCollectBlackList=false
        BF.Settings.profile.NavUseFeigndeath=true
        BF.Settings.profile.Script2Name=''
        BF.Settings.profile.ResolvedUnBind=true
        BF.Settings.profile.MPPercent=10
        BF.Settings.profile.AutoRelogIndex=1
        BF.Settings.profile.MaxBattleCount=1
        BF.Settings.profile.MailRecipient=''
        BF.Settings.profile.DoNotSellList=''
        BF.Settings.profile.AutoEquip=false
        BF.Settings.profile.MailRemoney='20'
}
-- Official service community communication function, Retail, Community
-- By default, only the last message sent by a user is retrieved. If you want to get previous messages, please store them yourself.
-- To send messages in the community,
-- To get the community ID
-- clubname, name of the community,
-- return clubid
function BF.GetClubID(clubname)
    -- To send a message in a specified community
    -- msg: message, the content of the message
    -- clubname: community name, name of the community
function BF.SendMessageInClub(msg, clubname)
    -- To get the chat messages from a specified community, by default the last one
    -- clubnameorID, community name or community ID
    -- time, seconds, default is 60. It fetches the messages from up to the last one sent within a specified time frame, default is 60 seconds,
    -- return, returns
    -- memberNote: string, the last message sent by a user
    -- messages: table, a table of messages, you can use BF.TableToString(messages) to print the content of the table
function BF.GetMessageFromClub(clubnameorID, time)

--记录信息,记录在BF目录下 调试信息.txt
function GL.WriteFile(txt)
end

--获取自己周围可通行的点
@return: x, y, z
function BF.CreateMovablePos(x, y, z, movdis, destdis)
end
--这个函数是法师任务学习天赋技能的
function BF.MageLearnTalent()
end
--这个函数是法师任务学习普通技能的
function BF.MageLearnSkill()
end
--参数1：NPC名称
--参数2：技能名称,
function BF.LearnSkill(szNpcName, szSkillName)
end

--BF.TimeLoginClose=true
function BF.TimeLogin()
end
--调试输出
function BF.msgbug(msg, ...)
end
--获取自己周围可通行的点
@return: x, y, z
function BF.CreateMovablePos(x, y, z, movdis, destdis)
end
--获取表中离自己最近点
function BF.GetPointNearSelf(list)
end

--获取NPC的功能
function BF.GetNpcFun(target)
end

--判断物品是否可以采集
@return: boolean(true or false)
function BF.GameObjectCanLoot(obj)
end
--进出副本
function BF.GoInDungeon(x, y, z, fx, fy, fz, list)
end
--GLHUDINTERRUPTS:Toggle(2)关闭打断
function BF.AutoFight(autoFight)
end
--GLHUDINTERRUPTS:Toggle(2)关闭打断
function BF.AutoHekili(autoFight)
end
--GLHUDINTERRUPTS:Toggle(2)关闭打断
function BF.AutoSelfRotation(autoFight)
end
--移动到目标击杀目标
function BF.GrindTo(x, y, z, id)
end
--获取表中离自己最近点
function BF.GetPointNearSelf(list)
end
--等待读条
function BF.WaitForAction()
end
--获取任务奖励
function BF.SetReward(id)
end
--自动穿装备
function BF.AutoEquipment(force, Rarity)
end
--参数1 技能ID或技能名字  参数2  要学技能等级 如果不传入参数 则所有学习所有等级
function BF.LearnSpell(id, nlevel)
end
--猎人学习宠物技能，
--skillname,技能名称，留空全学
--lv,技能等级，留空学习全部技能等级
--Learned,是否已学习，留空不判断，设置为true，返回是否学习，但是不进行学习
function BF.HunterTrainPet(skillname, lv, Learned)
end
--记录信息,记录在BF目录下 调试信息.txt
function BF.WriteDebug(txt)
end
--参数1 技能ID或技能名字  参数2  要学技能等级 如果不传入参数 则所有学习所有等级
function BF.LearnSpell(id, nlevel)
end
--公会银行参数
function BF.OpenGuildBankTab(n)
end
--生活技能练习附魔用
--spellid,生活技能id
--name,附魔名称
--equipid,要附魔的装备位置1到19，或者要附魔到的物品id,附魔羊皮纸38682
--times,次数
function BF.DoTradeFM(spellid, NameOrID, equipid, times)
end
--奥格在传送点处跳下
function BF.JumpDownInAG()
end
--部落沙塔斯接受任务
function BF.HordeAcceptBattleQuest()
end
--itemid 要找的物品id
--string 指定物品id的链接的第二行文字
--返回值1,是否找到物品
--返回值2,第几个背包
--返回值3,第几个空格
function BF.BagFindItem(itemid, string)
end
--x,y,z 宠物坐标
--func 回调函数
--range 停止范围
function BF.PetMoveTo(x, y, z, func, range)
end


--1为主天赋，2为副天赋
function BF.ExchangeTalent(index)
end
--参数1 人物名字
--参数2 交易物品名字
--参数3 交易物品数量
--参数4 交易超时时间
--参数5 交易金钱，交易金钱暂时不行
function BF.TradePlayer(name, stname, ammount, limittime, money)
end
--获取冬拥湖战斗开始 时间戳
function BF.CheckWintergrasp()
end
--排队战场
function BF.QueuedBattlefield()
end
--获取鱼漂
@return: v
function BF.GetFishFloat()
end
--获取是否上钩
@return: boolean(true or false)
function BF.FishFlag(float)
end
--游泳到海底
function BF.SwimmingToLand()
end
--重新运行传送前脚本
function BF.GetExOrScript()
end
--打怪过程中使用物品
function BF.MobUseId(target)
end
--打怪过程中回调
@return: boolean(true or false)
function BF.MobCallBack()
end
--全局添加
function BF.GlobalAdd()
end
--取消非最低价的物品
function BF.AuctionCancelHigherPrice(delay)
end
--取消寄售
function BF.CancelAuction()
end
--交易行获取物品价格
--tname,要查询的物品名称或id
--wait,查询后等待时间,秒，数字，可省略
--index,查询第几个的价格，数字，小于50，可省略
--sort,可省略，默认排序，可以是排序函数 默认排序是 function() SortAuctionClearSort('list') SortAuctionSetSort('list', 'unitprice', true) end
function BF.GetAuctionPrice(tname, wait, index, sort)
end
--交易行购买物品
function BF.AuctionBuy(tname, amount, rolename, keepgold, protectprice, mustbuygold, wait, sort)
end
--获取交易行价格，需要打开并放置物品到交易行，使用tdauction 插件
@return: BF
function BF.Get_AutctionPrice(tname, wait, index, sort)
end
--参数1：雕文id或者名称
--参数2：slot
--大型雕文 1,4,6
--小型雕文 2,3,5
function BF.EquipGlyph(NameOrID, slot)
end
--获取背包中有的指定部位的装备分数，没有返回0
--参数 position 装备位置1到19
function BF.GetBagEquipmentScore(position)
end
--遍历从npc处购买装备
function BF.BuyEquipment()
end
--参数2 ”player“,可省略，如果插件pawn不返回装备分数,设置为player返回装备等级
function BF.GetEquipmentScore(link, sign)
end
--获取荣誉值
@return: honorpoint
function BF.GetHonorpoint()
end
--获取竞技场点数
@return: Areapoint
function BF.GetAreapoint()
end
--从NPC处购买装备，
--参数1 npc的名称或者id
--参数2 荣誉值超过多少可以购买，可省略
function BF.BuyEquipmentFromNpc(idOrName, Honor)
end
--判断队友是否在身边
--参数1：队友名称，可空，不填写判断所有队友是否在身边，填写判断指定队友是否在身边
--参数2：范围，可空，默认60
@return: true
function BF.IsTeammateNearby(teammateName, range)
end
--换背包耐久度更高的装备
--参数1：物品在身上的位置1到19，默认17：盾牌
--参数2：耐久度1到100，默认10
function BF.Ex_HigherDurabilityItem(slot, durabilityPercent)
end
--获取在那个界面天赋加点最多
--返回 1,2,3
function BF.GetTalentTabIndex()
end
--判断装备是否比身上或者背包中的好
--参数1:物品链接
function BF.IsEquipmentBetter(link)
end
--加入组队邀请
function BF.JoinLFG()
end
--给中控系统发送信息
function BF.SendInfoToConsole(info)
end
--等待的角色退出
function BF.WaitRoleLogout()
end
--返回游戏剩余时间，3个值
--1，剩余天数
--2，具体日期
--3，剩余分钟数
@return: rd, oldDate, remaining

function BF.GetGameExpireTime()
end
--从Leatrix_Maps这个插件的箭头值，来提取地图之间互通的坐标。
function BF.CreateCROSS_MAPTabele()
end
--创建飞行点表
function BF.CreateFlyNodeTable()
end
--创建npc表
function BF.CreatNPCTable(tab, type)
end
--创建Object表
function BF.CreateObjectTable(tab, type)
end
--记录飞行点
function BF.RecordFlyNode()
end
--获取指定地图的子地图
--947
function BF.GetChildMapIDs(parentMapID)
end
--获取地图id所处的大陆id
@return: 1
function BF.GetCID(mapid)
end
--获取指定地图ID上一点的x,y,z,坐标，
--mapid:mapid
--cx:地图插件的光标位置x
--cy:地图插件的光标位置y
@return: wx, wy, 0
function BF.GetMapPostion(mapid, cx, cy)
end
--用于算跨地图寻路的路径,
--startMapID,起始地图
--targetMapID,终点地图
--return waylist 到达另外地图的路径坐标
--return mapidlist 到达终点地图需要经过的地图ID
@return: nil, nil
function BF.BFS(startMapID, targetMapID, crossMap)
end
--坐船或者飞艇跨地图
--end_id ，终点地图id
--start_id,起始地图id
function BF.CrossContient(start_id, end_id)
end
--获取表中离自己最近的点
--list为{{ X = -1590.6970 ,Y = 2469.1680, Z = 0.0000 },{ X = -1590.6970 ,Y = 2469.1680, Z = 0.0000 },} 形式
function BF.GetFlyNodeNearSelf(list)
end
--获取表中离目标点最近点
--list为{{ X = -1590.6970 ,Y = 2469.1680, Z = 0.0000 },{ X = -1590.6970 ,Y = 2469.1680, Z = 0.0000 },} 形式
function BF.GetFlyNodeNearGoal(x, y, z, list)
end
--获取使用飞机到达指定地图指定地点
--x,y,z,坐标
--end_id,到达地图ID 可省略start_id,px,py,pz
function BF.FindUseTaxi(x, y, z, end_id, start_id, px, py, pz)
end
--获取使用飞机到达指定地图指定地点
--end_id,到达地图ID 可省略start_id,px,py,pz
function BF.UseTaxiToMap(end_id, start_id, px, py, pz)
end
--和NPC交互
function BF.InteractNpc(npcid, option, x, y, z, waylist)
end
--导航到指定地图
--arg :mapid,地图id
function BF.ListMoveToMap(arg)
end
--寻找地图之间或者跨大陆的路径
--x,y,z,坐标
--end_id,终点地图id
--start_id,起始地图id，可省略
--range,寻路停止范围，可省略
--callback，回调函数，可省略
function BF.NavToMap(x, y, z, end_id, start_id, range, callback)
end
--寻找地图之间或者跨大陆的路径
--x,y,z,坐标
--end_id,终点地图id
--start_id,起始地图id，可省略
--range,寻路停止范围，可省略
--callback，回调函数，可省略
function BF.RunToMap(x, y, z, end_id, start_id, range, callback)
end
--导航到指定地图
--end_id,终点地图id
--start_id,起始地图id，可省略
--range,寻路停止范围，可省略
--callback，回调函数，可省略
function BF.NavToMapId(end_id, start_id, range, callback)
end
--寻路到地图某一个点
--x,y,地图插件坐标
--mapid,地图id，省略为当前地图
function BF.MapNavTo(x, y, mapid)
end
--获取最近的指定类型NPC
--type:Food ,Drink,Repair,Ammo
--返回 x,y,z ,Entry
--获取最近的指定类型NPC
--type:Food ,Drink,Repair,Ammo
--返回 x,y,z ,Entry
function BF.FindNearNpc(type, list)
end
--获取内置回城NPC添加到脚本
function BF.UseInnerTownNpc()
end
--根据内置NPC实行回城操作
function BF.SRM()
end
--把x,y,z文本坐标
--把GLWayPiont里的路径抓换为{ X = -775.5635 ,Y = 2104.5559, Z = 111.0000 }的形式
function BF.TxTToTable(arg)
end
--把表格写出
function BF.Net.TableToString(tbl, level, filteDefault, printedTables, path)
end
--写出文件信息
function BF.CreateCustomScript2(ScriptName, Hotspots, mobids)
end
--根据Routes插件生成所有采集路线,路径在“scripts/GL/Profile/Gl自己生成的脚本/“
--提前安装Routes插件生成采集路线，可能需要后续自己手动优化。
function BF.CreateScriptWithAddonRoute()
end
--根据Routes插件生成当前采集路线,路径在“scripts/GL/Profile/Gl自己生成的脚本/“
--提前安装Routes插件生成采集路线，可能需要后续自己手动优化。
function BF.ImproveScriptWithAddonRoute()
end
--为内置两个变量设置成BF的变量
--自动初始化    OnLoad(self)
--BF.Cube_codeEditor =codeEditor
--BF.Cube_cboSippets =cboSippets
--增加一个保存按钮 ，把Cube.lua,CubeEditor.lua,Browser.lua 按顺序添加进去
function BF.CubeEditor()
end
--打开编辑器
function BF.OpenCubeEditor(index)
end
--保存编辑器内容
function BF.CubeEditorSaveFile()
end
--面向角色背后
function BF.PlayerFaceBack()
end
--把当前zygor的任务流程转换为表，打印出来，路径"scripts/GL/打印的表.lua"，更改为想使用的文件名称
--BF.TableToFile(ZGV.CurrentGuide.steps)
function BF.TableToFile(tbl, level, printedTables, parentKey)
end
--获取NPC的功能
function BF.GetNpcFun(target)
end
--寻找视野范围内的邮箱，返回x,y,z
@return: a
function BF.FindMailBox()
end
--奥格上电梯
function BF.Ag_DtUp(num)
end
--以下函数为考古相关函数
--获取表中离自己最近的坐标点
--return  x,y,z
function BF.GetNearestPoint(list)
end
--获取指定大陆考古信息，返回表。
--参数1：大陆ID，可忽略。默认获取当前大陆ID，可以使用BF.GetMapId()获取当前大陆ID.
--return sult ,BF.TableToFile(sult)来查看考古信息
function BF.GetArchaeologyInfo(cid)
end
--有宝物时挖掘考古物品
function BF.DigArchaeologItem()
end
--在挖掘场自动挖掘宝物
function BF.FindArchaeologyItem(fly, setdis)
end
--采集白名单
@return: boolean(true or false)
function BF.IsInNeedCollectList(x, y, z)
end
--游泳到海底
function BF.SwimmingToLand()
end
--添加的BF函数
function BF.InteractNpc(npcid, option, x, y, z, waylist)
end
--Get npcid,x,y,z with title
function BF.GetNpcByTitle(title)
end
--可能有人使用这个函数，无限连接GL服务器，设置为空
function GL.Post()
end
function GL.LoadTinkrCoverter()
end
function GL.GossIpClosed()
end
@return: instance
function BF.Class()
end
function GL.InitSettings()
end
function GL.getUpdateRate()
end
function GL.UpdateOM()
end
function BF.SplitInput(Input)
end
@return: boolean(true or false)
function BF.BF_ObjectIsBehindA(unit1, unit2, degrees)
end
function GL.SmartCast(spell, Unit)
end
function GL.TableToString(tbl, level, filteDefault)
end
function GL.Helpers.Gatherers.Run()
end
function GL.Helpers.Trackers.Run()
end
@return: sX, 
function GL.Helpers.DrawLineGLC(sx, sy, sz, ex, ey, ez)
end
function GL.Helpers.QuestieHelper.Run()
end
function BF.DrawRoute()
end
function BF.export(value)
end
function BF.UI.Show()
end
function BF.UI.ShowTracking()
end
function BF.UI.Init()
end
function BF.UI.AddHeader(Text)
end
function BF.UI.AddToggle(Name, Desc, Default, FullWidth)
end
function BF.UI.AddToggle(Name, Desc, Default, FullWidth)
end
@return: GL
function BF.UI.AddRange(Name, Desc, Min, Max, Step, Default, FullWidth)
end
@return: GL
function BF.UI.AddDropdown(Name, Desc, Values, Default, FullWidth)
end
@return: GL
function BF.UI.AddInput(Name, Desc, Default, FullWidth)
end
function BF.UI.AddBlank(FullWidth)
end
function BF.UI.AddTab(Name)
end
function BF.UI.AddText(Text)
end
@return: GL
function BF.UI.AddLabel(Name)
end
function BF.UI.InitQueue()
end
function GL.GetMonsterNumber(X, Y, Z, Yards)
end
function GL.CanMoveBack()
end
function BF.TimeDelayTrue(sign, second)
end
@return: expirationTime
function GL.AuraRemain(unit, idOrName)
end
function BF.GL_RunScript(string)
end
function GL.RunScript(string)
end
function GL.LoadRotation()
end
function GL.WriteSelfRotation(sr_info)
end
function GL.ClearSelfRotation()
end
@return: cnstr

function GL.SetUI(cnstr, enstr)
end
function GL.UI.Add_SR_Tab(index, Name, iconid, spellID, enname)
end
@return: boolean(true or false)
function GL.UnitHasAura(unit, idOrName)
end
function GL.RunSelfRotation()
end
function GL.GetHekiliRecommenSpellID(index)
end
function GL.RunHekiliRotation()
end
@return: BF
function BF.ScirptNameUI(txt)
end
function BF.LoadTinkrCoverter()
end
@return: bsinfo
function BF.ADD_GLOBAL()
end
function BF.OpenTelegram()
end
function BF.BF_RunScript(string)
end
@return: boolean(true or false)
function BF.Split(input, delimiter)
end
@return: select
function BF.GetMapId()
end
function BF.TimeDelayTrue(sign, second, jumpone)
end
function BF.GetLastClickInfo()
end
function BF.ReAddMailFriend()
end
function BF.initialization()
end
function BF.Update_Unix()
end
@return: boolean(true or false)
function BF.Split(input, delimiter)
end
function BF.RunScriptAesEncrypted(content)
end
function BF.AesKernEnScript(path, path2)
end
@return: else
function BF.RunOlScript(content, line)
end
function BF.AutoUpdateScript(serverp, localp)
end
function BF.CollectListMove(list, fromstart)
end
function BF.BFGetPointNearSelf(list)
end
function BF.ListMove(x, y, z, arg, fun1)
end
function BF.BFGetPointNearSelf(list)
end

function BF.TaskRunBF(content)
end
function BF.ExchangeBotType(content)
end
@return: fbtime
function BF.GetRecentFBTime()
end
@return: boolean(true or false)
function BF.DeleteQuest(questId, all)
end
function BF.ShopBuy(NameOrId, count, smart)
end
function BF.MoveDelay(x, y, z, time)
end
function BF.AGToFlyNode()
end
@return: a
function BF.LFGGoToCityDeal()
end
@return: a
function BF.TLZMoveToLFG()
end
function BF.TLBMoveToBFC()
end
function BF.MakeNavPostionUnpass(x, y, z, range, mapId)
end
function BF.Grind(value)
end
function BF.TaskLearnSkill(nodt)
end
function BF.TakeAirShip(list, waitnext)
end
function BF.GLAuthor(RePoints)
end
@return: boolean(true or false)
function BF.SeniorAuthor(RePoints)
end
function BF.EquipBetterBag()
end
@return: boolean(true or false)
function BF.AddToolTip()
end
function BF.HunterSaveOneself()
end
@return: boolean(true or false)
function BF.SelectLowestHPMonster()
end
function BF.SelectLowestHPPlayer()
end
function BF.CreateTaskPoint()
end
function BF.CreateSelfPoint()
end
@return: x,y,z
function BF.GetPoistion(value)
end
function BF.GetTablePoistion(value)
end
function BF.HunterScanList(inlist)
end
function BF.ScanForCollect(x, y, z, nOuttime)
end

@return: 
function BF.GetGameTime()
end
@return: tonumber
function BF.GetComputerTime()
end
function BF.GetLocalGameTime()
end
@return: boolean(true or false)
function BF.enUSKey(RePoints)
end
@return: gold or 0
function BF.GetPalyerToTalMoney()
end

function BF.Bots.Restart()
end

function BF.CheckInInstance()
end
function BF.AfterTeleAction()
end
function BF.ForceAlert(msg)
end

@return: boolean(true or false)
function BF.LootPlayer()
end
function BF.Bots.Pulse()
end
function BF.RunTaskScript(content)
end
function BF.ExchangeScript(scriptName, fromor, index)
end
function BF.Logout(disconnect)
end
function BF.ChangWoWAccountIndex(index)
end
function BF.UpdateRelogInfo()
end

function BF.ReadRoleConfig(configure)
end
function BF.WriteRoleConfig(configure, value)
end
function BF.ReadConfig(configure, pid)
end
function BF.WriteConfig(configure, value)
end
function BF.InitSettings()
end
function BF.GetRelogAfterTime()
end
function BF.BattleChangeAccount()
end
function BF.TimeLoginSetting(immediately)
end
function BF.MultiTimeLogin()
end
function BF.OLTimeReachLogOut()
end
@return: boolean(true or false)
function BF.TriggerOffline()
end
function BF.TriggerBlackCollectList()
end
function BF.ChangeScriptSetting()
end
function BF.GetTodayGameTime()
end
@return: GetTime
function BF.GetScriptTime()
end
function BF.GetScriptName()
end
function BF.GetDayTimestampRange(timestamp)
end
function BF.update_Instancetable()
end
function BF.ChangeGameAccount()
end
function BF.GetBattleCount()
end
function BF.BFDeadCount()
end
function BF.GetCollectCount(objid)
end

function BF.LoadRotation()
end
function BF.AuthUserReg()
end
function BF.AuthCardReCharge()
end
function BF.GetAuthRePoints()
end
function BF.GetAuthExpireTime()
end
@return: string
function BF.UserReg()
end
function BF.GetDataValue()
end
function BF.CardReCharge()
end
function BF.ChangeBinding()
end
@return: instance
function BF.Class()
end
function BF.ReverseTable(t)
end
function BF.Log.Print(msg, ...)
end
function BF.Log.Status(msg, ...)
end
function BF.Log.System(msg, ...)
end
function BF.pmsg(msg, ...)
end
function BF.msg(msg, ...)
end
function BF.Alert(message1, message2, sound, fadetime, texture, refresh_old)
end
function BF.BFLog(content, size, sx, sy)
end
function BF.Log.Frame(msg, size, position)
end
function BF.Log.UIEr(message)
end
function BF.Log.Write(msg, ...)
end
function BF.Log.P(msg, ...)
end
function BF.Log.D(msg, ...)
end
function BF.Log.Time(msg, ...)
end
function BF.Log.Debug(msg, ...)
end
function BF.Log.Error(msg, ...)
end
function BF.Log.Warning(msg, ...)
end
function BF.Log.PathFly(msg, ...)
end
function BF.Log.Path(msg, ...)
end
function BF.Log.WriteFile(msg)
end

function BF.WebApi.PlaySound(type, message)
end
@return: string
function BF.WebApi.UrlEncode(s)
end
@return: string
function BF.WebApi.UrlDecode(s)
end
function BF.BuyAllTrainerService()
end
function BF.GetAreaId()
end
function BF.IsInBattlefield()
end
function BF.getUpdateRate()
end
function BF.ObjectManager.GetWoWUnitById(idOrName)
end
@return: Object
function BF.ObjectManager.GetGameObjectById(idOrName, orderBy)
end
@return: Object
function BF.ObjectManager.GetGameObjectByIdAndOrder(idOrName, orderBy)
end
@return: obj
function BF.ObjectManager.GetWoWUnitByIdAndOrder(idOrName, orderBy)
end
@return: Unit
function BF.ObjectManager.GetWoWUnitByGUID(guid)
end
function BF.ObjectManager.GetGameObject(condition, orderBy)
end
function BF.ObjectManager.GetWoWUnits(condition, orderBy)
end
function BF.ObjectManager.GetFriend(condition, orderBy)
end
function BF.ObjectManager.GetNumberAttackPlayer()
end
@return: UnitAttackPlayer
function BF.ObjectManager.GetUnitsAttackPlayer()
end
function BF.ObjectManager.GetUnitAttackPlayer(condition, orderBy)
end
function BF.ObjectManager.GetMonsterCount(x, y, z, range)
end
function BF.ObjectManager.GetPlayerCount(x, y, z, range)
end
@return: not unit
function BF.ObjectManager.GetMonsterNumber(x, y, z, range)
end
@return: not unit
function BF.ObjectManager.GetNearestEnemy(x, y, z, range)
end
function BF.ObjectManager.GetEnemyPlayerCount(range)
end
@return: x
function BF.ObjectManager.GetNearestEnemyPlayer()
end
@return: x
function BF.ObjectManager.GetNearestUnitAttackPlayer()
end
function BF.ObjectManager.GetNearestUnitAttackPlayerB()
end
@return: x
function BF.ObjectManager.GetLowHPUnitAttackPlayer()
end
function BF.ObjectManager.SimulateHerbs(id)
end
function BF.ObjectManager.SimulateOre(id)
end
function BF.ObjectManager.ClearSimulates()
end
@return: boolean(true or false)
function BF.Navigator.MoveToUnit(Unit, distance, force)
end
@return: boolean(true or false)
function BF.NotFlyPoint()
end
@return: boolean(true or false)
function BF.Navigator.MoveToUnitCombatReach(Unit, distance, force)
end
@return: boolean(true or false)
function BF.MountFly()
end
@return: mx, my, mz
function BF.AvoidObjects(x, y, z)
end
function BF.Navigator.MoveTo(x, y, z, distance, useEWT, supplementaryPath, force, callback)
end
@return: not BF
function BF.IsUseingNetNav()
end
function BF.Navigator.MoveToInternal(distance, callback)
end
function BF.Navigator.MoveToPath(path, distance, generatePath, startNear)
end
function BF.Navigator.MoveToInPath(x, y, z, path)
end
function BF.Navigator.MoveToVendorPath(x, y, z, path)
end
@return: boolean(true or false)
function BF.Navigator.Rest()
end
function BF.Navigator.MoveStop()
end
function BF.Navigator.Stop(stopMoving)
end
function BF.Navigator.StuckReport()
end
@return: str, L
function BF.Navigator.GetPathCorrectionInfo()
end
@return: nil
function BF.Navigator.FindStuckPathCorrection(x, y, z, endX, endY, endZ)
end
@return: nil
function BF.Navigator.FindPathCorrection(x, y, z, endX, endY, endZ)
end
function BF.DrawRoute()
end
@return: boolean(true or false)
function BF.HasQuest(questId)
end
function BF.IsQuestCompleted(questId, subquest)
end
@return: IsQuestCompleted
function BF.IsObjectiveComplete(index, questId)
end
@return: boolean(true or false)
function BF.GetQuestCompleted(questId)
end
function BF.AutoVendor.OnMerchantShow()
end
function BF.AutoVendor.BuyFoodAndDrink()
end
function BF.AutoVendor.DoRepair()
end
function BF.AutoVendor.BuyAmmo()
end
@return: boolean(true or false)
function BF.AutoVendor.IsProtectedItem(item)
end
@return: boolean(true or false)
function BF.AutoVendor.IsNotSellItem(item)
end
@return: boolean(true or false)
function BF.AutoVendor.IsSellItem(itemInfo)
end
function BF.AutoVendor.SellJunk()
end
@return: boolean(true or false)
function BF.AutoVendor.NeedRepair()
end
@return: durabilityPercent
function BF.GetDurabilityPercent(slot)
end
function BF.AutoVendor.NeedBuyConsumable(typeName)
end
@return: boolean(true or false)
function BF.AutoVendor.NeedBuyFood()
end
@return: boolean(true or false)
function BF.AutoVendor.NeedBuyDrink()
end
function BF.AutoVendor.BuyFood()
end
function BF.AutoVendor.BuyDrink()
end
@return: i, _itemName
function BF.AutoVendor.GetMerchantItem(_itemName)
end
function BF.AutoVendor.GetBestConsumable(typeName)
end
@return: boolean(true or false)
function BF.Split(input, delimiter)
end
function BF.DoMail()
end
function BF.AutoVendor.AutoMail()
end
function BF.AutoVendor.SendMail()
end
function BF.AutoVendor.NeedMail()
end
@return: boolean(true or false)
function BF.AutoVendor.IsMailItem(itemInfo)
end
function BF.AutoVendor.DoMail()
end
function BF.Bag.GetBagItems(func)
end
function BF.Bag.DeleteItems(id)
end
function BF.DeleteMoreItem(id, keep)
end
function BF.Bag.DeleteItem(id)
end
@return: boolean(true or false)
function BF.Ammunition.NeedBuyAmmo()
end
function BF.Ammunition.DestroyLowAmmo(bestItemID)
end
function BF.Ammunition.EquipAmmo(bestItemName, szforce)
end
function BF.Ammunition.BuyAmmo()
end
function BF.BlackList.Add(guid, timeInMilisec)
end
function BF.BlackList.Remove(guid)
end
function BF.BlackList.RemoveAll()
end
@return: boolean(true or false)
function BF.BlackList.IsBlackListed(guid)
end
function BF.MovementManager.MoveDuringCombat()
end
function BF.UnstuckCheckNew(x, y, z)
end
function BF.MovementManager.Jump()
end
function BF.MovementManager.MoveTo(...)
end
@return: PathToUnit
function BF.MovementManager.PathToUnit(unit)
end
@return: boolean(true or false)
function BF.MovementManager.CanMoveTo(...)
end
@return: boolean(true or false)
function BF.MovementManager.testCanMoveTo1(...)
end
function BF.MovementManager.Stop()
end
function BF.MovementManager.DisableIdle()
end
function BF.MovementManager.EnableIdle(dontMove)
end
@return: getglobal
function BF.GetReputation(szName)
end
@return: expirationTime
function BF.AuraRemain(unit, idOrName)
end
function BF.UITableToString(tbl, level, filteDefault)
end
function BF.RunCode()
end
function BF.FixedZ()
end
function BF.export(value)
end
function BF.UI.ExtraTool.Show()
end
function BF.UI.SplitTool.Show()
end
function BF.UI.Script.Show()
end
function BF.UI.Script.Init()
end
@return: tonumber
function BF.UI.EnScript.Init()
end
function BF.UI.EnScript.Show()
end
function BF.UI.AutoUpdateScript.Show()
end
function BF.UI.Settings.Show()
end
function BF.UI.Kard.Show()
end
@return: false
function BF.UI.Profile.Init(title)
end
function BF.UI.Profile.Show()
end
function BF.UI.Profile.Hide()
end
function BF.UI.Profile.AddHeader(Name, EnName)
end
function BF.UI.Profile.AddText(Name, EnName)
end
function BF.UI.Profile.AddMultiInput(Name, Desc, linenum, Default, FullWidth, EnName)
end
function BF.UI.Profile.AddInput(Name, Desc, Default, FullWidth, EnName)
end
function BF.UI.Profile.AddToggle(Name, Desc, Default, FullWidth, EnName)
end
function BF.UI.Profile.AddRange(Name, Desc, Min, Max, Step, Default, FullWidth, EnName)
end
function BF.UI.Profile.AddDropdown(Name, Desc, Values, Default, FullWidth, EnName)
end
function BF.UI.Profile.AddBlank(FullWidth)
end
function BF.UI.Profile.AddExe(Name, Desc, func, Default, FullWidth)
end
@return: BF
function BF.UI.Settings.Init()
end
function BF.BuyMount()
end
@return: boolean(true or false)
function BF.Mount.IsMounted()
end
function BF.Mount.Dismount()
end
function BF.IsFlyableArea()
end
function BF.FindMounts()
end
function BF.Mount.SmartMount()
end
@return: boolean(true or false)
function BF.PlayerHasMount(mountIDToCheck)
end
@return: boolean(true or false)
function BF.LootBodys()
end
function BF.Mount.MountUp(szforce)
end
@return: boolean(true or false)
function BF.Mount.CanMount()
end
@return: boolean(true or false)
function BF.IsInNoAttackList(x, y, z)
end
function BF.IsInAttackBlacklist(Name)
end
function BF.TaskBot.Start()
end
function BF.TaskBot.Stop()
end
function BF.NeedCombat()
end
function BF.StopAttack()
end
function BF.CanUseHearthStone()
end
function BF.UseHearthStone()
end
function BF.GetItemCooldown(itemID)
end
function BF.GoToTownChangeScript()
end
@return: boolean(true or false)
function BF.NeedGotoTown()
end
function BF.Stand()
end
@return: boolean(true or false)
function BF.CanRest()
end
@return: boolean(true or false)
function BF.TaskBot.Pulse()
end
@return: boolean(true or false)
function BF.TaskBot.DoQuest(name, value)
end
function BF.FindBestItemName(typeName)
end
function BF.FindItemName(typeName)
end
@return: not unit
function BF.GetEliteNumber(x, y, z, range)
end
@return: boolean(true or false)
function BF.PassElite(x, y, z)
end
@return: boolean(true or false)
function BF.PassMonter(x, y, z)
end
@return: boolean(true or false)
function BF.PassPlayer(x, y, z)
end
function BF.NewSearchGrindMob()
end
@return: boolean(true or false)
function BF.MySearchMob()
end
function BF.SearchGrindMob()
end
function BF.SearchGrindPlayer()
end
@return: boolean(true or false)
function BF.HasMobId(mobIds, id)
end
function BF.GatherUseId(target)
end
@return: boolean(true or false)
function BF.GrindBot.HasGatherId(id)
end
function BF.GrindGetPointNearSelf(list)
end
function BF.Log.TaskTip(str)
end
function BF.GrindBot.Run()
end
function BF.PullGrindMob()
end
@return: boolean(true or false)
function BF.PullMobs()
end
function BF.GrindBot.Start()
end
function BF.GrindBot.Stop()
end
@return: CollectingNow or false
function BF.GatherBot.IsCollectingNow()
end
function BF.GatherBot.Test()
end
function BF.GatherBot.Start()
end
function BF.GatherBot.Stop()
end
function BF.GatherBot.Reset()
end
function BF.GatherBot.Cancel()
end
@return: true
    else
function BF.CollectUseFeigndeath()
end
function BF.Dive(Z)
end
@return: boolean(true or false)
function BF.IsBlackPoint(x, y, z)
end
@return: boolean(true or false)
function BF.IsInNoCollectList(x, y, z)
end
@return: boolean(true or false)
function BF.IsInCollectBlacklist(Name)
end
function BF.MageCollectUsePolymorph()
end
function BF.MageCollectUseIceBlink()
end
@return: boolean(true or false)
function BF.IsPartiNpc(id)
end
function BF.RecordCollection(CollectingNow)
end
@return: boolean(true or false)
function BF.GatherBot.Pulse()
end
function BF.Gather()
end
function BF.WaitTime(time)
end
@return: BF
function BF.Wait(time)
end
function BF.PetSuperJumpTo(x, y, z, delay)
end
function BF.Move(x, y, z, stopdis, delay)
end
function BF.NavToInCombat(x, y, z, range)
end
function BF.NavTo(x, y, z, range, callback)
end
function BF.AutoAllFight(autoFight)
end
function BF.EnableRotation()
end
function BF.DisableRotation()
end
function BF.DungeonBot.SearchMob(id)
end
function BF.DungeonBot.Start()
end
function BF.DungeonBot.Stop()
end
function BF.DungeonBot.Pulse()
end
function BF._selectAttackTarget()
end
function BF._getNearestAttackTarget()
end
@return: UnitHealth
function BF.GetP_HP()
end
@return: UnitPower
function BF.GetP_MP()
end
@return: BF
function BF.Wait(time)
end
function BF.AvoidBoom(sec, jgcount, fbcount)
end
@return: boolean(true or false)
function BF.AllEnemiesHasDebuff(buffid)
end
@return: boolean(true or false)
function BF.UnitDebuffs(obj, idOrName)
end
function BF.BFMove2D(x, y, z, stopdis, delay)
end
function BF.GetMailBox()
end
function BF.CreateCustomScript()
end
function BF.WaitCasting()
end
function BF.TinkrOpenNpc(id)
end
function BF.BFOpenNpc(id)
end
@return: boolean(true or false)
function BF.MakePostionMoreCost(x, y, z, cost)
end
function BF.ResolveEquipMent(write)
end
function BF.BreakDownEquipment(LimitRarity, breakbind)
end
function BF.EquipBag()
end
function BF.Composparti()
end
function BF.TableToString(tbl, level, filteDefault, printedTables)
end
function BF.GetRewardEquipmentInfo(itemLink)
end
function BF.GetPlayerEquipmentInfo(itemEquipLoc)
end
@return: BF
function BF.GetLocalFlyNode(name)
end
function BF.UseFlyTaxi(szName)
end
function BF.TakeTaxi(NpcId, FlyNodeName, num)
end
function BF.WaitTaxi()
end
function BF.SetRotation(name, value)
end
function BF.SetRotation(name, value)
end
function BF.SetSetting(name, value)
end
@return: BF
function BF.GetSetting(name)
end
@return: unit
function BF.SelectUnitByLocationAndId(x, y, z, range, unitId, faceNow)
end
function BF.BFWriteFile(txt)
end
function BF.WriteFile(path, txt, cover, addtime)
end
function BF.BFLearnTalent(szname, lv, arg3, pet)
end
function BF.LearnTalent()
end
function BF.TalentNameToNum(table)
end
@return: boolean(true or false)
function BF.SafeRetrieve()
end
function BF.SuperJumpTo(x, y, z, delay)
end
@return: boolean(true or false)
function BF.CancelAura(idOrName)
end
@return: boolean(true or false)
function BF.IsInCoord(x1, y1, z1, x2, y2, z2, x3, y3, z3, x4, y4, z4)
end
function BF.JumpForZ(Z)
end
function BF.BFMove3D(x, y, z, stopdis, delay)
end
function BF.BFMoveZ(Z, hd)
end
function BF.BFMove2DZ(x, y, z, stopdis, delay)
end
function BF.FlyMoveToZ(x, y, z, pitch)
end
function BF.FlyMove(x, y, z, pitch)
end
function BF.JoinBattle()
end
function BF.MoveForZ(Z, hd, outtime)
end
function BF.TakePayMail(FreeBagSlots, takepaymail)
end
function BF.TakeMail(FreeBagSlots, takepaymail)
end
@return: boolean(true or false)
function BF.BagDestroyGray()
end
@return: boolean(true or false)
function BF.IsDestroyItem(itemInfo)
end
function BF.DestroyItem()
end
@return: boolean(true or false)
function BF.IsDestroyItem(itemInfo)
end
function BF.ForBlackUI(boolean, txt)
end
function BF.BlackUI(boolean, txt)
end
function BF.BFSetCharIndex(index)
end
function BF.BF_RunScriptInCore(string)
end
function BF.GL_RunScript(string)
end
function BF.LoadFile(path)
end
function BF.AutoUpdateScriptB(serverp, localp)
end
@return: math
function BF.AltClickMove()
end
function BF.AutoUpdateAll()
end
function BF.FixedStationCollect()
end
function BF.FixedCollect(NameorId)
end
function BF.Collect(Pointer, PosX, PosY, PosZ)
end
function BF.Collect(NameorId, dis)
end
function BF.Collect(Pointer, PosX, PosY, PosZ)
end
function BF.ListMoveIndx(x, y, z, arg, nav)
end
function BF.MoveToStation(x, y, z, func, range)
end
function BF.BFGetPointNearSelf(list)
end
function BF.ListMoveS(x, y, z, arg, nav)
end
function BF.MoveToStation(x, y, z, func, range)
end
function BF.BFGetPointNearSelf(list)
end
function BF.ListMove(x, y, z, Nav)
end
function BF.DoTrade(spellid, NameOrID, times)
end
function BF.CommonFlyCollectSet()
end
function BF.HunterFlyCollectSet()
end
function BF.LoadMap()
end
function BF.KiwiFarm()
end
function BF.WriteRoleDate()
end
function BF.RoleTableToString(tbl, level, filteDefault)
end
function BF.GetDailyRoleDate()
end
@return: 0
function BF.GetGuildBankFreeBag(tab)
end
function BF.GetGuildBankItem(item, Rarity, freebags)
end
@return: boolean(true or false)
function BF.IsNotStorageItem()
end
function BF.GuildBankDeposit()
end
function BF.ChatToFreind()
end
function BF.AcceptQuest(index, QuestName)
end
function BF.FinishQuest(index, QuestName, Reward)
end
function BF.SelectQuestName(name)
end
function BF.TaskSetting()
end
function BF.SaveAuctionPrice()
end
function BF.WriteAuctionDate()
end
function BF.RoleTableToString(tbl, level, filteDefault)
end
function BF.UseItem(itemid)
end
function BF.UseItemName(name)
end
function BF.UseTrinket(slotId)
end
function BF.UsePotions()
end
function BF.ClickReplaceEnchant()
end
function BF.UsePotionTrinket()
end
function BF.UseItem(itemid)
end
function BF.InviteGroup(szrolename)
end
@return: boolean(true or false)
function BF.RestartUseHearthStone()
end
@return: CalculateTotalNumberOfFreeBagSlots
function BF.GetFreeBagSlots()
end
function BF.SaveRoleData(filename, msg)
end
function BF.AutctionSRM()
end
@return: boolean(true or false)
function BF.BattleSuccessed()
end
function BF.OutlandToMainCity()
end
function BF.SaveGameAccount(value)
end
function BF.BattleSetting()
end
function BF.GetTableFun(tname, fname)
end
function BF.GetNearFlyNode(x, y, z)
end
function BF.GetFlyNodeName(szName)
end
function BF.HordOpenFlyNode()
end
function BF.MoveTo(x, y, z, range, mstop)
end
function BF.BF_MoveTo(tx, ty, tz, range)
end
function BF.OutPositon(x, y, z, arg, range, extra)
end
function BF.MoveOutStation(x, y, z, func, range)
end
function BF.BFGetPointNearSelf(list)
end
function BF.LimitBattleQueue()
end
function BF.ZCAlert()
end
function BF.ZombiesEvent()
end
@return: boolean(true or false)
function BF.NeedOptimizingFPS(callback)
end
@return: boolean(true or false)
function BF.NeedOptimizingFPS()
end
function BF.ExtraLimit()
end
function BF.CastTarget(szname)
end
function BF.DungeonListMove(x, y, z, arg)
end
function BF.BFGetPointNearSelf(list)
end
function BF.MoveToStation(x, y, z, func, range)
end
function BF.DMoveTo(x, y, z, callback, range, Nav)
end
function BF.GetRaidInfo()
end
@return: leader
function BF.GetRaidLeader()
end
function BF.TeamUp(partylist)
end
function BF.FindGLMessage(api, postmessage)
end
function BF.VehicleAimGetAngle()
end
function BF.PostToConsole(info)
end
function BF.GetBagAllItemCount()
end
function BF.ConnectConsole()
end
function BF.IsInMultiTime(timestr)
end
function BF.Draw(x, y, z, r, time, txt)
end
@return: boolean(true or false)
function BF.HasCreatePet(szid)
end
function BF.GetWeekDay(y, m, d)
end
function BF.SetGraphicsLow()
end
function BF.JoinBattleGround()
end
function BF.JoinArena()
end
function BF.Fish(x, y, z, count, enchanting, sx, sy, sz)
end
@return: boolean(true or false), losx, losy, losz
function BF.CreateLandMoveNode(fromX, fromY, fromZ, toX, toY, toZ, dis)
end
function BF.GetBankFreeSlot()
end
function BF.BankDeposit()
end
function BF.DealGLAccountWindow()
end
@return: string
function BF.PostToServer(message, api)
end
@return: true
    elseif not IsFlying
function BF.StopFlyDealWater()
end
function BF.StopFly()
end
function BF.MountFlyToZ(Z, hd, outtime)
end
@return: v
function BF.CreateFishNode(fromX, fromY, fromZ)
end
@return: boolean(true or false)
function BF.FishOnWater()
end
@return: boolean(true or false)
function BF.MoveToFishPoint(x, y, z, stopdis)
end
function BF.CollectFishSchool()
end
function BF.FishSchool(x, y, z, Pointer)
end
function BF.FishListMove(list, fromstart)
end
function BF.BFGetPointNearSelf(list)
end
function BF.GetBagItemCount(szname)
end
@return: math
function BF.IsFacing(obj1, obj2, degrees)
end
@return: math
function BF._ObjectIsFacing(obj1, obj2, degrees)
end
function BF.EquipWeap()
end
@return: boolean(true or false)
function BF.GatherCallBack()
end
@return: name, killingBlows, honorableKills, deaths, honorGained, faction, race, class, classToken, damageDone, healingDone, bgRating, ratingChange, preMatchMMR, mmrChange, talentSpec
function BF.GetBattleInfo(index)
end
function BF.RunRotationMacro()
end
function BF.LeaveParty()
end
function BF.PerCheck()
end
function BF.GetGM()
end
@return: boolean(true or false)
function BF.WhisperFilter(message)
end
function BF.UITableToString(tbl)
end
function BF.BFWriteFile(txt)
end
function BF.ScriptUITableToString(tbl)
end
function BF.BFWriteFile(txt)
end
function BF.Auctionator.AH.PostAuction(...)
end
function BF.Auctionator.AH.CancelAuction(auction)
end
@return: 0
function BF.SelectPrice(itemlink)
end
@return: boolean(true or false)
function BF.AuctionCanSellItem(itemname, price)
end
function BF.GetAuctionOwnerCount(itemID)
end
function BF.AutoAuction()
end
function BF.AutoAuction()
end
function BF.SaveAuctionPrice()
end
function BF.AUCTIONATOR_CONFIG()
end
function BF.DK_LearnSkill(usehearthstoneback)
end
function BF.CloseLuaErrorWarning()
end
@return: boolean(true or false)
function BF.AuctionSell(szname, limitprice, waittime, index, amount, force, ignoreself, cancel, useset)
end
function BF.SelectPrice(itemlink)
end

function BF.GetBankItem(item, Rarity, freebags)
end
function BF.GetTradeSkillSpellID()
end
function BF.SetPawn_Talent()
end
function BF.GetB_AuthCod(s, r, key)
end
function BF.RandomMoveInCombat()
end
function BF.StuckNRestart()
end
function BF.GetFactionInBattleField()
end
function BF.DungeonGrinding(tab)
end
function BF.JoinBattle_N1()
end
@return: boolean(true or false)
function BF.Dungeon_DoTask(tab)
end
function BF.F_Draw(draw)
end
function BF.Test_Draw(draw)
end
function BF.Nav_Draw(draw)
end
@return: new_x_forward, new_y_forward, z
function BF.find_adjacent_positions(x, y, z, angle, N)
end
function BF.GetMoveablePosition(dis, x, y, z)
end
function BF.TraceMove(x, y, z)
end
function BF.NewStuckMethod()
end
@return: mx, my, mz
function BF.AvoidObjects(x, y, z)
end
@return: boolean(true or false)
function BF.IsSubmerged(unit)
end
@return: points
function BF.SmoothPoints(points, angel, radius, counts)
end
@return: tab
function BF.SmoothPath(tab)
end
function BF.Test_Draw(draw)
end
function BF.SellItem(tab)
end
function BF.message(text, force)
end
function BF.TakeAndDeleteMail(FreeBagSlots, keep)
end
@return: C_WowTokenPublic
function BF.GetWowTokenPrice()
end
function BF.BuyWowToken()
end
function BF.UseWowToken()
end
function BF.GetGLTokenDay()
end
@return: mapid
function BF.IsInVas()
end
@return: childInfo
function BF.GetMapNameID(mapname, parentMapID)
end
function BF.VehicleCast(id)
end
@return: Near, list
function BF.GetMapFlyNode(mapid, list)
end
function BF.OpenNearFlyNode()
end
function BF.BFGetPointNearSelf(list)
end
function BF.WriteAllMap()
end
function BF.ImproveCrossMap()
end
function BF.CreateScriptWithAddonQuestie(item, monsterid)
end
function BF.WriteFile(path, itemName, mobids, hotspots_str)
end
function BF.CreatScript2UI()
end
function BF.InitBarMenu()
end
function BF.BarCreateButton(tx_path, msg1, msg2, des, onClickEvent, auto, st)
end
function BF.GetAllBagSolt()
end
@return: 0
function BF.GetEquipMentDurability()
end
function BF.GetTotalWealth()
end
@return: boolean(true or false)
function BF.UnitHasAura(unit, idOrName)
end
function BF.GetPerHGold()
end
function BF.GetUnlockerLeftTime()
end
function BF.BarUpdate()
end
function BF.OnQuit()
end
function BF.VehicleAimGetAngle()
end
function BF.AimAngle(pitch)
end
@return: boolean(true or false)
function BF.IsEquippableByPlayer(itemid)
end
@return: boolean(true or false)
function BF.IsEquippableByPlayer(itemLink)
end
function BF.DestroyAllQuestItem()
end
function BF.DeleteAllQuest()
end
function BF.ObjectManager.GetObject(idOrName, dis)
end
function BF.BFWriteFile(txt)
end
function BF.Ag_DtDown(num)
end
@return: tx, ty, tz
function BF.GetDigsitePosition(tip, dis)
end
function BF.CheckDisPerTime(time, range, callback)
end
@return: boolean(true or false)
function BF.IsInCataMap()
end
function BF.GetArchaeologItem(SzName)
end
function BF.FlyToLand()
end
function BF.GetPosZ(fromX, fromY, mapid)
end
function BF.GetTabZ(t, mapid, name)
end
function BF.GetRealZ(StartX, StartY)
end
function BF.GetMountTypeID()
end
function BF.ClickFinishQuest(questid)
end
function BF.GetZgvCollectionPath(tab)
end
function BF.CreateScriptWithAddonZgv()
end
function BF.GetLocalFlyNode(name)
end
function BF.UseFlyTaxi(szName)
end
function BF.TakeTaxi(NpcId, FlyNodeName, num)
end
function BF.WaitTaxi()
end
function BF.AutoGoTaxiNode(nodeid)
end
@return: boolean(true or false)
function BF.PlayerHasMount(mountIDToCheck)
end
function BF.FB_Fly(dis)
end
function BF.FB_Land(dis)
end
@return: children
function BF.GetFlagScore()
end
@return: mapid
function BF.IsInVas()
end
function BF.BindLocation(x, y, z, NpcId, szName)
end
function BF.GetRealZ(StartX, StartY)
end
@return: not BF
function BF.IsUseingNetNav()
end
function BF.SetAuctionDuration(duration)
end
@return: boolean(true or false)
function BF.IsInTown()
end
function BF.WaitLoading()
end
function BF.AcceptTask(questid, npcid, option, x, y, z, waylist)
end
function BF.SubmitTask(questid, npcid, option, x, y, z, waylist)
end
function BF.GetMapName(mapid)
end
function BF.WriteDebugFile(txt, path)
end

function BF.Net.UseEquipment(item, EquipLoc)
end
function BF.Net.ScriptCheck()
end
function BF.Net.InteractUnit(obj)
end
function BF.Net.ConfirmBinder()
end
function BF.Net.VehicleAimGetAngle()
end
@return: bsinfo
function BF.Net.GetContainerItemInfo(containerIndex, slotIndex)
end
@return: boolean(true or false)
function BF.Net.CheckBinderDist()
end
@return: boolean(true or false)
function BF.Zgv.stepcond(t)
end
function BF.Zgv.LearnTalent()
end
function BF.Zgv.UseHearthStone()
end
function BF.Zgv.GrindGetPointNearSelf(list)
end
function BF.Zgv.GetGoToPoint(x, y, z, list)
end
function BF.Zgv.GrindTab(tab)
end
function BF.Zgv.DoGrind()
end
function BF.Zgv.GoToTown()
end
function BF.Zgv.GetGuidePath(path)
end
@return: Zgv
function BF.Zgv.AddAction(key, t)
end
function BF.Zgv.stepmustdo(t)
end
function BF.Zgv.stepinit()
end
@return: boolean(true or false)
function BF.Zgv.stepfinish(t)
end
@return: C_QuestLog
function BF.Zgv.GetQuestName(questID)
end
function BF.Zgv.GetMapPosition(mapid, cx, cy)
end
function BF.Zgv.AddPathToUI()
end
function BF.Zgv.ClickFinishQuest(questid)
end
function BF.Zgv.AcceptQuest(index, QuestName)
end
function BF.Zgv.FinishQuest(index, QuestName, Reward)
end
function BF.Zgv.OpenNpc(id)
end
function BF.Zgv.InteractNpc(x, y, z, Entry)
end
function BF.Zgv.SRM()
end
function BF.Zgv.ListMove(x, y, z, arg, fun1, fun2)
end
function BF.BFGetPointNearSelf(list)
end
function BF.Zgv.ListMoveSuper(arg, fun1, fun2)
end
function BF.BFGetPointNearSelf(list)
end
function BF.Zgv.GrindTo(id, x, y, z)
end
@return: boolean(true or false)
function BF.Zgv.IsCollectable(id)
end
function BF.Zgv.AddNowGrind(mobids, hotspots, gatherids, condition)
end
function BF.Zgv.AddNowGrind_Hotspots(hotspots)
end