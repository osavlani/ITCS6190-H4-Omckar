## Commands Used

```
docker --version
java -version
mvn -version
```

1. To verify installation.

```
docker compose up -d
```

2. Was provided to start the containers but didn't work for codespaces configuration, so codespaces yml file was made to be use by command:

```
docker compose -f docker-compose.codespaces.yml up -d
mvn clean package
```

3. Was used to build the project

```
docker cp target/WordCountUsingHadoop-0.0.1-SNAPSHOT.jar resourcemanager:/tmp/
docker cp shared-folder/input/data/input.txt resourcemanager:/tmp/
```

4. These commands were used to copy the java archive and dataset to the working directory.

```
docker exec -it resourcemanager bash
cd /tmp
```

5. To enter container bash, and go to tmp folder.

```
hadoop fs -mkdir -p /input/data
hadoop fs -put ./input.txt /input/data
hadoop fs -ls /input/data
```

6. To make the input directory and move the test text file to the input directory.

```
hadoop jar /tmp/WordCountUsingHadoop-0.0.1-SNAPSHOT.jar \
  com.example.controller.Controller /input/data/input.txt /output
```

7. To execute the job.

```
hadoop fs -cat /output/*
```

8. To view the output.

```
hdfs dfs -get /output /tmp/
exit
docker cp resourcemanager:/tmp/output/. shared-folder/output/
```

9. To copy the output in the resource manager, then exit bash and get that copy onto the local machine.

```
docker compose down
```

10. To stop the process.

## Input and Output

Input:

Polska gurom
Lalala lala
Suii
polska gurom
YT is that
lalala lala
suii
Whew meh is that

Output:

bash-4.2$ hadoop fs -cat /output/*
gurom   2
lala    2
that    2
Lalala  1
Whew    1
Polska  1
Suii    1
suii    1
polska  1
lalala  1
meh     1

## What I observed

The map phase was fast, it took maybe 5 or 6 seconds and the reduce phase took roughly the same. Total job time was under 30 seconds end to end. I noticed that during the build task cpu utilization was full and if there was more capability it might've taken more processing, making me think docker configuration is an I/O based job. Datanodes live were just 1, and I was wondering how it would affect if we had 3 datanodes as we were supposed to in the original compose file. Output ordering was predictable, highest frequency first and matching frequency is placed random mostly in the order they were processed.

## Problems and fixes

In step 2, container configuration, the original compose file was taking forever to build, maybe crashing at the end.

Even after removing the compose file and placing codespaces based compose file it still looks for docker-compose.yml when docker-compose.codespaces.yml exists. To fix this we mention the compose file needed to be looked up in the command.

```
docker compose -f docker-compose.codespaces.yml up -d
```

Misc: while building we get an alert that cpu is at full capacity, didn't matter and the build compiled successfully.

In step 6, making an input directory. This returns an error saying input directory not found. The error is traced back to a previous command which tells bash to change directory to tmp as soon as we enter it.

```
cd tmp
```

but the starting directory is still /opt/hadoop. This is found out by using command: pwd (present working directory).

And then we cd to tmp again and check using pwd to confirm.

(I realised I pasted both commands together and cd / tmp was executed before bash shell opened)

In step 7, while executing the process kept giving errors. Error mentioned no Datanodes running. So I exited the bash and used command:

```
docker ps
```

To check containers active and running. Saw datanodes missing and executed the build command again to cover for services inactive or missing.

The error kept repeating, so with the help of Claude it was found out there were incompatible cluster IDs saved from previous failed runs being referenced in the current ones.

Fix was to compose down the containers and start again:

```
docker compose -f docker-compose.codespaces.yml down -v
docker compose -f docker-compose.codespaces.yml up -d
```
