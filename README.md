With customer_growth as(
select top 10 count(distinct(ID)) as NewCustomer , cast(Created_date as date) as Date, LocationID ,BranchCode  from dbo.Customer_Registered cr 
group by cast(Created_date as date), LocationID ,BranchCode),

lost_customer as (
select top 10 count(distinct(ID)) as LostCustomer , cast(Stopdate as date) as Date, LocationID ,BranchCode  from dbo.Customer_Registered cr 
where stopdate is not null 
group by cast(stopdate as date) , LocationID ,BranchCode  ),

Growth_Lose as (
select G.Date,NewCustomer,LostCustomer, G.LocationID,G.BranchCode from customer_growth G
left join lost_customer L on  G.Date = L.Date
union 
select L.Date,NewCustomer,LostCustomer, L.LocationID,L.BranchCode from customer_growth G
right join lost_customer L on  G.Date = L.Date) 

select Date , isNull(NewCustomer,0) as TotalNewCustomers , isNull(LostCustomer,0) as TotalLostCustomer, LocationID,BranchCode from Growth_Lose
order by Date desc 





select top 10 count(distinct(ID)) as NewCustomer , LocationID , BranchCode , cast(Created_date as date) as Date 
into #Growth_Customer
from dbo.Customer_Registered cr 
group by cast(Created_date as date) , locationID , BranchCode 

select top 10 count(distinct(ID)) as LostCustomer , LocationID , BranchCode , cast(Stopdate as date) as Date 
into #Lost_Customer
from dbo.Customer_Registered cr 
where stopdate is not null 
group by cast(stopdate as date) , LocationID,BranchCode

select * into #Growth_Lose from (
select G.Date,NewCustomer,LostCustomer,G.LocationID,G.BranchCode from #Growth_Customer G
left join #Lost_Customer L on  G.Date = L.Date
union 
select L.Date,NewCustomer,LostCustomer,L.LocationID,L.BranchCode from #Growth_Customer G
right join #Lost_Customer L on  G.Date = L.Date) x

select Date , isNull(NewCustomer,0) as TotalNewCustomers , isNull(LostCustomer,0) as TotalLostCustomer,BranchFullName,LocationName 
from #Growth_Lose G
left join location L on L.LocationID  = G.LocationID and L.BranchCode  = G.BranchCode 
order by Date desc 

drop table #Growth_Customer
drop table #Lost_Customer
drop table #Growth_Lose