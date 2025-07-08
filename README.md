# 🚀 **Taller de Triggers en MySQL**

## 📌 **Objetivo**

En este taller, aprenderás a utilizar **Triggers** en MySQL a través de casos prácticos. Implementarás triggers para validaciones, auditoría de cambios y registros automáticos.

------

## **🔹 Caso 1: Control de Stock de Productos**

### **Escenario:**

Una tienda en línea necesita asegurarse de que los clientes no puedan comprar más unidades de un producto del stock disponible. Si intentan hacerlo, la compra debe **bloquearse**.

### **Tarea:**

1. Crear las tablas `productos` y `ventas`.
2. Implementar un trigger `BEFORE INSERT` para evitar ventas con cantidad mayor al stock disponible.
3. Probar el trigger.

```sql
CREATE TABLE productos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(50),
    stock INT
);

CREATE TABLE ventas (
    id INT PRIMARY KEY AUTO_INCREMENT,
    id_producto INT,
    cantidad INT,
    FOREIGN KEY (id_producto) REFERENCES productos(id)
);
```

**INSERTS**

```mysql
INSERT INTO productos (nombre,stock)
VALUES ('monitor',100);
```

**SOLUCION**

```mysql
DELIMITER //

CREATE TRIGGER before_compra
BEFORE INSERT ON ventas
FOR EACH ROW
BEGIN
	DECLARE stock_actual INT;
	SELECT stock INTO stock_actual
	FROM productos
	WHERE id = NEW.id_producto;

	IF NEW.cantidad > stock_actual THEN
	SIGNAL SQLSTATE '45000'
	SET MESSAGE_TEXT = 'NO hay stock suficiente para hacer la compra';
	END IF;
END //

DELIMITER ;
```

**RESULTADO**

```
mysql> INSERT INTO ventas(id_producto,cantidad)
    -> VALUES (1,500);
ERROR 1644 (45000): NO hay stock suficiente para hacer la compra
mysql> INSERT INTO ventas(id_producto,cantidad)
    -> VALUES (1,50);
Query OK, 1 row affected (0,00 sec)
mysql> SELECT * FROM ventas;
+----+-------------+----------+
| id | id_producto | cantidad |
+----+-------------+----------+
|  1 |           1 |       50 |
+----+-------------+----------+
1 row in set (0,00 sec)
```

## *🔹 Caso 2: Registro Automático de Cambios en Salarios**

### **Escenario:**

La empresa **TechCorp** desea mantener un registro histórico de todos los cambios de salario de sus empleados.

### **Tarea:**

1. Crear las tablas `empleados` y `historial_salarios`.
2. Implementar un trigger `BEFORE UPDATE` que registre cualquier cambio en el salario.
3. Probar el trigger.

```sql
CREATE TABLE empleados (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(50),
    salario DECIMAL(10,2)
);

CREATE TABLE historial_salarios (
    id INT PRIMARY KEY AUTO_INCREMENT,
    id_empleado INT,
    salario_anterior DECIMAL(10,2),
    salario_nuevo DECIMAL(10,2),
    fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (id_empleado) REFERENCES empleados(id)
);
```

**INSERT**

```mysql
INSERT INTO empleados(nombre,salario)
VALUES ('benito camelas',1500);
```

**SOLUCION**

```mysql
DELIMITER //

CREATE TRIGGER after_salario_update
AFTER UPDATE ON empleados
FOR EACH ROW
BEGIN
	INSERT INTO historial_salarios(id_empleado,salario_anterior,salario_nuevo)
	VALUES (OLD.id,OLD.salario,NEW.salario);
END //

DELIMITER ;
```

**RESULTADO**

```
mysql> UPDATE empleados SET salario = 2000.00 WHERE id = 1;
Query OK, 1 row affected (0,01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> SELECT * FROM  historial_salarios;
+----+-------------+------------------+---------------+---------------------+
| id | id_empleado | salario_anterior | salario_nuevo | fecha               |
+----+-------------+------------------+---------------+---------------------+
|  1 |           1 |          1500.00 |       2000.00 | 2025-07-08 12:32:12 |
+----+-------------+------------------+---------------+---------------------+
1 row in set (0,00 sec)
```

## **🔹 Caso 3: Registro de Eliminaciones en Auditoría**

### **Escenario:**

La empresa **DataSecure** quiere registrar toda eliminación de clientes en una tabla de auditoría para evitar pérdidas accidentales de datos.

### **Tarea:**

1. Crear las tablas `clientes` y `clientes_auditoria`.
2. Implementar un trigger `AFTER DELETE` para registrar los clientes eliminados.
3. Probar el trigger.

```sql
CREATE TABLE clientes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(50),
    email VARCHAR(50)
);

CREATE TABLE clientes_auditoria (
    id INT PRIMARY KEY AUTO_INCREMENT,
    id_cliente INT,
    nombre VARCHAR(50),
    email VARCHAR(50),
    fecha_eliminacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**INSERT**

```mysql
INSERT INTO clientes(nombre,email)
VALUES ('yamile perdomo','yaper@gmail.com');
```

**SOLUCION**

```mysql
DELIMITER //
CREATE TRIGGER after_clientes_delete
AFTER DELETE ON clientes
FOR EACH ROW
BEGIN
	INSERT INTO clientes_auditoria(id_cliente,nombre,email)
	VALUES (OLD.id,OLD.nombre,OLD.email);
END //

DELIMITER ;
```

**RESULTADO**

```
mysql> DELETE FROM clientes WHERE id = 1;
Query OK, 1 row affected (0,00 sec)

mysql> SELECT * FROM clientes_auditoria;
+----+------------+----------------+-----------------+---------------------+
| id | id_cliente | nombre         | email           | fecha_eliminacion   |
+----+------------+----------------+-----------------+---------------------+
|  1 |          1 | yamile perdomo | yaper@gmail.com | 2025-07-08 12:39:39 |
+----+------------+----------------+-----------------+---------------------+
1 row in set (0,00 sec)
```

## **🔹 Caso 4: Restricción de Eliminación de Pedidos Pendientes**

### **Escenario:**

En un sistema de ventas, no se debe permitir eliminar pedidos que aún están **pendientes**.

### **Tarea:**

1. Crear las tablas `pedidos`.
2. Implementar un trigger `BEFORE DELETE` para evitar la eliminación de pedidos pendientes.
3. Probar el trigger.

```sql
CREATE TABLE pedidos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    cliente VARCHAR(100),
    estado ENUM('pendiente', 'completado')
);
```

**INSERT**

```mysql
INSERT INTO pedidos(cliente,estado)
VALUES ('yamile perdomo','pendiente');
```

**SOLUCION**

```mysql
DELIMITER //

CREATE TRIGGER before_pedido_delete
BEFORE DELETE ON pedidos
FOR EACH ROW
BEGIN 
	IF old.estado = 'pendiente' THEN
	SIGNAL SQLSTATE '45000'
	SET MESSAGE_TEXT = 'EL pedido sigue pendiete';
	END IF;
END //

DELIMITER ;
```

**RESULTADO**

```
mysql> INSERT INTO pedidos(cliente,estado)
    -> VALUES ('yamile perdomo','pendiente');
Query OK, 1 row affected (0,01 sec)

mysql> DELETE FROM pedidos WHERE id = 4;
ERROR 1644 (45000): EL pedido sigue pendiete

```

