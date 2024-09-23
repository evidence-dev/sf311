# Categories

```sql categories
select 
category,
request_type,
count(*) as cases
from sf311.cases
group by all
order by cases desc
```

<DataTable data={categories} search=true rows=25>
    <Column id=category/>
    <Column id=request_type wrap=true/>
    <Column id=cases contentType=colorscale/>
</DataTable>


