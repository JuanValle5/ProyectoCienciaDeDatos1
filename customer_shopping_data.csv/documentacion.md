# Proyecto 1 — Introducción a la Ciencia de Datos  
## Diseño de Bodega de Datos

### Dataset utilizado
Customer Shopping Dataset  
https://www.kaggle.com/datasets/mehmettahiraslan/customer-shopping-dataset

Este dataset contiene información sobre compras realizadas por clientes en diferentes centros comerciales, incluyendo información sobre los clientes, métodos de pago, categorías de productos y valores de las compras.

---

# 1. Análisis del Dataset

El dataset contiene las siguientes columnas:


invoice_no
customer_id
gender
age
category
quantity
price
payment_method
invoice_date
shopping_mall


### Descripción de cada columna

| Columna | Descripción |
|------|------|
| invoice_no | Número de factura de la compra |
| customer_id | Identificador único del cliente |
| gender | Género del cliente |
| age | Edad del cliente |
| category | Categoría del producto comprado |
| quantity | Cantidad comprada |
| price | Precio unitario del producto |
| payment_method | Método de pago utilizado |
| invoice_date | Fecha de la compra |
| shopping_mall | Centro comercial donde se realizó la compra |

Cada fila del dataset representa **una transacción de compra realizada por un cliente**.


# 2. Problemas Identificados en el Dataset

Durante el análisis del dataset se identificaron varios problemas que impedían utilizar directamente los datos en una bodega de datos.

## 2.1 Ausencia de claves primarias adecuadas

El dataset contiene identificadores como:


invoice_no
customer_id


Sin embargo estos identificadores:

- Son claves naturales provenientes del sistema fuente
- Son de tipo texto
- No están optimizados para un modelo analítico

### Solución

Se utilizaron **claves sustitutas (Surrogate Keys)** para cada tabla del modelo dimensional.

Esto permite:

- Mejor rendimiento en consultas
- Independencia del sistema fuente
- Mejor manejo de relaciones entre tablas

---

## 2.2 La fecha no está estructurada para análisis

El dataset contiene:


invoice_date


En formato texto.

Esto dificulta realizar análisis como:

- Ventas por mes
- Ventas por año
- Comparación entre fines de semana y días laborales

### Solución

Se creó una **dimensión de fecha (`dim_date`)** que contiene:

- día
- mes
- año
- indicador de fin de semana

Esto permite realizar análisis temporales más eficientes.

---

## 2.3 No existe información de producto individual

El dataset no contiene:

- product_id
- product_name

Solo existe:


category


Esto significa que el nivel más bajo de análisis disponible es la **categoría del producto**.

### Solución

Se creó la dimensión:


dim_category


En lugar de una dimensión de productos.

---

## 2.4 La edad no es un atributo histórico

El dataset contiene:


age


El problema es que la edad cambia con el tiempo.

En un sistema ideal se almacenaría la **fecha de nacimiento** para calcular la edad dinámicamente.

### Solución

Se decidió mantener la edad como atributo descriptivo en la dimensión cliente, aclarando en el informe que lo ideal sería almacenar la fecha de nacimiento.

---

## 2.5 No existe el total de venta

El dataset contiene:


quantity
price


Pero no incluye el valor total de la compra.

### Solución

Durante el proceso ETL se calculará:


total_amount = quantity * price

Esta métrica será almacenada en la tabla de hechos.

---

# 3. Selección del Modelo de Bodega de Datos

Para el diseño de la bodega de datos se evaluaron dos modelos:

- Modelo Estrella
- Modelo Copo de Nieve

Se decidió utilizar **Modelo Estrella**.

## Justificación

El modelo estrella fue seleccionado porque:

- Es más simple de implementar
- Es más fácil de entender
- Mejora el rendimiento de las consultas analíticas
- Es ampliamente utilizado en sistemas de Business Intelligence

Además, para un proyecto académico permite representar claramente las relaciones entre hechos y dimensiones.

---

# 4. Diseño del Modelo Estrella

El modelo estrella consiste en una **tabla de hechos central** rodeada por **tablas de dimensiones**.

## Tabla de Hechos


fact_sales


Contiene las métricas del negocio.

### Métricas

- quantity
- price
- total_amount

---

## Dimensiones

Las dimensiones describen el contexto de cada venta.

- dim_customer
- dim_category
- dim_payment
- dim_mall
- dim_date

---

# 5. Estructura Final del Modelo

## fact_sales

| Campo | Descripción |
|------|------|
sale_key | Clave primaria
invoice_no | Número de factura
customer_key | FK cliente
date_key | FK fecha
category_key | FK categoría
payment_key | FK método de pago
mall_key | FK centro comercial
quantity | Cantidad comprada
price | Precio unitario
total_amount | Valor total de la compra

---

## dim_customer

| Campo | Descripción |
|------|------|
customer_key | Clave primaria
customer_id | Identificador original del cliente
gender | Género del cliente
age | Edad del cliente

---

## dim_category

| Campo | Descripción |
|------|------|
category_key | Clave primaria
category_name | Nombre de la categoría

---

## dim_payment

| Campo | Descripción |
|------|------|
payment_key | Clave primaria
payment_method | Método de pago

---

## dim_mall

| Campo | Descripción |
|------|------|
mall_key | Clave primaria
shopping_mall | Nombre del centro comercial

---

## dim_date

| Campo | Descripción |
|------|------|
date_key | Clave primaria
full_date | Fecha completa
day | Día
month | Mes
year | Año
is_weekend | Indica si es fin de semana

---

# 6. Script SQL de Creación de la Bodega de Datos

```sql
CREATE TABLE dim_customer (
    customer_key SERIAL PRIMARY KEY,
    customer_id VARCHAR(20),
    gender VARCHAR(10),
    age INT
);

CREATE TABLE dim_category (
    category_key SERIAL PRIMARY KEY,
    category_name VARCHAR(50)
);

CREATE TABLE dim_payment (
    payment_key SERIAL PRIMARY KEY,
    payment_method VARCHAR(50)
);

CREATE TABLE dim_mall (
    mall_key SERIAL PRIMARY KEY,
    shopping_mall VARCHAR(100)
);

CREATE TABLE dim_date (
    date_key SERIAL PRIMARY KEY,
    full_date DATE,
    day INT,
    month INT,
    year INT,
    is_weekend BOOLEAN
);

CREATE TABLE fact_sales (
    sale_key SERIAL PRIMARY KEY,
    invoice_no VARCHAR(20),

    customer_key INT,
    date_key INT,
    category_key INT,
    payment_key INT,
    mall_key INT,

    quantity INT,
    price DECIMAL(10,2),
    total_amount DECIMAL(12,2),

    FOREIGN KEY (customer_key) REFERENCES dim_customer(customer_key),
    FOREIGN KEY (date_key) REFERENCES dim_date(date_key),
    FOREIGN KEY (category_key) REFERENCES dim_category(category_key),
    FOREIGN KEY (payment_key) REFERENCES dim_payment(payment_key),
    FOREIGN KEY (mall_key) REFERENCES dim_mall(mall_key)
);

---

# 7. Conclusión

A partir del análisis del dataset se identificaron varias limitaciones estructurales que impedían realizar análisis analíticos eficientes.

Para resolver estos problemas se diseñó una bodega de datos basada en un **modelo estrella**, separando las métricas del negocio en una tabla de hechos y la información descriptiva en tablas de dimensiones.

Este modelo permite realizar consultas analíticas de forma eficiente como:

* Ventas por categoría
* Clientes con mayor volumen de compras
* Métodos de pago más utilizados
* Comparación de ventas por mes
* Análisis de ventas por centro comercial

Este diseño servirá como base para el proceso ETL y las consultas analíticas que se desarrollarán en las siguientes fases del proyecto.


https://lucid.app/lucidchart/23fe525d-d786-4f93-86e1-c4a9d9060803/edit?viewport_loc=675%2C1081%2C3632%2C1647%2CfwQcDkAJnwES&invitationId=inv_cc4962f6-8dc5-48a6-ab80-5c5ca90ecffa