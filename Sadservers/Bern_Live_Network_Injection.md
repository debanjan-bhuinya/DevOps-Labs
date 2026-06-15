# Scenario: "Bern": Docker web container can't connect to db container

**Level:** Hard  
**Tags:** `docker`, `php`, `networking`, `dns`, `runtime-injection`  

## 📝 Problem Description
An isolated WordPress frontend container and a MariaDB database backend container are deployed as standalone units. The application fails with an `Error establishing a database connection` because the default Docker bridge network does not support automatic name resolution. The application code expects the database to be reachable at the hostname `mysql`.

## 💡 The Solution Strategy
Instead of destroying the running containers or refactoring the deployment with complex Compose engines, Docker allows administrators to dynamically re-route production traffic on the fly. By provisioning a user-defined network mesh and attaching the live containers using runtime network connection flags, we can inject a network-scoped DNS alias (`mysql`) directly into the active cluster topology.

### The Commands
```bash
# 1. Provision an isolated, user-defined bridge network space
sudo docker network create production-mesh

# 2. Attach the database container to the mesh while injecting the required 'mysql' DNS alias
sudo docker network connect --alias mysql production-mesh mariadb

# 3. Intersect the web frontend container into the same communications grid
sudo docker network connect production-mesh wordpress

# 4. Verify the live network resolution loop works flawlessly
sudo docker exec wordpress mysqladmin -h mysql -u root -ppassword ping
# Output returns: mysqld is alive
