# 【原创】SQL栏目树的代码
>date: 2009-06-05T00:03:13+08:00


本文由网友whl供稿，特此感谢！  

/\*\*  

  \* Desc: 取栏目树 ,过滤用户权限和无效栏目  

  \* Author: WHL  

  \* Date: 2009-05-31 15:17  

  \*/  

  

   

/\*\* 1. 取某用户有权限（np\_cms\_column\_security表有记录且t.action\_1 = ‘1’）的栏目的树 \*\*/



```
create or replace view V_NP_CTREE_BS as
select B.* from (
select A.*, lag(A.column_id) over(partition by A.column_id order by 0 ) RK
  from (select /*+choose */
         t.*
          from np_cms_column t
         where t.is_active = '1'
        connect by prior t.column_id = t.parent_id
         start with t.column_id in (select t.column_id
                                      from np_cms_column_security t
                                     where t.subject_id = 'mazj'
                                          /*这里添加角色过滤*/
                                       and t.action_1 = '1'))A) B
 where not exists
 (select 0
          from (select distinct d.column_id
                  from np_cms_column d
                connect by prior d.column_id = d.parent_id
                 start with d.column_id in
                    (select t.column_id
                       from np_cms_column_security t
                      where t.subject_id = 'mazj'
                           /* 这里添加角色过滤*/
                        and t.action_1 = '0'
                           /* 排除有权限树下的非授权ID,既 Action_1=0的*/
                        and exists
                      (select 0
                               from (select distinct d.column_id
                                       from np_cms_column d
                                     connect by prior d.column_id =
                                                 d.parent_id
                                      start with d.column_id in
                                                 (select t.column_id
                                                    from np_cms_column_security t
                                                   where t.subject_id =
                                                         'mazj'
                                                        /*这里添加角色过滤*/
                                                     and t.action_1 = '1')) C1
                              where C1.column_id = t.column_id))
                        and d.is_active = '1') C
         where C.column_id = B.column_id and B.RK is null) and B.RK is null
union all
select c.*, 0 RK from np_cms_column c where c.parent_id = 0;

```

————————————————————————  

/\*\* 2.得到栏目的虚拟父亲ID（考虑到把断层的节点接起来）\*\*/



```
create or replace view V_NP_CTREE_PA as
select B.*,
       (case B.column_id
         when 1 then 0 else nvl(B.father, 1) end) VFA
  from (select v.*,
               (select vv.column_id
                  from V_NP_CTREE_BS vv
                 where vv.column_id = v.parent_id) FATHER
          from V_NP_CTREE_BS v) B;

```

————————————————————————  

/\*\* 3. 取出门户需要的栏目树 \*\*/



```
--create or replace view V_NP_CTREE_RS as
select
 D.*, LPAD(' ', 2 * level - 1) || SYS_CONNECT_BY_PATH(D.COLUMN_NAME, '/') &quot;Path&quot;
  from (select c.*
          from V_NP_CTREE_PA c
         order by c.VFA, c.disorder desc, c.column_id desc) D
connect by prior D.column_id = D.VFA
 start with D.column_id = 1;
 

```

————————————————————————  

（**本文版权由whl所，转载时请注明作者和出处**）


