---
id: athena-logs-queries
title: Athena and IC queries
sidebar_label: Athena and IC queries
---

## Athena queries

To get uids and number of requests per uid order by number of requests from last 14 days limit to 100, run this query:

```
select
  (substr(uid, 5)) as user_id,
  count(uid) as number_of_requests
from
  circlek.drupal_requests
where 
  file_date >= (current_date - interval '14' day) 
  and (substr(uid, 5)) <> '0' -- not superuser
  and (substr(uid, 5)) <> 'na' -- not na
group by uid
order by count(uid) desc
limit 100;
```

This is the query to Athena circlek.drupal_requests table.

## Inner Circle queries

This query returns user id, user name, roles the user is assigned to, e-mail and init. In e-mail, init and username there might be an e-mail, so it's not distributed in just one mail field but in several different fields. As for list of entities to look for - this is the result of Athena query, but becasue it's separated database you need to first get the list of uids from Athena. The easiest way to do this is to run the above query but without number_of_requests and export result to csv file. Then you can easily replace `"` with null and then replace end of line `\n` with `, `. This way you will get the list of user ids to place in where clause and order by field clause. Make sure you set `SET GLOBAL sql_mode='';` - without it you can experience some issues with queries, it's complaining about order by clause. Here is the query with only three uids, but the original was with 100 which would make the line quite long.

```
SET GLOBAL sql_mode='';

SELECT
  entity_id as user_id,
  users_field_data.name as username,
  group_concat(user__roles.roles_target_id) as roles_assigned,
  users_field_data.mail,
  users_field_data.init
FROM 
  user__roles
LEFT JOIN
  users_field_data
  ON user__roles.entity_id = users_field_data.uid
WHERE 
  entity_id IN (51961, 30426, 159341)
group by
  entity_id
order by FIELD(entity_id, 51961, 30426, 159341)
;

```

There's also another query that returs more data, like node titles provided by author, comments and so on:
```
SELECT
users_field_data.uid,
user__roles.roles_target_id as role,
users_field_data.name,
users_field_data.timezone,
users_field_data.mail,
init,
roles_target_id,
node_field_data.title as node_title,
comment_field_data.name as comment_username,
comment__field_comment.field_comment_value as comment,
users_field_data.status as status,
node_field_data.changed as node_updated,
from_unixtime(comment_field_data.changed, "%Y-%m-%d %h:%i:%s") as comment_updated
FROM
users_field_data
LEFT JOIN
user__roles
ON users_field_data.uid = user__roles.entity_id
LEFT JOIN
node_field_data
ON users_field_data.uid = node_field_data.uid
LEFT JOIN
comment_field_data
ON users_field_data.uid = comment_field_data.uid
LEFT JOIN
comment__field_comment
ON comment_field_data.cid = comment__field_comment.entity_id
where users_field_data.uid IN (51961, 30426, 159341)
;
```
