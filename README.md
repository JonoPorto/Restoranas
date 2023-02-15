# Restoranas

![alt text](https://github.com/JonoPorto/Restoranas/blob/main/scheme.png?raw=true)


Duomenų reikalavimai:

* Restorane užsakymus atlikinėja klientai. Kiekvienas klientas turi unikalų ID. Lentelėje yra saugomi klientų vardai, adresas, telefono numeris. 
Kiekvienas klientas gali atlikti daug užsakymų.

* Klientai restorane atlieka užsakymus. Kiekvienam užsakymui yra priskiriamas padavėjas ir pirkimo čekio numeris. Taip pat fiksuojama užsakymo data ir klientas, kuris atliko užsakymą.

* Pirkėjai, darydami užsakymą, gali pasirinkti vieną ar kelis patiekalus, galima užsisakyti du ar daugiau tokios pat arba skirtingos rūšies patiekalus. Saugomas patiekalo ID, pagal kurį jis gali būti identifikuojamas, ir patiekalų kiekis.

* Kiekvienas perkamas patiekalas gali būti identifikuojamas pagal jo ID. Patiekalas turi savo pavadinimą, kainą ir virėją, kuris tą patiekalą gamina.

* Patiekalus gamina skirtingi virėjai. Kiekvienas virėjas gauna skirtingą atlyginimą. Vienas virėjas gali gaminti kelis skirtingus patiekalus.

* Virėjas, tam, kad paruoštų užsakytą patiekalą, teikia užsakymus tiekėjui. Kiekvienas tiekėjas turi nurodytą ID, miestą ir pavadinimą.

* Tiekėjas tuomet teikia ingredientus, kurie taip pat turi savo atskirą pavadinimą, aprašymą. Vienas ingredientas gali būti naudojamas daugybei patiekalų.



Ar padavėjas kažkada buvo klientu?
```sql
select name, phone, count(id), 
CASE 
when count(id) = 0 then 'padavėjas nebuvo klientu' 
else 'padavėjas buvo klientu' 
end as 'ar padavėjas buvo klientu?' 
from customer 
where phone IN (select phone from waiter) 
group by id 
```
Kiek klientų užsisakė patiekalą, kurio kaina daugiau negu 50€?
```sql
select count(c.id) 
from customer c inner join orders o 
on c.id = o.customer_id inner JOIN order_items oi 
on o.invoice_nr = oi.invoice_nr inner join meal m 
on oi.meal_id = m.id 
where m.price >= 50 
```
Rasti padavėjus, kurie yra pristatę daugiau negu 50 užsakymų, kuriuos užsakė klientai, kurių vardai turi daugiau negu 5 raides. Išrikiuoti pagal ABC.
```sql
select waiter.name, count(*) from waiter 
join orders on waiter.id = orders.waiter_id 
join customer on orders.customer_id = customer.id 
group by waiter.name 
having count(*) > 50 and length(customer.name) > 12 
order by waiter.name 
```
Pateikti daugiausiai pinigų išleidusio kliento vardą, telefono numerį bei išleistą sumą.
```sql
select customer.name, customer.phone, sum(quantity * price) from customer 
join orders on customer.id = orders.customer_id  
join order_items on orders.invoice_nr = order_items.invoice_nr  
join meal on order_items.meal_id = meal.id  
group by customer.name 
having sum(quantity*price) =  
(select
max(spent)
from(
select customer.name, customer.phone, sum(quantity * price) as spent from customer  
join orders on customer.id = orders.customer_id  
join order_items on orders.invoice_nr = order_items.invoice_nr  
join meal on order_items.meal_id = meal.id  
group by customer.name  
order by sum(quantity * price) desc)customer); 
```
Kiek pinigų yra išleidęs kiekvienas klientas? Parodyti kliento vardą, telefono numerį, išleistą sumą. Išrikiuoti nuo mažiausio iki didžiausio.
