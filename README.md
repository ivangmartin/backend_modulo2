# backend_modulo2

## General

En este base de datos puedes encontrar un montón de apartementos y sus reviews, esto está sacado de hacer webscrapping.

Pregunta Si montarás un sitio real, ¿Qué posible problemas pontenciales les ves a como está almacenada la información?

Indica aquí que problemas ves


## Consultas

### Basico

Saca en una consulta cuantos apartamentos hay en España.

```javascript
db.listingsAndReviews.find({"address.country": "Spain"})
```
Lista los 10 primeros:

Sólo muestra: nombre, camas, precio, government_area
Ordenados por precio.

```javascript
db.listingsAndReviews
.find({"address.country": "Spain"},{name: 1, bedrooms: 1, price: 1, "address.government_area": 1, \_id: 0})
.sort({price: 1})
.limit(10)
```
Filtrando
Queremos viajar comodos, somos 4 personas y queremos:
4 camas.
Dos cuartos de baño.

```javascript
db.listingsAndReviews.find({beds: {$gte: 4},bathrooms: {$gte: 2}})
```
Al requisito anterior,hay que añadir que nos gusta la tecnología queremos que el apartamento tenga wifi.

```javascript
db.listingsAndReviews.find({beds: {$gte: 4},bathrooms: {$gte: 2}, amenities: "Wifi"})
```
Y bueno, un amigo se ha unido que trae un perro, así que a la query anterior tenemos que buscar que permitan mascota Pets Allowed

```javascript
db.listingsAndReviews.find({beds: {$gte: 4},bathrooms: {$gte: 2}, amenities: "Wifi",amenities: "Pets allowed"})
```
Operadores lógicos
Estamos entre ir a Barcelona o a Portugal, los dos destinos nos valen, peeero... queremos que el precio nos salga baratito (50 $), y que tenga buen rating de reviews

```javascript
db.listingsAndReviews.find(
{
$or: [
      {"address.market": "Barcelona"},
      {"address.country": "Portugal"}
    ]
  ,price: {$lte: 50},"review_scores.review_scores_rating": {$gte: 80}
}
)
```
Agregaciones
Basico
Queremos mostrar los pisos que hay en España, y los siguiente campos:
Nombre.
De que ciudad (no queremos mostrar un objeto, sólo el string con la ciudad)
El precio (no queremos mostrar un objeto, sólo el campo de precio)

```javascript
db.listingsAndReviews.aggregate(
[{$match: {
"address.country": "Spain"
}}, {$project: {
name: 1,
ciudad: "$address.market",
precio: "$price",
_id: 0
}}]
)
```
Queremos saber cuantos alojamientos hay disponibles por pais.

```javascript
db.listingsAndReviews.aggregate(
[{$group: {
_id: "$address.country",
cuantosAlojamientos: {
$sum: 1
}
}}]
)
```
Opcional
Queremos saber el precio medio de alquiler de airbnb en España.

```javascript
db.listingsAndReviews.aggregate(
[{$match: {
"address.country": "Spain"
}}, {$group: {
_id: "$address.country",
media: {
$avg: "$price"
}
}}])
```
¿Y si quisieramos hacer como el anterior, pero sacarlo por paises?

```javascript
db.listingsAndReviews.aggregate(
[{$group: {
_id: "$address.country",
media: {
$avg: "$price"
}
}}])
```
Repite los mismos pasos pero agrupando también por numero de habitaciones.

```javascript
db.listingsAndReviews.aggregate(
[{$group: {
_id: {
Pais: "$address.country",
NumeroHabitacionces: "$bedrooms",
},
media: {
$avg: "$price"
}
}}])
```
## Desafio
Queremos mostrar el top 5 de apartamentos más caros en España, y sacar los siguentes campos:

Nombre.
Ciudad.
Amenities, pero en vez de un array, un string con todos los ammenities.

```javascript
db.listingsAndReviews.aggregate([{$match: {
  "address.country": "Spain"
}}, {$sort: {
price: -1
}}, {$limit: 5}, {$project: {
nombre: "$name",
  ciudad: "$address.market",
precio: "$price",
    amenities: {
    $reduce: {
      input: "$amenities",
      initialValue: "",
      in: {
        $cond: [ { $eq: [ "$$value", ""] },
        {$concat: ["$$value","","$$this"]},
        {$concat: ["$$value",", ","$$this"]}]
      }
    }

}
}}]
)
```