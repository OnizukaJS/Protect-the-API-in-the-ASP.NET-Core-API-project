
# Protect the API in the ASP.NET Core API project

Test user : alice@alice.ch
Test password: Alice!01

Test user : bob@bob.ch
Test password: Bobby!01

## Exercise Goal

The aim of this exerise is to learn how to secure an ASP.NET Core API which is used from a second Razor page application using the client crendentials flow.

## Step 1 

Add the required nuget package to the WebApi project. We will use OpenIddict in this demo, but many others exist.

```xml
<PackageReference Include="OpenIddict.Validation.AspNetCore" Version="3.1.1" />
<PackageReference Include="OpenIddict.Validation.SystemNetHttp" Version="3.1.1" />
```

## Step 2 

Configure the WebApi project Startup authentication. The configuration must exactly match the configuration in the identity provider. 

Note: using OpenIddict.Validation.AspNetCore; is required for this package

```csharp
public void ConfigureServices(IServiceCollection services)
{
	services.AddAuthentication(options =>
    {
        options.DefaultScheme = OpenIddictValidationAspNetCoreDefaults.AuthenticationScheme;
    });

        // Register the OpenIddict validation components.
        services.AddOpenIddict()
            .AddValidation(options =>
            {
                // Note: the validation handler uses OpenID Connect discovery
                // to retrieve the address of the introspection endpoint.
                options.SetIssuer("https://localhost:44318/");
                options.AddAudiences("rs_dataEventRecordsApi");

                // Configure the validation handler to use introspection and register the client
                // credentials used when communicating with the remote introspection endpoint.
                //options.UseIntrospection()
                //        .SetClientId("rs_dataEventRecordsApi")
                //        .SetClientSecret("dataEventRecordsSecret");

                // disable access token encyption for this
                options.UseAspNetCore();

                // Register the System.Net.Http integration.
                options.UseSystemNetHttp();

                // Register the ASP.NET Core host.
                options.UseAspNetCore();
            });

        services.AddAuthorization(options =>
        {
            options.AddPolicy("dataEventRecordsPolicy", policyUser =>
            {
                policyUser.RequireClaim("scope", "dataEventRecords");
            });
        });

	services.AddControllers();
}
```

## Step 3

Remove the default Microsoft mappings in the WebApi project and add the UseAuthentication after the UseRouting()

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
	IdentityModelEventSource.ShowPII = true;
	JwtSecurityTokenHandler.DefaultInboundClaimTypeMap.Clear();

	if (env.IsDevelopment())
	{
		app.UseDeveloperExceptionPage();
	}
	else
	{
		app.UseExceptionHandler("/Error");
		app.UseHsts();
	}

	app.UseStaticFiles();
	app.UseRouting();

	app.UseAuthentication();
	app.UseAuthorization();

	app.UseEndpoints(endpoints =>
	{
		endpoints.MapControllers();
	});
}
```

## Step 4

Add the [Authorize] attribute to the API controller in the WebApi project

```csharp
[Authorize("dataEventRecordsPolicy")]
[Route("api/[controller]")]
public class ValuesController : Controller
{
    [HttpGet]
    public IEnumerable<string> Get()
		
```

## Step 5

Use the secure endpoint in the Razor Page application in the Index page

```csharp
public async Task OnGetAsync()
{
	//var result = await _apiService.GetUnsecureApiDataAsync();
	Data = await _apiService.GetApiDataAsync();

	Console.WriteLine(resultSecure);
}
```

## Step 6

Run the applications in IIS Express. Start all projects in Visual Studio.

Or run from the cmd, first the identityProvider, then the WebAPI , then the Razor App

```
dotnet run
```