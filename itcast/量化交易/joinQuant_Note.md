##### 基本框架

````
  写法一：
  def initialize(context):
      run_daily(period,time='every_bar')
      这里是用来写初始化代码的地方,例子中就是选定要交易的股票为平安银行

  def period(context):
      这里是用来写周期循环代码的地方,例子中就是买100股的平安银行
      
  写法二：
  def initialize(context):
      g.security = '000001.XSHE' #g代表全局变量

  def handle_data(context,data):
      这里是用来写周期循环代码的地方,例子中就是买100股的平安银行
````

##### [定时器运行函数](https://www.joinquant.com/help/api/help?name=api_old#%E5%AE%9A%E6%97%B6%E8%BF%90%E8%A1%8C%E5%87%BD%E6%95%B0%E5%8F%AF%E9%80%89)

````
   # 按月运行
    run_monthly(func, monthday, time='09:00', reference_security)
    # 按周运行
    run_weekly(func, weekday, time='09:00', reference_security)
    # 每天内何时运行
    run_daily(func, time='09:00', reference_security)
````

##### [下单函数API](https://www.joinquant.com/help/api/help?name=api#order-method)

````
下单函数：
	order(g.security, 100) #买100股.security，如果为卖的话，可以写成负数
   order函数参数说明：
   		security：代码	
   		amount 交易数量, 正数表示买入, 负数表示卖出
   		style参数决定下的订单是市价单还是限价单，默认是None代表市价单
   		side参数决定是开空单还是多单，默认为多单，股票只能多单，股指期货等其他品类可以开空单
   		pindex参数是在多资金仓位时选择资金仓位的，股票一般用不到
	
	order_target(security,amount)：将股票仓位调整至一定数量，如：
	order_target("000001.XSHE",1000) 表示 如果目前平安银行的持股数量低于1000股就买入，高于就是卖出，不高不低就不动
	
	order_value(security,value)，含义是买卖一定价值量（单位：元）股票 ，如：
	order_value("000001.XSHE",10000) 表示买入10000元的平安银行，如果当前股票市价是10元，则代表买入1000股
	
	order_target_value(security,value)，通过买卖，将股票仓位调整至一定价值量（单位：元），如：
	order_target_value("000001.XSHE",10000) 调整平安银行的持股价值量至10000元 即：如果目前平安银行的持股价值量（按股票市价算）低于10000元就买入，高于就是卖出，不高不低就不动。   		
````

##### [content数据解析，非常重要](https://www.joinquant.com/view/community/detail/04a0251d77b31e782afa0f321c459d10)
![](https://image.joinquant.com/79e1891ccd9745dddaddc9a2cf18fa6a)

````
	应用：计算收益率、止损
	for stk in g.security:
	  order(stk,100)
	  # 获得股票持仓成本
	  cost=context.portfolio.positions[stk].avg_cost
	  # 获得股票现价
	  price=context.portfolio.positions[stk].price
	  # 计算收益率
	  ret=price/cost-1
	  # 如果收益率小于-0.01，即亏损达到1%则卖出股票，幅度可以自己调，一般10%
	  if ret<-0.01:
	      order_target(stk,0)
	      print('触发止损')
````

##### [循环、多股票策略](https://www.joinquant.com/view/community/detail/1d3520aa01c254eabb8220bce032e3dc)

##### [获取典型常用数据](https://www.joinquant.com/view/community/detail/c688e86342b472f380c8fb9fc58eec54) [数据](https://www.joinquant.com/data)
	
* [获取历史行情](https://www.joinquant.com/help/data/index#%E5%8E%86%E5%8F%B2%E8%A1%8C%E6%83%85%E6%95%B0%E6%8D%AE)

	

````
获取沪深300指数信息（没实际意义）：
obj = get_security_info('000300.XSHG')
log.info(obj.display_name)
log.info(obj.start_date)

获取所有指数：
log.info(get_all_securities(['index']))

获取指数中所有个股:
indexs = get_index_stocks('000300.XSHG')# 获取所有沪深300的股票,只返回的代码
set_universe(indexs) #设置为股票池

获取历史行情数据：
history(count, unit, field, security_list=['index_list'], df=True, skip_paused=False, fq='pre')
参数说明：
	count: 数量, 返回的结果集的行数 
	unit: 单位时间长度, 几天或者几分钟, 现在支持’Xd’,’Xm’, X是一个正整数, 分别表示X天和X分钟
	field: 要获取的数据类型, 支持属性里面的所有基本属性.
	security_list: 要获取数据的指数列表
	df: 若是True, 返回[pandas.DataFrame],默认是True.
	skip_paused: 是否跳过不交易日期
	fq: 复权选项: pre前复权  None不复权, 返回实际价格  post后复权
	
````

##### 获取单只个股历史数据，[具体参考](https://www.joinquant.com/view/community/detail/c688e86342b472f380c8fb9fc58eec54)
![](https://image.joinquant.com/c53d18cafeca95a6c146e712a4864cef)

##### [获取历史数据](https://joinquant.com/help/api/help?name=api_old#get_bars-%E8%8E%B7%E5%8F%96%E5%8E%86%E5%8F%B2%E6%95%B0%E6%8D%AE)，可以同时获取多只个股且可以获取分钟、日线、周线、月线
![](img/get_bars.png)

'''
data = get_bars(security='000001.XSHG', count=100, unit='1w',
                include_now=False, 
                end_dt='2019-07-10', fq_ref_date=None)
'''

##### [获取财务数据](https://www.joinquant.com/help/api/help?name=api_old#get_fundamentals-%E6%9F%A5%E8%AF%A2%E8%B4%A2%E5%8A%A1%E6%95%B0%E6%8D%AE)  [涉及到的财务字段与表](https://www.joinquant.com/help/api/help?name=Stock#%E8%B4%A2%E5%8A%A1%E6%95%B0%E6%8D%AE%E5%88%97%E8%A1%A8)

````
query模板：
	query(表.字段).filter(筛选条件).order_by(排序方法).limit(数量上限)
	
例如：获取上证中流通市值最大的5只票
	
	 #获取上证指数成分股的股票代码
    stcks = get_index_stocks('000001.XSHG')
    q = query(
            # 获取 市值表.股票代码，市值表.流通市值
            valuation.code, valuation.circulating_market_cap
        ).filter(
            #上证指数中过滤
            valuation.code.in_(stocks)
        ).order_by(
            # 排序 按市值从大到小排
            valuation.circulating_market_cap.desc()
            # 数量 上限5条数据
    ).limit(5)
    w = get_fundamentals(q)
    log.info(w)
    
    如上面的valuation表示市值表
````


##### [获取行业成份股](https://www.joinquant.com/help/api/help?name=api_old#get_industry_stocks-%E8%8E%B7%E5%8F%96%E8%A1%8C%E4%B8%9A%E6%88%90%E4%BB%BD%E8%82%A1)，[行业概念](https://www.joinquant.com/data/dict/plateData)

````

获取所有农业股：
log.info(get_industry_stocks('A01', date=None))

````

