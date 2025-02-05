# Questions Week 2

Complete the Quiz shown below. It’s a set of 6 multiple-choice questions to test your understanding of workflow orchestration, Kestra and ETL pipelines for data lakes and warehouses.

## 1. Within the execution for `Yellow` Taxi data for the year `2020` and month `12`: what is the uncompressed file size (i.e. the output file `yellow_tripdata_2020-12.csv` of the `extract` task)?

**Answer:** ``134.5 MB`` is the file size of the CSV. 

In order to get this value, i added an extra shell command after the file was uncompressed:

```yaml
commands:
      - wget -qO- https://github.com/DataTalksClub/nyc-tlc-data/releases/download/{{inputs.taxi}}/{{render(vars.file)}}.gz | gunzip > {{render(vars.file)}}
      - stat --format="%s" {{render(vars.file)}}
```

I ran the flow and in the logs of the ``extract`` task i saw that the file size was printed in bytes ``134481400``. When the value is divided by 6 it would give a result of 134.48 Mb


## 2. What is the rendered value of the variable `file` when the inputs `taxi` is set to `green`, `year` is set to `2020`, and `month` is set to `04` during execution?

**Answer:** `green_tripdata_2020-04.csv` would be rendered because in the "Flow" script we defined our variable file name as ``{{inputs.taxi}}_tripdata_{{inputs.year}}-{{inputs.month}}.csv``


## 3. How many rows are there for the `Yellow` Taxi data for all CSV files in the year 2020?

**Answer:** 24,648,499 rows

In order to look how many rows there were for 2020, i filtered in the filename column for filenames that had 2020 in them.

```sql
SELECT COUNT(*)
FROM public.green_tripdata 
WHERE filename LIKE '%yellow_tripdata_2020%'
```

## 4. How many rows are there for the `Green` Taxi data for all CSV files in the year 2020?

**Answer:** 1,734,051 rows

In order to look how many rows there were for 2020, i filtered in the filename column for filenames that had 2020 in them.

```sql
SELECT COUNT(*)
FROM public.green_tripdata 
WHERE filename LIKE '%green_tripdata_2020%'
```

## 5. How many rows are there for the `Yellow` Taxi data for the March 2021 CSV file?

**Answer:** 1,925,152 rows

```sql
SELECT COUNT(*) 
FROM yellow_tripdata
WHERE filename LIKE '%yellow_tripdata_2021-03%'
```

## 6. How would you configure the timezone to New York in a Schedule trigger?

**Answer:** Add a `timezone` property set to `America/New_York` in the `Schedule` trigger configuration

Just as seen in the documentation

```yaml
triggers:
  - id: daily
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "@daily"
    timezone: America/New_York
```
 