# Actividad_Events
Actividad
Haciendo uso de las siguientes tablas para la base de datos de pizza realice los siguientes ejercicios de Events centrados en el uso de ON COMPLETION PRESERVE y ON COMPLETION NOT PRESERVE :

CREATE TABLE IF NOT EXISTS resumen_ventas (
fecha       DATE      PRIMARY KEY,
total_pedidos INT,
total_ingresos DECIMAL(12,2),
creado_en DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS alerta_stock (
  id              INT AUTO_INCREMENT PRIMARY KEY,
  ingrediente_id  INT UNSIGNED NOT NULL,
  stock_actual    INT NOT NULL,
  fecha_alerta    DATETIME NOT NULL,
  creado_en DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (ingrediente_id) REFERENCES ingrediente(id)
);
Resumen Diario Único : crear un evento que genere un resumen de ventas una sola vez al finalizar el día de ayer y luego se elimine automáticamente llamado ev_resumen_diario_unico.
Resumen Semanal Recurrente: cada lunes a las 01:00 AM, generar el total de pedidos e ingresos de la semana pasada, manteniendo el evento para que siga ejecutándose cada semana llamado ev_resumen_semanal.
Alerta de Stock Bajo Única: en un futuro arranque del sistema (requerimiento del sistema), generar una única pasada de alertas (alerta_stock) de ingredientes con stock < 5, y luego autodestruir el evento.
Monitoreo Continuo de Stock: cada 30 minutos, revisar ingredientes con stock < 10 e insertar alertas en alerta_stock, dejando el evento activo para siempre llamado ev_monitor_stock_bajo.
Limpieza de Resúmenes Antiguos: una sola vez, eliminar de resumen_ventas los registros con fecha anterior a hace 365 días y luego borrar el evento llamado ev_purgar_resumen_antiguo.




## Actividad



Haciendo uso de las siguientes tablas para la base de datos de `pizza` realice los siguientes ejercicios de `Events` centrados en el uso de **ON COMPLETION PRESERVE** y **ON COMPLETION NOT PRESERVE** :

```
CREATE TABLE IF NOT EXISTS resumen_ventas (
fecha       DATE      PRIMARY KEY,
total_pedidos INT,
total_ingresos DECIMAL(12,2),
creado_en DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS alerta_stock (
  id              INT AUTO_INCREMENT PRIMARY KEY,
  ingrediente_id  INT UNSIGNED NOT NULL,
  stock_actual    INT NOT NULL,
  fecha_alerta    DATETIME NOT NULL,
  creado_en DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (ingrediente_id) REFERENCES ingrediente(id)
);
```

1. Resumen Diario Único : crear un evento que genere un resumen de ventas **una sola vez** al finalizar el día de ayer y luego se elimine automáticamente llamado `ev_resumen_diario_unico`.
2. Resumen Semanal Recurrente: cada lunes a las 01:00 AM, generar el total de pedidos e ingresos de la semana pasada, **manteniendo** el evento para que siga ejecutándose cada semana llamado `ev_resumen_semanal`.
3. Alerta de Stock Bajo Única: en un futuro arranque del sistema (requerimiento del sistema), generar una única pasada de alertas (`alerta_stock`) de ingredientes con stock < 5, y luego autodestruir el evento.
4. Monitoreo Continuo de Stock: cada 30 minutos, revisar ingredientes con stock < 10 e insertar alertas en `alerta_stock`, **dejando** el evento activo para siempre llamado `ev_monitor_stock_bajo`.
5. Limpieza de Resúmenes Antiguos: una sola vez, eliminar de `resumen_ventas` los registros con fecha anterior a hace 365 días y luego borrar el evento llamado `ev_purgar_resumen_antiguo`.

-----------------------------------------------------------------------------------------------------------------------------------------------------------

CREATE TABLE IF NOT EXISTS ingrediente (
  id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  nombre VARCHAR(100) NOT NULL,
  stock INT NOT NULL DEFAULT 0,
  unidad VARCHAR(20) DEFAULT 'unidad',
  creado_en DATETIME DEFAULT CURRENT_TIMESTAMP
);

SOLUCIÓN 

1. Resumen Diario Único

CREATE EVENT ev_resumen_diario_unico
ON SCHEDULE AT CURRENT_DATE + INTERVAL 1 DAY
ON COMPLETION NOT PRESERVE
DO
INSERT INTO resumen_ventas (fecha, total_pedidos, total_ingresos)
SELECT 
    DATE(p.fecha_pedido) AS fecha,
    COUNT(*) AS total_pedidos,
    SUM(p.total) AS total_ingresos
FROM pedido p
WHERE DATE(p.fecha_pedido) = CURRENT_DATE - INTERVAL 1 DAY
GROUP BY DATE(p.fecha_pedido);



2. Resumen Semanal Recurrente

CREATE EVENT ev_resumen_semanal
ON SCHEDULE EVERY 1 WEEK
STARTS (TIMESTAMP(CURRENT_DATE) + INTERVAL ((9 - DAYOFWEEK(CURRENT_DATE)) % 7) DAY + INTERVAL 1 HOUR)
ON COMPLETION PRESERVE
DO
INSERT INTO resumen_ventas (fecha, total_pedidos, total_ingresos)
SELECT 
    DATE_SUB(CURRENT_DATE, INTERVAL (DAYOFWEEK(CURRENT_DATE) + 5) DAY) AS fecha,
    COUNT(*) AS total_pedidos,
    SUM(total) AS total_ingresos
FROM pedido
WHERE fecha_pedido >= DATE_SUB(CURRENT_DATE, INTERVAL (DAYOFWEEK(CURRENT_DATE) + 6) DAY)
  AND fecha_pedido < DATE_SUB(CURRENT_DATE, INTERVAL (DAYOFWEEK(CURRENT_DATE) - 1) DAY);



3. Alerta de Stock Bajo Única

   CREATE EVENT ev_alerta_stock_bajo_unica
   ON SCHEDULE AT CURRENT_TIMESTAMP + INTERVAL 10 SECOND
   ON COMPLETION NOT PRESERVE
   DO
   INSERT INTO alerta_stock (ingrediente_id, stock_actual, fecha_alerta)
   SELECT id, stock, NOW()
   FROM ingrediente
   WHERE stock < 5;

   

4.  Monitoreo Continuo de Stock

   CREATE EVENT ev_monitor_stock_bajo
   ON SCHEDULE EVERY 30 MINUTE
   STARTS CURRENT_TIMESTAMP
   ON COMPLETION PRESERVE
   DO
   INSERT INTO alerta_stock (ingrediente_id, stock_actual, fecha_alerta)
   SELECT id, stock, NOW()
   FROM ingrediente
   WHERE stock < 10;

   

5. Limpieza de Resúmenes Antiguos

   CREATE EVENT ev_purgar_resumen_antiguo
   ON SCHEDULE AT CURRENT_TIMESTAMP + INTERVAL 1 MINUTE
   ON COMPLETION NOT PRESERVE
   DO
   DELETE FROM resumen_ventas
   WHERE fecha < CURRENT_DATE - INTERVAL 365 DAY;

   -----------------------------------------------------------------------------------------------------------------------------------------------------

   SET GLOBAL event_scheduler = ON;
