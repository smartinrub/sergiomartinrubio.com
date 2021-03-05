---
name: Reparto Manager
image: https://lh3.googleusercontent.com/0PzrQ70-l6hrv7BBvi28TLGaLmqvAo0bJmG4rlJpUSZ3SzeIN-YRbqBYV9UXyfarLcsr_lr4bbGlhonYzbuyiDjAlI32ls2RlDoblXTrlyB4wQorS_lNUV9MWDWrr_dirkj5zzBufA=w600
company: Samsung Course
date:  2018-01-15
layout: post
---

## Reparto Manager

{% include elements/video.html id="CIOtugmEqR0" %}

**Reparto Manager** is a simple and light application made for delivery companies, and its main purpose is to optimize the management of orders based on locations, sale prices, and order priorities. Through an intuitive interface, Reparto Manager will allow you to generate new orders and improve logistics.

### Features

Automate delivery of products through the following operations:
- Register a new order given customer_name , Postcode, distance, price, phone and email.
- Choose the best order to deliver: The closest one will be selected; then, by the highest price; and then by date. The order can be removed, call the customer or send an email.
- Choose the best group of orders. A group of orders is composed for those who have the same cod_postal. The group price is the sum of all orders from a particular group, and the selected group will be the one with the highest price. All orders from a group can be deleted or can be selected individually to obtain individual information.
- Show a map with orders locations grouped by zip code and display a delivery route.

### Technologies

1. **Storage**: **SQLite Database**
2. **Geolocation**: **Google Maps**

#### Database
A database is used to store orders with the respective customer information, such as name, zip code, distance, price, phone and email. This information can be retrieve by SQL queries.

_SQL_ has been used, since this application can host a large amount of information, and this technology also makes it easier for us to run complex queries using techniques of nested subqueries or derived tables.

_SQL_ query to obtain the best order:

```java
db.rawQuery("SELECT _id, nombre, telefono, email FROM pedidos WHERE _id = (SELECT MIN(_id) FROM pedidos WHERE precio IN (SELECT MAX(precio) FROM pedidos WHERE distancia IN (SELECT MIN(distancia) FROM pedidos)))", null);
```
Query to retrieve best group of orders:

```java
db.rawQuery("SELECT codigo_postal, MAX(derivada.total) FROM 
(SELECT codigo_postal, SUM(precio) AS total FROM pedidos GROUP BY codigo_postal)derivada",null);

db.rawQuery("SELECT _id, nombre, telefono, email FROM pedidos WHERE codigo_postal = " + codigoPostalMejorGrupo() ,null);
```

There previous two query are the same as:
```sql
SELECT _id, nombre, codigo_postal, distancia, precio
FROM pedidos WHERE codigo_postal = (SELECT
derivada2.codigo_postal FROM (SELECT codigo_postal, MAX(derivada.total) FROM (SELECT codigo_postal, SUM(precio) AS total FROM pedidos GROUP BY codigo_postal)derivada)derivada2)
```

However, Android is not able to process it.

On the other hand, we will use _SQL_ queries to delete orders or find locations that will be placed on **Google Maps**.

#### Geolocation

**Google Maps API** for **Android** allows us to place markers to locate orders, showing the best route by postal code on the Google Maps view.

Moreover, when clicking on one of the personalized markers, a message will pop up and it will show same information like postal code and number of orders for a particular postal code.

>Postal codes are retrieved in advance by _SQL_ queries.

### Functional Description

Reparto Manager is composed by 6 different views, including the Splash screen which runs during start up. Views:

1.	**Splash** (Splash)
2.	**Main** (Main)
3.	**Create Order** (CrearPedido)
4.	**Best Order Information** (InfoMejorPedido)
5.	**Best Group of Orders Information** (InfoMejorGrupo)
6.	**Map** (Mapa)
