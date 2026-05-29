#satellite #foreman #rhs #manual_intervention #redhat #postgres 
#### Delete a host
```sql
CREATE OR REPLACE FUNCTION delete_foreman_host(p_host_id INT) RETURNS VOID AS $$
BEGIN
    DELETE FROM template_invocation_input_values
        WHERE template_invocation_id IN (
            SELECT id FROM template_invocations WHERE host_id = p_host_id
        );
    DELETE FROM template_invocations WHERE host_id = p_host_id;
    DELETE FROM targeting_hosts WHERE host_id = p_host_id;
    DELETE FROM katello_host_installed_packages WHERE host_id = p_host_id;
    DELETE FROM host_status WHERE host_id = p_host_id;
    DELETE FROM fact_values WHERE host_id = p_host_id;
    DELETE FROM katello_host_available_module_streams WHERE host_id = p_host_id;
    DELETE FROM host_facets_reported_data_facets WHERE host_id = p_host_id;
    DELETE FROM hosts_resources WHERE host_id = p_host_id;
    DELETE FROM resource_quotas_hosts WHERE host_id = p_host_id;
    DELETE FROM nics WHERE host_id = p_host_id;
    DELETE FROM hosts WHERE id = p_host_id;
END;
$$ LANGUAGE plpgsql;
```






