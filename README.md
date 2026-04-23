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

Pasos

1 Crear tablespace UNDOTBS2 

	CREATE UNDO TABLESPACE UNDOTBS2 DATAFILE 
	
    	'+DGSYS1' SIZE 512M AUTOEXTEND ON NEXT 1M MAXSIZE UNLIMITED
		
	ONLINE
	
	RETENTION NOGUARANTEE
	
	BLOCKSIZE 16K
	
	FLASHBACK ON;

2 Cambiar el tablespace UNDO de la base de datos

	alter system set undo_tablespace=undotbs2;

3 Borra el tablespace undotbs1 

drop tablespace undotbs1 including contents and datafiles;

4 Comprobar que el tablespace UNDOTBS2 está online y con segmentos

	select tablespace_name,owner,segment_name, status from dba_rollback_segs where tablespace_name='UNDOTBS2' and status ='ONLINE';

5 Comprobar que el tablespace UNDOTBS1 está offline y sin segmentos

	select tablespace_name,owner,segment_name, status from dba_rollback_segs where tablespace_name='UNDOTBS1' and status ='OFFLINE';

6 Comprobar que NO existan sessiones en el tablespace UNDOTBS1. Si existen, eliminarlas

	select SID, substr(username,1,10) username,serial#,segment_name from v$transaction,dba_rollback_segs,v$session where saddr=ses_addr and xidusn=segment_id;
	
	alter system kill session '___,___';

7 Borrar el tablespace UNDOTBS1 

	Nota: Es muy importante el paso anterior, de lo contrario no se debe borrar el tablespace UNDOTBS1	

	drop tablespace undotbs1 including contents and datafiles;
	
8 Crear tablespace UNDOTBS1

	CREATE UNDO TABLESPACE UNDOTBS1 DATAFILE 
	
    	'+DGSYS1' SIZE 512M AUTOEXTEND ON NEXT 5M MAXSIZE UNLIMITED
		
	ONLINE
	
	RETENTION NOGUARANTEE
	
	BLOCKSIZE 16K
	
	FLASHBACK ON;

9 Cambiar el tablespace UNDO de la base de datos

	alter system set undo_tablespace=undotbs1;

10 Borrar el tablespace UNDOTBS2

	Nota: Es muy importante el paso 6, de lo contrario no se debe borrar el tablespace UNDOTBS2

	drop tablespace undotbs2 including contents and datafiles;
