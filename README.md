# mysql_json - a MySQL UDF for parsing JSON 
-- Works on Ubuntu 12.04.5 on gcc version 4.6.3 with MySQL 5.5 

The UDF introduces one function: json_get, that parses a JSON object or an array and returns one of the properties.
By using to the UDF it is possible to write queries accessing the properties of JSON objects stored in MySQL.

### Examples:
```
SELECT json_get('{"a":1}', 'a')       => 1
SELECT json_get('{"a":1}', 'b')       => NULL (variable missing)
SELECT json_get('[1,2,3]', 2)         => 3
SELECT json_get('{"a":[2]}', 'a', 0)  => 2

# Also it manages the edge cases in this way:

SELECT json_get('{"a":{"b":2}}', 'a') => object
SELECT json_get('{"a":[1,2,3]}', 'a') => array
SELECT json_get('{a:[1,2,3}',    'a') => NULL (not a json)

# Verify if it is a valid JSON:

SELECT ISNULL(json_get('{"a":1}'));   => 0  # Valid
SELECT ISNULL(json_get('{"a":1'));    => 1  # Invalid

# Create an example table:

CREATE TABLE `message` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `data` text,
  PRIMARY KEY (`id`)
);
INSERT INTO message (id,data) VALUES(1,'{"from":"chris","title":"Awesome Article","body":"Lorem ipsum dolor sit amet, consectetur adipiscing elit."}');
INSERT INTO message (id,data) VALUES(2,'{"from":"loren","title":"Another Article","body":"Lorem ipsum dolor sit amet, consectetur adipiscing elit."}');
INSERT INTO message (id,data) VALUES(3,'{"from":"jason","title":"How to run a query","body":"Lorem ipsum dolor sit amet, consectetur adipiscing elit."}');

# Run queries on JSON values:

SELECT json_get(data,'title') FROM message WHERE id=2;
SELECT id,data FROM message WHERE json_get(data,'from')='chris';
SELECT id,data FROM message WHERE json_get(data,'title') LIKE '%Article%';
```


## Installation:
Make sure you have g++ and the MySQL client dev installed:
```
% sudo apt-get install g++ libmysqlclient-dev
```
Then compile it:
```
% git clone git@github.com:ChrisCinelli/mysql_json.git
% cd mysql_json
% git module init
% git module update
% g++ -Wall -O3 -fomit-frame-pointer-shared -fPIC -s -std=c++0x  mysql_json.cc  -o mysql_json.so
% sudo cp mysql_json.so /usr/lib/mysql/plugin
% mysql -u root
mysql> create function json_get returns string soname 'mysql_json.so';
```

### See Also
[Original Author Blog]

[original author blog]:http://blog.kazuhooku.com/2011/09/mysqljson-mysql-udf-for-parsing-json.html
