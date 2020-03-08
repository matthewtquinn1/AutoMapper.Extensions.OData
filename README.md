# AutoMapper.Extensions.OData
Creates LINQ expressions from ODataQueryOptions and executes the query.

To use, call the Get or GetAsync extension method from your OData controller.  IMapper is an AutoMapper interface.

```c#
public static ICollection<TModel> Get<TModel, TData>(this IQueryable<TData> query, IMapper mapper, ODataQueryOptions<TModel> options);
public static async Task<ICollection<TModel>> GetAsync<TModel, TData>(this IQueryable<TData> query, IMapper mapper, ODataQueryOptions<TModel> options)
```

```c#
    public class CoreBuildingController : ODataController
    {

	private readonly IMapper _mapper;
        public CoreBuildingController(MyDbContext context, IMapper mapper)
        {
            Context = context;
            _mapper = mapper;
        }

        MyDbContext Context { get; set; }

        [HttpGet]
        public async Task<IActionResult> Get(ODataQueryOptions<CoreBuilding> options)
        {
            return Ok(await Context.BuildingSet.GetAsync(_mapper, options));
        }
    }
```

<br><br>
## Do not use the EnableQuery Attribute
Using `EnableQuery` with `AutoMapper.Extensions.OData` will result in some operations being applied more than once e.g. in the [tests](https://github.com/AutoMapper/AutoMapper.Extensions.OData/blob/5b4a9c8bef4c408268603e4c2186ca65b930559c/AutoMapper.OData.EFCore.Tests/AllTests.cs#L342),
if `TMandator` has a total of two records then **without** `EnableQuery` applied to the controller action, the OData query `http://localhost:16324/opstenant?$skip=1&$top=1&$orderby=Name` will return one record as expected. However **with** `EnableQuery` applied
no records will be returned because the skip operation has been applied twice.


<br><br>
### OData query examples:

``` 
	http://localhost:<port>/opstenant?$top=5&$expand=Buildings&$filter=Name eq 'One'&$orderby=Name desc
	http://localhost:<port>/opstenant?$top=5&$expand=Buildings&$filter=Name ne 'One'&$orderby=Name desc
	http://localhost:<port>/opstenant?$filter=Name eq 'One'
	http://localhost:<port>/opstenant?$top=5&$expand=Buildings&$orderby=Name desc
	http://localhost:<port>/opstenant?$orderby=Name desc
	http://localhost:<port>/opstenant?$orderby=Name desc&$count=true
	http://localhost:<port>/opstenant?$top=5&$filter=Name eq 'One'&$orderby=Name desc&$count=true
	http://localhost:<port>/opstenant?$top=5&$select=Name, Identity
	http://localhost:<port>/opstenant?$top=5&$expand=Buildings&$filter=Name ne 'One'&$orderby=Name desc
	http://localhost:<port>/opstenant?$top=5&$expand=Buildings($expand=Builder($expand=City))&$filter=Name ne 'One'&$orderby=Name desc

	http://localhost:<port>/corebuilding?$top=5&$expand=Builder,Tenant&$filter=name ne 'One L1'&$orderby=Name desc
	http://localhost:<port>/corebuilding?$top=5&$expand=Builder($expand=City),Tenant&$filter=name ne 'One L2'&$orderby=Name desc
```
