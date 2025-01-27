# Question 1. Understanding docker first run

**Run docker with the python:3.12.8 image in an interactive mode, use the entrypoint bash.**

**What's the version of pip in the image?**

Once I had the container running with the specs from the question, I typed in the console ``pip --version`` and the pip version was ``24.3.1`` 

# Question 2. Understanding Docker networking and docker-compose

**Given the following docker-compose.yaml, what is the hostname and port that pgadmin should use to connect to the postgres database?**

```yaml
services:
  db:
    container_name: postgres
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: 'postgres'
      POSTGRES_PASSWORD: 'postgres'
      POSTGRES_DB: 'ny_taxi'
    ports:
      - '5433:5432'
    volumes:
      - vol-pgdata:/var/lib/postgresql/data

  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: "pgadmin@pgadmin.com"
      PGADMIN_DEFAULT_PASSWORD: "pgadmin"
    ports:
      - "8080:80"
    volumes:
      - vol-pgadmin_data:/var/lib/pgadmin  

volumes:
  vol-pgdata:
    name: vol-pgdata
  vol-pgadmin_data:
    name: vol-pgadmin_data
```

The host name ``postgres`` and port ``5432`` would have to specified in Pgadmin. **postgres:5432**

The reason why ``postgres`` is the host name it's because the container name was ``postgres``. The port ``5432`` had to be used instead of ``5433`` because that is the port of the host machine but since the Postgres DB and Pgadmin containers are in the same network, the port (5432) from the Postgres DB container had to be used.

# Question 3. Trip Segmentation Count

**During the period of October 1st 2019 (inclusive) and November 1st 2019 (exclusive), how many trips, respectively, happened:**

1. **Up to 1 mile**
2. **In between 1 (exclusive) and 3 miles (inclusive),**
3. **In between 3 (exclusive) and 7 miles (inclusive),**
4. **In between 7 (exclusive) and 10 miles (inclusive),**
5. **Over 10 miles**

Correct answers are:

``104,838; 199,013; 109,645; 27,688; 35,202``

These results could be found by doing this type of query
```sql
SELECT COUNT(trip_distance)
FROM green_taxi_trips
WHERE trip_distance > 1 AND trip_distance <= 3
```

# Question 4. Longest trip for each day

**Which was the pick up day with the longest trip distance? Use the pick up time for your calculations.**

**Tip: For every day, we only care about one single trip with the longest distance.**

Correct answer:
``2019-10-31``

By grouping per day and seeing which trip had the longest distance I was able to get the answer. I did this query to find the answer:

```sql
SELECT 
	lpep_pickup_datetime::DATE AS DAY_PICKUP,
	MAX(trip_distance) AS MX_DISTANCE_DAY
FROM green_taxi_trips
GROUP BY lpep_pickup_datetime::DATE
ORDER BY MX_DISTANCE_DAY DESC
```

# Question 5. Three biggest pickup zones

**Which were the top pickup locations with over 13,000 in total_amount (across all trips) for 2019-10-18?**

**Consider only lpep_pickup_datetime when filtering by date.**

Correct answer:
``East Harlem North, East Harlem South, Morningside Heights``

By grouping per pickup location and summing their total amounts i was able to see the top 3 pickup locations. I used the taxi_zone_lookup.csv as reference for the location name.
```sql
SELECT 
	"PULocationID" AS PICKUP_LOCATION,
	SUM(total_amount) AS TOTAL
FROM green_taxi_trips
WHERE lpep_pickup_datetime::DATE = '2019-10-18'
GROUP BY "PULocationID"
ORDER BY TOTAL DESC
```

# Question 6. Largest tip

**For the passengers picked up in October 2019 in the zone named "East Harlem North" which was the drop off zone that had the largest tip?**

**Note: it's tip , not trip**

**We need the name of the zone, not the ID.**

Correct answer:
``JFK Airport``

By filtering accordingly we can then group by drop off location and see which location had the biggest tip.
```sql
SELECT 
	"DOLocationID" AS DROP_OFF_LOCATION,
	MAX(tip_amount) AS BIGGEST_TIP
FROM green_taxi_trips
WHERE 
	lpep_pickup_datetime::DATE >= '2019-10-01' AND 
	lpep_pickup_datetime::DATE <= '2019-10-31' AND
	"PULocationID" = 74
GROUP BY "DOLocationID"
ORDER BY BIGGEST_TIP DESC
LIMIT 100;
```

# Question 7. Terraform Workflow

**Which of the following sequences, respectively, describes the workflow for:**

1. **Downloading the provider plugins and setting up backend,**
2. **Generating proposed changes and auto-executing the plan**
3. **Remove all resources managed by terraform**

Correct Answer: ``terraform init, terraform apply -auto-approve, terraform destroy``
