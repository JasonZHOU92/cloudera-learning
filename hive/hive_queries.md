# Hive Queries

## Functions
hive> show functions;
hive> describe function length

hive> select length("Hello World");
hive> select length(order_status) from orders limit 100;

## String functions
substr
instr
like
rlike
length
lcase or lower
ucase or uppper
trim, ltrim, rtrim
lpad, rpad
split
initcap
case


select subtr("Hello World. How are you", 14);
How are you

select substr("Hello World. How are you", 7, 5);
World

select substr("Hello World. How are you", -3);
you

select substr("Hello World. How are you", -7, 3);
are

select instr("Hello World. How are you", "World");
7

select like("Hello World", "World");
false

select like("Hello World", "%World%");
true

select rlike("Hello World", "%World%");
false

select rlike("Hello World", "World");
true

select rlike("Hello World", "[a-zA-Z]{5}");
true

select upper("Hello World");
HELLO WORLD

select lower("Hello World");
hello world

select trim(" Hello World  ");
Hello World

select length(trim(" Hello World  "))
11

select length(ltrim(" Hello World  "));
13

select length(rtrim(" Hello World  "));
12

select lpad(2, 12, '0');
000000000002

select rpad(2, 12, '0');
200000000000

select cast(substr("2019-05-10", 6, 2) as int);
05

select split("Hello world, how are you", " ");


## Time Functions
current_date
current_timestamp
data_add
date_format
date_sub
datediff
day
dayofmonth
to_date
to_unix_timestamp
to_utc_timestamp
from_unix_timestamp
from_utc_timestamp
minute
month
months_between
next_day

## Aggregation
min, count, max, mean, sum
