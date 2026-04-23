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
