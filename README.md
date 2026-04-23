# oracle-free-space-tablespace
Release space from sysaux, temp and undotbs

Tablespace SYSAUX

Pasos

1 Crear tablespace TS_AUDITORIA	

2 Mover la tabla aud$ para el nuevo tablespace

	begin
	
	dbms_audit_mgmt.set_audit_trail_location(
	
	audit_trail_type => dbms_audit_mgmt.audit_trail_aud_std, -- this move table FGA_LOG$
	
	audit_trail_location_value => 'TS_AUDITORIA');
	
	end;
	
	/

3 Mover la tabla FGA_LOG$ para el nuevo tablespace

	begin
	
	dbms_audit_mgmt.set_audit_trail_location(
	
	audit_trail_type => dbms_audit_mgmt.audit_trail_fga_std, -- this move table FGA_LOG$
	
	audit_trail_location_value => 'TS_AUDITORIA');
	
	end;
	/
  
4 Mover la tabla AUD$UNIFIED para el nuevo tablespace

	begin
	
	dbms_audit_mgmt.set_audit_trail_location(
	
	audit_trail_type => dbms_audit_mgmt.audit_trail_unified, -- this move table AUD$UNIFIED$
	
	audit_trail_location_value => 'TS_AUDITORIA_1');
	
	end;
	/

Tablespace TEMP

Pasos

1 Crear tablespace TEMP2

	CREATE TEMPORARY TABLESPACE TEMP2 TEMPFILE 
	
  	'+DGSYS1' SIZE 100M AUTOEXTEND ON NEXT 1M MAXSIZE UNLIMITED
	
	TABLESPACE GROUP ''
	
	EXTENT MANAGEMENT LOCAL UNIFORM SIZE 1M;
	
2 Cambiar el tablespace por defecto de la base de datos

	alter database default temporary tablespace TEMP2;

	Nota:Si toma mucho tiempo borrar el tablespace es necesario verificar si existen sesiones usando este tablespace y eliminarlas
	select tu.username,s.sid,s.serial#,s.program from v$tempseg_usage tu, v$session s where tu.session_addr=s.saddr and tu.tablespace='TEMP';
	alter system kill session '___,___';

3 Borrar el tablespace TEMP

	drop tablespace TEMP including contents and datafiles;

4 Crear tablespace TEMP

	CREATE TEMPORARY TABLESPACE TEMP TEMPFILE 
	
  	'+DGSYS1' SIZE 100M AUTOEXTEND ON NEXT 1M MAXSIZE UNLIMITED
	
	TABLESPACE GROUP ''
	
	EXTENT MANAGEMENT LOCAL UNIFORM SIZE 1M;

5 Cambiar el tablespace por defecto de la base de datos

	alter database default temporary tablespace TEMP;

6 Borrar el tablespace TEMP2

	drop tablespace TEMP2 including contents and datafiles;
