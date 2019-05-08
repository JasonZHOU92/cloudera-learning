
# Hive case function
hive> describe function case;
OK
CASE a WHEN b THEN c [WHEN d THEN e]* [ELSE f] END - When a = b, returns c; when a = d, return e; else return f
Time taken: 0.049 seconds, Fetched: 1 row(s)

CASE a WHEN b THEN c [WHEN d THEN e]* [ELSE f] END - When a = b, returns c; when a = d, returns e; else returns f.

CASE WHEN a THEN b [WHEN c THEN d]* [ELSE e] END - When a = true, returns b; when c = true, returns d; else returns e.


select order_status,
  case order_status
    when 'CLOSED' then 'No Action'
    when 'COMPLETE' then 'No Action'
    when 'ON_HOLD' then 'Pending_Action'
    when 'PAYMENT_REVIEW' then  'Pending_Action'
    when 'PENDING' then 'Pending_Action'
    when 'PROCESSING' then 'Pending_Action'
    when 'PENDING_PAYMENT' then 'Pending_Action'
    else 'Risky';

select order_status,
  case  
    when order_status in ('CLOSED', 'COMPLETE') then 'No Action'
    when order_status in ('ON_HOLD', 'PAYMENT_REVIEW', 'PENDING', 'PROCESSING', 'PENDING_PAYMENT') then 'Pending_Action'
    else 'Risky';


select nvl(order_status, 'Status_Missing') from orders limit 100;
select case order_status when order_status is null then 'Status_Missing' else order_status;
