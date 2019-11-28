# TimCo Retail Manager
Retail management system built as a part of Youtube Series by Tim Corey

## Working with .Net Framework
As i dont have __.NET framework__, only notes are updated. No implementation.

Enabled Individual Authorization at the creation of Web API project with MVC.

### Working on Authorization
To get identity of the user in controller
``` C#
  // ApiController.cs
  using Microsoft.AspNet.Identity;
  
  // Returns the resource id of the logged in user
  string userId = RequestContext.Principal.Identity.GetUserId();
```

### Adding Swagger
Package : __Swashbuckle__

By default the installation of swagger package from nuget package manager enabled basic swagger. By default SwaggerConfig.cs file is created. XML comments above the method name for documentation is **not** a suggested way. It disables the swagger default documentation. 
* Describe enums as strings : rather than using ints, it will display literal string, which is more user friendly
* Enable Swagger UI makes the UI looks good
* Can inject CSS and JS

#### UI field to set token
Didn't use the option in auto generated SwaggerConfig.cs. Instead creating a class to do the same in a better way according to Tim. Details are below.

Implement IDocumentFilter, which got Apply method passing in swaggerDocument. Below code is to add one new route "/token" to the existing swagger and put it in the "Auth" category. Command is "POST" command and data supplied should come through "application/x-www-form-urlencoded" and followed by definition of three parameters
``` C#
// AuthTokenOperation.cs : IDocumentFilter
swaggerDoc.paths.Add("/token", new PathItem
  {
    post = new Operation
    {
      tags = new List<string> { "Auth" },
      consumes = new List<string>
      {
        "application/x-www-form-urlencoded"
      },
      parameters = new List<Parameter>
      {
        new Parameter
        {
          type = "string",
          name = "grant_type",
          required = true,
          @in = "formData",
          @default = "password"
        },
        new Parameter
        {
          type = "string",
          name = "username",
          required = false,
          @in = "formData"
        },
        new Parameter
        {
          type = "string",
          name = "password",
          required = false,
          @in = "formData"
        }
      }
    }
  }

```
Add the DocumentFilter in SwaggerConfig.
```C#
c.DocumentFilter<AuthTokenOperation>();
```
[Github Reference](https://stackoverflow.com/questions/51117655/how-to-use-swagger-in-asp-net-webapi-2-0-with-token-based-authentication)

The above implementation exposes a method to get access token based on user credentials. Next part deal with sending the token with all request from the API.

For sending token with all request/operations, IOperationFilter is implemented which also got an Apply method with Operation parameter. 
```C#
// AuthorizationOperationFilter : IOperationFilter
if (operation.parameters == null) // GetAll methods
{
  operation.parameters = new List<Parameter>();
}
operation.parameters.Add(new Parameter
{
  name = "Authorization",
  @in = "header",
  description = "access token",
  required = false,
  type = "string"
});

// SwaggerConfig.cs
c.OperationFilter<AuthorizationOperationFilter>();
```
