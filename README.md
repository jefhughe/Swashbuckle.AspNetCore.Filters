# Swashbuckle.AspNetCore.Filters
[![Build status](https://ci.appveyor.com/api/projects/status/l6cowxsqp2n9e4sl?svg=true)](https://ci.appveyor.com/project/mattfrear/swashbuckle-aspnetcore-examples)

| :mega: Rename to Swashbuckle.AspNetCore.Filters |
|--------------|
| This project was formerly called Swashbuckle.AspNetCore.Examples, but it has grown from there to become a grab-bag of various filters I have created as I have used Swashbuckle over the last 3 or so years. So I have renamed it.|

This library contains a bunch of filters for [Swashbuckle.AspNetCore](https://github.com/domaindrivendev/Swashbuckle.AspNetCore).
- Add example requests https://mattfrear.com/2016/01/25/generating-swagger-example-requests-with-swashbuckle/
- Add example responses https://mattfrear.com/2015/04/21/generating-swagger-example-responses-with-swashbuckle/
- Add comments for request and response properties
- Add security info to each operation that has an `[Authorize]` endpoint, allowing you to send OAuth2 bearer tokens via Swagger-UI https://mattfrear.com/2018/07/21/add-an-authorization-header-to-your-swagger-ui-with-swashbuckle-revisited/
- Add a file upload button https://mattfrear.com/2018/01/02/add-an-upload-button-to-your-swagger-page/
- Add any old request header to all requests
- Add any old response header to all responses
- Add an indicator to each endpoint to show if it has an `[Authorize]` header (and for which policies and roles)

## Where to get it
From NuGet.

| Version of Swashbuckle you're using | You'll want this version of this package |
|-------------------------------------|-----------------------------------------|
| Swashbuckle 1.0 - 5.5 | https://www.nuget.org/packages/Swashbuckle.Examples/ |
| Swashbuckle.AspNetCore version 1.0.0 - 2.5.0 | https://www.nuget.org/packages/Swashbuckle.AspNetCore.Examples/ |
| Swashbuckle.AspNetCore version 3.0 and above | https://www.nuget.org/packages/Swashbuckle.AspNetCore.Filters/ |

## What's included
### Request example

Populate swagger's `definitions.YourObject.example` with whatever object you like.

This is great for manually testing and demoing your API as it will prepopulate the request with some useful data, so that 
when you click the example request in order to populate the form, instead of getting an autogenerated request like this:

![autogenerated request with crappy data](https://mattfrear.files.wordpress.com/2016/01/untitled.png?w=700)

You’ll get your desired example, with useful valid data, like this:

![custom request with awesome data](https://mattfrear.files.wordpress.com/2016/01/capture2.jpg?w=700)

You can see the example output in the underlying swagger.json file, which you can get to by starting your solution and 
navigating to swagger/v1/swagger.json

![swagger.json](https://mattfrear.files.wordpress.com/2016/01/capture.jpg)

### Response example

Allows you to add custom data to the example response shown in Swagger. So instead of seeing the default boring data like so:

![response with crappy data](https://mattfrear.files.wordpress.com/2015/04/response-old.png)

You'll see some more realistic data (or whatever you want):

![response with awesome data](https://mattfrear.files.wordpress.com/2015/04/response-new.png?w=700&h=358)

### Document request or response properties

Lets you add a comment-like description to properties on your request or response, e.g.
![descriptions](https://mattfrear.files.wordpress.com/2017/09/descriptions.jpg)

### Security requirements filter

Adds security information to each operation so that you can send an Authorization header to your API. Useful for API endpoints that have JWT token
authentication. e.g.

![authorization button](https://mattfrear.files.wordpress.com/2018/07/authbutton.jpg)

![bearer token](https://mattfrear.files.wordpress.com/2018/07/authbuttonclicked.jpg)

### File upload button

Adds a button for uploading a file via IFormFile
![file upload button](https://mattfrear.files.wordpress.com/2018/01/fileupload.jpg)

### Add a request header

Adds any string to your request headers for all requests. I use this for adding a correlationId to all requests.
![request header](https://mattfrear.files.wordpress.com/2018/01/header.jpg)

### Add a response header

Allows you to specify response headers for any operation
![response headers](https://user-images.githubusercontent.com/169179/35051682-b8217740-fb9d-11e7-8bec-98d4b088dfa5.png)

### Add Authorization to Summary

If you use the `[Authorize]` attribute  to your controller or to any actions, then (Auth) is added to the action's summary,
along with any specified policies or roles.

![authorization](https://user-images.githubusercontent.com/169179/36599686-1d939bcc-18a8-11e8-8f81-d8706f1f0dc1.JPG)

## Installation
Install the [NuGet package](https://www.nuget.org/packages/Swashbuckle.AspNetCore.Filters/), then enable whichever filters 
you need when you enable SwaggerGen:

```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    // Add framework services.
    services.AddMvc();

    services.AddSwaggerGen(c =>
    {
        c.SwaggerDoc("v1", new Info { Title = "My API", Version = "v1" });
        
        // [SwaggerRequestExample] & [SwaggerResponseExample]
        // version < 3.0 does this: c.OperationFilter<ExamplesOperationFilter>(); 
        c.AddSwaggerExamples(services.BuildServiceProvider()); // version 3.0 and above

        c.OperationFilter<DescriptionOperationFilter>(); // [Description] on request or response properties
        
        c.OperationFilter<AddFileParamTypesOperationFilter>(); // Adds an Upload button to endpoints which have [AddSwaggerFileUploadButton]
        c.OperationFilter<AddHeaderOperationFilter>("correlationId", "Correlation Id for the request"); // adds any string you like to the request headers - in this case, a correlation id
        c.OperationFilter<AddResponseHeadersFilter>(); // [SwaggerResponseHeader]
        c.OperationFilter<AppendAuthorizeToSummaryOperationFilter>(); // Adds "(Auth)" to the summary so that you can see which endpoints have Authorization
	
	// add Security information to each operation for OAuth2
	c.OperationFilter<SecurityRequirementsOperationFilter>();
	
	// if you're using the SecurityRequirementsOperationFilter, you also need to tell Swashbuckle you're using OAuth2
        c.AddSecurityDefinition("oauth2", new ApiKeyScheme
        {
            Description = "Standard Authorization header using the Bearer scheme. Example: \"bearer {token}\"",
            In = "header",
            Name = "Authorization",
            Type = "apiKey"
        });
    });
}
```

## How to use
### How to use - Request examples

Decorate your controller methods with the included SwaggerRequestExample attribute:

```csharp
[SwaggerRequestExample(typeof(DeliveryOptionsSearchModel), typeof(DeliveryOptionsSearchModelExample))]
public async Task<IHttpActionResult> DeliveryOptionsForAddress(DeliveryOptionsSearchModel search)
```

Now implement it, in this case via a DeliveryOptionsSearchModelExample (which should implement IExamplesProvider), 
which will generate the example data. It should return the type you specified when you specified the `[SwaggerRequestExample]`.
	
```csharp
public class DeliveryOptionsSearchModelExample : IExamplesProvider
{
    public object GetExamples()
    {
        return new DeliveryOptionsSearchModel
        {
            Lang = "en-GB",
            Currency = "GBP",
            Address = new AddressModel
            {
                Address1 = "1 Gwalior Road",
                Locality = "London",
                Country = "GB",
                PostalCode = "SW15 1NP"
            },
            Items = new[]
            {
                new ItemModel
                {
                    ItemId = "ABCD",
                    ItemType = ItemType.Product,
                    Price = 20,
                    Quantity = 1,
                    RestrictedCountries = new[] { "US" }
                }
            }
        };
    }
```

In the Swagger document, this will populate the request's schema object [example property](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md#schemaExample).
The spec for this says:

Field Name | Type | Description
---|:---:|---
example | Any | A free-form property to include an example of an instance for this schema.

#### List Request examples
As of version 2.4, `List<T>` request examples are supported. For any `List<T>` in the request, you may define a SwaggerRequestExample for `T`. 
Your IExamplesProvider should only return a single `T` and not a `List<T>`.
Working example:

```csharp
[SwaggerRequestExample(typeof(PeopleRequest), typeof(ListPeopleRequestExample), jsonConverter: typeof(StringEnumConverter))]
public IEnumerable<PersonResponse> GetPersonList([FromBody]List<PeopleRequest> peopleRequest)
{

// and then:

public class ListPeopleRequestExample : IExamplesProvider
{
    public object GetExamples()
    {
        return new PeopleRequest { Title = Title.Mr, Age = 24, FirstName = "Dave in a list", Income = null };
    }
}

```

### How to use - Response examples

Decorate your methods with the new SwaggerResponseExample attribute:
```csharp
[SwaggerResponse(200, type: typeof(IEnumerable<Country>))]
[SwaggerResponseExample(200, typeof(CountryExamples))]
[SwaggerResponse(400, type: typeof(IEnumerable<ErrorResource>))]
public async Task<HttpResponseMessage> Get(string lang)
```

Now you’ll need to add an Examples class, which will implement IExamplesProvider to generate the example data

```csharp	
public class CountryExamples : IExamplesProvider
{
    public object GetExamples()
    {
        return new List<Country>
        {
            new Country { Code = "AA", Name = "Test Country" },
            new Country { Code = "BB", Name = "And another" }
        };
    }
}
```

In the Swagger document, this will populate the response's [example object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md#exampleObject).
The spec for this says:

Field Pattern | Type | Description
---|:---:|---
{mime type} | Any | The name of the property MUST be one of the Operation produces values (either implicit or inherited). The value SHOULD be an example of what such a response would look like.

Example response for application/json mimetype of a Pet data type:

```js
{
  "application/json": {
    "name": "Puma",
    "type": "Dog",
    "color": "Black",
    "gender": "Female",
    "breed": "Mixed"
  }
}
```

Note that this differs from the Request example in that the mime type is a required property on the response example but not so on the request example.

#### Known issues
- For requests, in the Swagger 2.0 spec there is only one schema for each request object defined across all the API endpoints. So if you are using the same request object in multiple API endpoints,
i.e. on multiple controller actions like this:

```csharp
DeliveryOptions.cs
public async Task<IHttpActionResult> DeliveryOptionsForAddress(DeliveryOptionsSearchModel search)
...

// maybe in some other controller, e.g. Search.cs
public async Task<IHttpActionResult> Search(DeliveryOptionsSearchModel search)
```

That DeliveryOptionsSearchModel object is only defined once in the entire Swagger document and it can only have one **request** example defined.

### How to use - Document request and response properties
Define the SwaggerResponse, as usual:
```csharp
[HttpPost]
[Route("api/values/person")]
[SwaggerResponse(200, type: typeof(PersonResponse), description: "Successfully found the person")]
public PersonResponse GetPerson([FromBody]PersonRequest personRequest)
{
```
Now add `System.ComponentModel.Description` attributes to your request or response object:
```csharp
public class PersonResponse
{
	public int Id { get; set; }

	[Description("The first name of the person")]
	public string FirstName { get; set; }

	public string LastName { get; set; }

	[Description("His age, in years")]
	public int Age { get; set; }
```

### How to use - Security requirements filter

First you need to already have OAuth2 configured correctly, and some of your controllers and actions locked down with the `[Authorize]` attribute.

Then you need to tell Swagger that you're using OAuth2, as shown in the Installation section above:
```csharp
    services.AddSwaggerGen(c =>
    {
        c.AddSecurityDefinition("oauth2", new ApiKeyScheme
        {
            Description = "Standard Authorization header using the Bearer scheme. Example: \"bearer {token}\"",
            In = "header",
            Name = "Authorization",
            Type = "apiKey"
        });
```
This adds a securityDefinition to the bottom of the Swagger document, which Swagger-UI renders as an "Authorize" button, which when clicked brings up the Authorize dialog box shown above.

Then, when you enable the SecurityRequirementsOperationFilter:
```csharp
	// add Security information to each operation for OAuth2
	c.OperationFilter<SecurityRequirementsOperationFilter>();
```
It adds a security property to each operation, which renders in Swagger-UI as a padlock next to the operation:
![locked down actions](https://mattfrear.files.wordpress.com/2018/07/securityonaction.jpg)

By default, the SecurityRequirementsOperationFilter also adds 401 and 403 to each operation that has `[Authorize]` on it:
![401 and 403](https://mattfrear.files.wordpress.com/2018/07/401-403.jpg)

If you don't want to do that you can pass false when you configure it:

```csharp
	c.OperationFilter<SecurityRequirementsOperationFilter>(false);
```

### How to use - File upload button

Add the `[AddSwaggerFileUploadButton]` attribute to any controller actions which takes an IFormFile, e.g.
```csharp
[AddSwaggerFileUploadButton]
[HttpPost("upload")]
public IActionResult UploadFile(IFormFile file)
{
```

### How to use - Request Header
When you enable the filter in your `Startup.cs`, as per the Installation section above, you can specify the name and description of the new header parameter.
This will add the input box to *every* controller action.

### How to use - Response headers

Specify one or more `[SwaggerResponseHeader]` attributes on your controller action, like so:
```csharp
[SwaggerResponseHeader(HttpStatusCode.OK, "Location", "string", "Location of the newly created resource")]
[SwaggerResponseHeader(HttpStatusCode.OK, "ETag", "string", "An ETag of the resource")]
public IHttpActionResult GetPerson(PersonRequest personRequest)
{
```
### How to use - Authorization summary
Specify `[Authorization]` headers on either a Controller:
```csharp
[Authorize]
public class ValuesController : Controller
```
or on an action:
```csharp
[Authorize("Customer")]
public PersonResponse GetPerson([FromBody]PersonRequest personRequest)
```
You can optionally specify policies `[Authorize("Customer")]` or roles `[Authorize(Roles = "Customer")]` and they will be added to the Summary too.

## Pascal case or Camel case?
The default is camelCase. If you want PascalCase you can pass in a `DefaultContractResolver` like so:
`[SwaggerResponseExample(200, typeof(PersonResponseExample), typeof(DefaultContractResolver))]`

## Render Enums as strings
By default `enum`s will output their integer values. If you want to output strings you can pass in a `StringEnumConverter` like so:
`[SwaggerResponseExample(200, typeof(PersonResponseExample), jsonConverter: typeof(StringEnumConverter))]`

## Advanced: Examples with Dependency injection

If for some reason you need to have examples with DI (for example, read them from a database) you may:

Provide required dependencies in example provider classes:

```csharp
internal class PersonRequestExample : IExamplesProvider
{
    private readonly IHostingEnvironment _env;

    public PersonRequestExample(IHostingEnvironment env)
    {
        _env = env;
    }
    public object GetExamples()
    {
        return new PersonRequest { Age = 24, FirstName = _env.IsDevelopment() ? "Development" : "Production", Income = null };
    }
}
```

Register your example provider as a service in Startup:

```csharp
services.AddTransient<PersonRequestExample>();
```

Pass service collection to ExamplesOperationFilter as a parameter:

```csharp
services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new Info { Title = "My API", Version = "v1" });
    c.OperationFilter<ExamplesOperationFilter>(services.BuildServiceProvider());
});
```
