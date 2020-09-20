# Hello-fengqi
风起在github上的第一个仓库。
//本周回流用户数=本周用户活跃数-本周新增用户数-上周活跃用户数
一旦遇到减法问题就使用left join就对了

insert into table ads_back_count
select
    "2019-12-20",
    concat(date_add(next_day("2019-12-20","MO"),-7),"_",date_add(next_day("2019-12-20","MO"),-1)),
    count(*)
from
    select t1.mid_id
        from
        (
    (select mid_id from dws_uv_detail_wk where dt=concat(date_add(next_day("2019-12-20","MO"),-7),"_",date_add(next_day("2019-12-20","MO"),-1))) t1
    left join 
    (select mid_id from dws_new_mid_day create_date <=date_add(next_day("2019-12-20","MO"),-1) and >= date_add(next_day("2019-12-20","MO"),-7)) t2
    on t1.mid_id = t2.mid_id
    left join
    (select mid_id from dws_uv_detail_wk where dt=concat(date_add(next_day("2019-12-20","MO"),-14),"_",date_add(next_day("2019-12-20","MO"),-8))) t3
    on t1.mid_id = t3.mid_id
    where t2.mid_id is null and t3.mid_id is null;
       )

//流失用户数(7天没有登录的用户数)
insert into table ads_wastage_count
select
    "2019-12-20",
    count(*)
from
    (select
        mid_id
        from
        dws_uv_detail_day
        group by mid_id
        having max(dt<=date_add("2019-12-20",-7))
    )

//最近连续3周活跃用户数
insert into table ads_continuity_wk_count
select
    "2019-12-20",
    concat(date_add(next_day("2019-12-20","MO"),-21)+"_"+date_add(next_day("2019-12-20","MO"),-1)),
    count(*)
from
    (select mid_id
        from 
        dws_uv_detail_wk           //使用周表进行分析
        where wk_dt<=
        concat(date_add(next_day('2020-12-12',"MO"),-7),"_",date_add(next_day('2020-12-12',"MO"),-1))
        and
        wk_dt>=
        concat(date_add(next_day('2020-12-12',"MO"),-7*3),"_",date_add(next_day('2020-12-12',"MO"),-7*2-1))
        group by mid_id
        having count(*)=3
    )

//最近七天连续3天活跃用户数    我们从里面开始分析
insert into table ads_continuity_uv_count
select
    '2019-12-20',
    concat(date_add('2019-12-20',-6),'_','2019-12-20'),
    count(*)
from
(
    select mid_id
    from
    (
        select mid_id      
        from
        (
            select 
                mid_id,
                date_sub(dt,rank) date_dif                //计算活跃用户的日期和排名之间的差值进行运算
            from
            (
                select 
                    mid_id,
                    dt,
                    rank() over(partition by mid_id order by dt) rank
                    from dws_uv_detail_day
                where dt>=date_add('2019-12-20',-6) and dt<='2019-12-20'    //查询最近七天的活跃用户数，然后对日期进行排名
            )t1
        )t2 
        group by mid_id,date_dif                    //对同用户进行分组，计算差值的个数
        having count(*)>=3                             //将差值大于3天的提取出来
    )t3 
    group by mid_id                                      //去重
)t4;
