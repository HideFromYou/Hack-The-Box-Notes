# Footprinting Lab — Medium

## Status

**Status: Completed**

## Lab notes

The Medium practical lab was completed.

The exercise reinforced enumeration of NFS and SMB, inspection of exposed files, credential discovery, and checking whether discovered credentials were reused across services. Database access and SQL queries were then used to locate the target account record.

## Methodology

1. Enumerate exposed network services.
2. Investigate NFS exports and SMB shares.
3. Inspect accessible files for operational information and credentials.
4. Test discovered credentials against relevant services within the lab scope.
5. Enumerate the database and query the relevant table.

## Key takeaways

- NFS and SMB can expose files that reveal credentials or internal operational details.
- Credential reuse can connect otherwise separate services.
- Database access should be followed by structured enumeration of databases, tables, and columns.

Target-specific credentials and flags are intentionally excluded from this public repository.
