# Conocimiento
Ejercicio # 10 extracción minera, procedimientos y vistas 

#scrip 

USE mineria;

#procedimientos 

DELIMITER //

CREATE PROCEDURE ProgramarVoladura (
    IN p_fecha DATETIME,
    IN p_zona INT,
    IN p_responsable INT,
    IN p_tipo_explosivo VARCHAR(50),
    IN p_cantidad DECIMAL(10,2),
    IN p_radio DECIMAL(10,2),
    IN p_procedimientos TEXT
)
BEGIN
    DECLARE zona_existe INT;
    DECLARE empleado_existe INT;

    SELECT COUNT(*) INTO zona_existe FROM ZONA WHERE id_zona = p_zona;
    SELECT COUNT(*) INTO empleado_existe FROM EMPLEADO WHERE id_empleado = p_responsable;

    IF zona_existe = 0 THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Zona no existe';
    ELSEIF empleado_existe = 0 THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Empleado no existe';
    ELSE
        INSERT INTO VOLADURA
        (fecha_hora, id_zona, id_responsable, tipo_explosivo, cantidad, radio_seguridad, procedimientos)
        VALUES
        (p_fecha, p_zona, p_responsable, p_tipo_explosivo, p_cantidad, p_radio, p_procedimientos);
    END IF;
END //

DELIMITER ;


DELIMITER //

CREATE PROCEDURE RegistrarExtraccionDiaria (
    IN p_fecha DATE,
    IN p_turno VARCHAR(20),
    IN p_zona INT,
    IN p_mineral VARCHAR(50),
    IN p_cantidad DECIMAL(10,2),
    IN p_ley DECIMAL(5,2),
    IN p_horas DECIMAL(5,2)
)
BEGIN
    INSERT INTO EXTRACCION
    (fecha, turno, id_zona, tipo_mineral, cantidad, ley_mineral, horas_trabajadas)
    VALUES
    (p_fecha, p_turno, p_zona, p_mineral, p_cantidad, p_ley, p_horas);
END //

DELIMITER ;


DELIMITER //

CREATE PROCEDURE AsignarPersonalTurno (
    IN p_extraccion INT,
    IN p_empleado INT
)
BEGIN
    DECLARE existe INT;

    SELECT COUNT(*) INTO existe
    FROM EMPLEADO
    WHERE id_empleado = p_empleado;

    IF existe = 0 THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Empleado no válido';
    ELSE
        INSERT INTO EXTRACCION_EMPLEADO
        VALUES (p_extraccion, p_empleado);
    END IF;
END //

DELIMITER ;


DELIMITER //

CREATE PROCEDURE ProcesarLoteMineral (
    IN p_fecha DATE,
    IN p_mineral VARCHAR(50),
    IN p_cantidad DECIMAL(10,2),
    IN p_metodo VARCHAR(50),
    IN p_concentrado DECIMAL(10,2),
    IN p_recuperacion DECIMAL(5,2),
    IN p_calidad VARCHAR(50)
)
BEGIN
    INSERT INTO PROCESAMIENTO
    (fecha, tipo_mineral, cantidad_entrada, metodo, concentrado, porcentaje_recuperacion, calidad_final)
    VALUES
    (p_fecha, p_mineral, p_cantidad, p_metodo, p_concentrado, p_recuperacion, p_calidad);
END //

DELIMITER ;


DELIMITER //

CREATE PROCEDURE RegistrarAnalisisLaboratorio (
    IN p_fecha DATETIME,
    IN p_zona INT,
    IN p_tipo VARCHAR(50),
    IN p_tecnico INT,
    IN p_validacion VARCHAR(50)
)
BEGIN
    INSERT INTO ANALISIS_LAB
    (fecha_hora, id_zona, tipo_analisis, id_tecnico, validacion)
    VALUES
    (p_fecha, p_zona, p_tipo, p_tecnico, p_validacion);
END //

DELIMITER ;

#vistas 

CREATE VIEW V_ProduccionDiaria AS
SELECT 
    e.fecha,
    z.id_zona,
    e.tipo_mineral,
    SUM(e.cantidad) AS total
FROM EXTRACCION e
JOIN ZONA z ON e.id_zona = z.id_zona
GROUP BY e.fecha, z.id_zona, e.tipo_mineral;
 
CREATE VIEW V_DisponibilidadMaquinariaPesada AS
SELECT 
    tipo,
    ubicacion,
    estado
FROM MAQUINARIA;
 
CREATE VIEW V_CalidadMineral AS
SELECT 
    z.id_zona,
    e.tipo_mineral,
    AVG(e.ley_mineral) AS promedio_ley
FROM EXTRACCION e
JOIN ZONA z ON e.id_zona = z.id_zona
GROUP BY z.id_zona, e.tipo_mineral;
 
CREATE VIEW V_VoladurasProximasSemana AS
SELECT 
    v.fecha_hora,
    z.id_zona,
    v.tipo_explosivo
FROM VOLADURA v
JOIN ZONA z ON v.id_zona = z.id_zona
WHERE v.fecha_hora BETWEEN NOW() AND DATE_ADD(NOW(), INTERVAL 7 DAY);

CREATE VIEW V_IncidentesSeguridad AS
SELECT 
    tipo,
    ubicacion,
    causas,
    COUNT(*) AS total
FROM INCIDENTE
GROUP BY tipo, ubicacion, causas;


