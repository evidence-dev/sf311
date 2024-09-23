# {params.neighborhood}

```sql summary
with case_summary as (
    select count(*) as count,
    count(*) filter (status = 'Closed') as closed
    from sf311.cases
    where neighborhood = '${params.neighborhood}'
)

select *, closed / count as close_rate
from case_summary
```

<BigValue
    data={summary}
    value=count
    fmt=num0
    title="Cases YTD"
    comparison=close_rate
    comparisonDelta=false
    comparisonFmt=pct
/>

```sql last_100
select * from sf311.cases
where neighborhood = '${params.neighborhood}'
and latitude <> 0
order by opened desc
limit 100
```

```sql top_categories
select category, count(*) as cases
from sf311.cases
    where neighborhood = '${params.neighborhood}'
group by all
order by cases desc
limit 10
```

<Grid cols=2>
    <PointMap
        title="Last 100 Cases"
        data={last_100}
        lat=latitude
        long=longitude
        startingZoom=13
        name=selected_point
        tooltipType=click
        tooltip={[
            {id: 'category', showColumnName: false, valueClass: 'font-bold text-lg'},
            {id: 'status'},
            {id: 'url', showColumnName: false, contentType: 'link', linkLabel: 'SF City Record &rarr;', valueClass: 'font-bold mt-1'}
        ]}
    />

    <BarChart
        title="Top 10 Categories"
        data={top_categories}
        x=category
        y=cases
        swapXY=true
        chartAreaHeight=200
    />
    
</Grid>



```sql spike_detection
WITH case_counts as (
    select 
        category,
        date_trunc('week', opened) as date,
        count(*) as cases
    from sf311.cases
    where neighborhood = '${params.neighborhood}'
    group by all
),

category_stats AS (
    SELECT 
        category,
        date,
        cases,
        AVG(cases) OVER (PARTITION BY category ORDER BY date ROWS BETWEEN 30 PRECEDING AND CURRENT ROW) AS rolling_avg,
        STDDEV(cases) OVER (PARTITION BY category ORDER BY date ROWS BETWEEN 30 PRECEDING AND CURRENT ROW) AS rolling_stddev
    FROM 
        case_counts
)

SELECT 
    category,
    date,
    cases,
    rolling_avg,
    rolling_stddev,
    (cases - rolling_avg) / rolling_stddev as stddev,
    CASE 
        WHEN cases > (rolling_avg + 2 * rolling_stddev) THEN 'Spike'
        ELSE 'Normal'
    END AS status
FROM 
    category_stats
```

```sql all_spikes
select *
from ${spike_detection}
where status = 'Spike'
```

# Weekly Volume Spikes in {params.neighborhood}

<Details title='What is a volume spike?'>
    A volume spike is defined as a weekly case count exceeding 2x the standard deviation of the previous 30 weeks
</Details>


```sql categories_with_spikes
select category, sum(cases) as cases 
from ${all_spikes}
group by all
order by cases desc
```

{#if categories_with_spikes.length > 0}

<Grid cols=3>

{#each categories_with_spikes as row}

    <LineChart
        title={row.category}
        data={spike_detection.where(`category = '${row.category}'`)}
        x=date
        y=cases
        handleMissing=zero
    >
        <ReferencePoint
            data={all_spikes.where(`category = '${row.category}'`)}
            x=date
            y=cases
            label=status
            color=red
            symbolSize=16
            symbolBorderWidth=1
            symbolBorderColor=red
            symbolOpacity=0.25
        />
    </LineChart>

{/each}

</Grid>

{:else}

There are no spikes above 2 standard deviations from the 30 week rolling average

{/if}


