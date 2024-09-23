##### Neighborhood Explorer

{#if inputs.map_input.neighborhood === true}

    # All Neighborhoods

{:else}

    # {inputs.map_input.neighborhood}

    [See neighborhood deep dive &rarr;](./{inputs.map_input.neighborhood})

{/if}


```sql neighborhoods
select neighborhood, count(*) as cases
from sf311.cases
where neighborhood is not null
group by all
order by cases desc
```


```sql filtered_trend
select date_trunc('week', opened) as date,
count(*) as cases
from sf311.cases
where opened <= '2024-08-31'
and (neighborhood = '${inputs.map_input.neighborhood}'
or '${inputs.map_input.neighborhood}' = 'true')
group by all
order by date
```



<Grid cols=2>
    <Group>

        ### Neighborhood Selector

        <AreaMap
            data={neighborhoods}
            geoJsonUrl='https://evd-geojson.b-cdn.net/sf_hoods.geojson'
            geoId=name
            areaCol=neighborhood
            value=cases
            name=map_input
        />

    </Group>
    <Group>

        ### Weekly Case Trend

        <LineChart
            data={filtered_trend}
            x=date
            y=cases
            chartAreaHeight=280
        />

    </Group>
</Grid>

### Category Breakdown

```sql category_breakdown
select
    category,
    count(*) as cases
from sf311.cases
where neighborhood = '${inputs.map_input.neighborhood}'
or '${inputs.map_input.neighborhood}' = 'true'
group by all
order by cases desc
```

<DataTable data={category_breakdown}>
    <Column id=category/>
    <Column id=cases contentType=colorscale/>
</DataTable>


## Neighborhood List

<DataTable data={neighborhoods} link=neighborhood>
    <Column id=neighborhood/>
    <Column id=cases contentType=colorscale/>
</DataTable>


