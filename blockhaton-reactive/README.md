# Hacking the rubick

### Quickstart

- ask for a Postgres database from @vikiival ([HERE](https://github.com/kodadot/rubick/issues/70))
- setup rubick


```bash
docker exec -it rubick-db-1 psql -U postgres -d postgres
```

in the postgres shell, run the following commands:
```sql
CREATE DATABASE squid;
```

close the shell and run dump:
```bash
docker exec -i rubick-db-1 psql -U postgres -d squid < rubick.sql
```