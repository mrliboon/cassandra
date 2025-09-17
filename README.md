Cassandra on Minikube: Setup & CRUD Operations
==============================================

1. Start Minikube Cluster
```
$ minikube start --memory=5000 --cpus=4
```

2. Deploy Cassandra
```
$ kubectl apply -f cassandra-service.yaml
$ kubectl apply -f cassandra-statefulset.yaml
$ kubectl get pods -w
```

3. Test CRUD Operations via Service
```
- Forward port to access Cassandra:
  $ kubectl port-forward svc/cassandra 9042:9042

- Connect using cqlsh:
  $ docker run --rm -it --network host cassandra:4.1 cqlsh 127.0.0.1 9042

- CQL Commands:
  -- Create a keyspace
  CREATE KEYSPACE testks WITH REPLICATION = {'class': 'SimpleStrategy', 'replication_factor': 1};

  -- Use the keyspace
  USE testks;

  -- Create a table
  CREATE TABLE users (
      id UUID PRIMARY KEY,
      name text,
      age int
  );

  -- Insert data
  INSERT INTO users (id, name, age) VALUES (uuid(), 'Alice', 30);
  INSERT INTO users (id, name, age) VALUES (uuid(), 'Bob', 25);

  -- Read data
  SELECT * FROM users;

  -- Update data
  UPDATE users SET age = 31 WHERE id = <Alice’s UUID>;

  -- Delete data
  DELETE FROM users WHERE id = <Bob’s UUID>;
```

4. Verify Cassandra Pods
```
- Start a Cassandra client pod:
  $ kubectl run cql-client --rm -it --image=cassandra:4.1 --restart=Never -- bash

- Connect to each Cassandra pod:
  <cql-client bash> $ cqlsh cassandra-0.cassandra 9042

  -- Useful commands:
     SHOW HOST;
     SELECT keyspace_name FROM system_schema.keyspaces;
     USE testks;
     SELECT table_name FROM system_schema.tables WHERE keyspace_name = 'testks';
     SELECT * FROM users;

  <cql-client bash> $ exit

  Repeat for other pods:
  <cql-client bash> $ cqlsh cassandra-1.cassandra 9042
```