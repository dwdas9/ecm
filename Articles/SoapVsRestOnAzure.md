# SOAP and REST - The Basics, and How REST Lives on Azure App Service
## A Practical Guide


## Part 1 - SOAP: What It Is and How It Works

### What SOAP Is

SOAP stands for **Simple Object Access Protocol**. It is a messaging protocol for exchanging structured information between systems over a network. It uses **XML** for its message format and typically runs over **HTTP or HTTPS**.

SOAP was the dominant web service standard through the 2000s. It is what I used in the original AFU project - the `Service1.asmx` that Captiva called to validate account numbers.

### The Structure of a SOAP Message

Every SOAP message is an XML document with this structure:

```xml
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
    <soap:Header>
        <!-- Optional. Authentication tokens, transaction IDs etc. -->
    </soap:Header>
    <soap:Body>
        <!-- The actual request or response content -->
        <ValidateSL210PQ xmlns="http://tempuri.org/">
            <strIn>XXXXXXXXXX|%|%AO|%|%XXXXXX</strIn>
        </ValidateSL210PQ>
    </soap:Body>
</soap:Envelope>
```

The response comes back in the same envelope structure:

```xml
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
    <soap:Body>
        <ValidateSL210PQResponse xmlns="http://tempuri.org/">
            <ValidateSL210PQResult>1001</ValidateSL210PQResult>
        </ValidateSL210PQResponse>
    </soap:Body>
</soap:Envelope>
```

### The WSDL Contract

A SOAP service publishes a **WSDL** (Web Services Description Language) document. This is the machine-readable contract - it describes every method, every parameter, every data type, and the endpoint URL.

```
http://[server]:81/IACustomWebServices/Service1.asmx?WSDL
```

Any client - regardless of platform or language - reads the WSDL and generates a proxy class from it. In .NET this was done with `wsdl.exe`:

```bat
wsdl.exe /language:vb http://[server]:81/IACustomWebServices/Service1.asmx?WSDL
```

This generated `Service1.vb` - a class that wrapped all the SOAP mechanics. The calling code saw only:

```vb
Dim ws = New Service1
Dim result = ws.SL210PQ(strIn, True)
```

### Key Characteristics of SOAP

| Characteristic | Detail |
|---|---|
| Message format | XML - always |
| Transport | Usually HTTP/HTTPS. Can also run over SMTP, TCP |
| Contract | Defined by WSDL - strongly typed |
| Error handling | Built-in `SOAPFault` structure |
| Standards | WS-Security, WS-ReliableMessaging, WS-AtomicTransaction |
| Verbosity | High - XML envelope overhead on every message |
| Interoperability | Very high - any language, any platform can read a WSDL |
| Typical use | Enterprise systems, banking, legacy integrations, B2B |


---

## Part 2 - REST: What It Is and How It Works

### What REST Is

REST stands for **Representational State Transfer**. It is an architectural style - not a protocol. It uses standard **HTTP** methods and **URLs** to expose resources and operations.

Where SOAP wraps everything in XML envelopes and routes through a single endpoint, REST uses the HTTP method and the URL together to express what is being done and to what.

### The Four HTTP Methods You Use Most

| Method | What It Does | Example |
|---|---|---|
| `GET` | Retrieve data - no side effects | `GET /api/validate/account?accountNo=XXXX` |
| `POST` | Create or submit data | `POST /api/documents` |
| `PUT` | Replace/update existing data | `PUT /api/documents/123` |
| `DELETE` | Delete data | `DELETE /api/documents/123` |

For the AFU validation service, all three endpoints were `GET` - they retrieved a validation result without changing any data.

### The Structure of a REST Request and Response

A REST request is a plain HTTP call:

```
GET /api/validate/account?accountNo=XXXXXXXXXX&transactionCode=AO&branchCode=XXXXXX&productCode=XX&holderNo=1&populate=true HTTP/1.1
Host: captivavalidationsl210.azurewebsites.net
Accept: application/json
```

The response is JSON:

```json
{
    "code": "1001",
    "outputTxt1": "CUSTOMER NAME",
    "outputTxt2": "",
    "message": ""
}
```

No envelope. No XML. Just an HTTP request and a JSON response.

### Key Characteristics of REST

| Characteristic | Detail |
|---|---|
| Message format | Usually JSON. Can be XML, plain text, binary |
| Transport | HTTP/HTTPS - always |
| Contract | No formal standard. Usually OpenAPI (Swagger) |
| Error handling | HTTP status codes - 200, 400, 404, 500 etc. |
| Standards | No WS-* stack - uses HTTP standards directly |
| Verbosity | Low - JSON is compact |
| Interoperability | Very high - any HTTP client can consume it |
| Typical use | Web APIs, mobile backends, cloud services, modern integrations |

### SOAP vs REST - Side by Side

| Aspect | SOAP | REST |
|---|---|---|
| Message format | XML - mandatory | JSON - typical, flexible |
| Contract | WSDL - formal, machine-readable | OpenAPI / Swagger - optional but standard |
| Transport | HTTP, SMTP, TCP | HTTP only |
| Client generation | `wsdl.exe` generates a typed proxy | `HttpClient` - or generate from OpenAPI |
| Overhead | High - XML envelope on every message | Low - plain HTTP |
| Error model | SOAPFault in XML | HTTP status codes |
| State | Stateless by design | Stateless by design |
| Use today | Legacy enterprise, B2B, banking | Everything new |

---

## Part 3 - How REST Lives on Azure App Service

### What Azure App Service Is

Azure App Service is a fully managed hosting platform for web applications and APIs. You deploy your code. Microsoft manages the underlying server, OS, patching, scaling, and networking.

For a REST API built in ASP.NET Core, Azure App Service is the natural home. You do not manage IIS. You do not manage Windows Server. You push code and it runs.

### The Hosting Model

```
Your Code (ASP.NET Core Web API)
        ↓
Azure App Service (Windows Plan)
        ↓
IIS running on a managed Windows Server VM
        ↓
HTTPS endpoint exposed publicly
        ↓
https://[app-name].azurewebsites.net
```

You never see or touch the VM or IIS directly. Azure manages it.

### How ASP.NET Core Web API Works

When you create an ASP.NET Core Web API project, the entry point is `Program.cs`:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register services
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure the request pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

**What each line does:**

| Line | Purpose |
|---|---|
| `AddControllers()` | Registers all controller classes in the project |
| `AddEndpointsApiExplorer()` | Enables endpoint discovery for Swagger |
| `AddSwaggerGen()` | Generates the OpenAPI specification |
| `UseSwagger()` | Serves the OpenAPI JSON at `/swagger/v1/swagger.json` |
| `UseSwaggerUI()` | Serves the Swagger test UI at `/swagger` |
| `UseHttpsRedirection()` | Redirects HTTP to HTTPS |
| `MapControllers()` | Wires URL routes to controller actions |

### How a Controller Works

A controller is a class that handles HTTP requests. Each public method in a controller is an endpoint.

```csharp
[ApiController]          // Marks this as an API controller
[Route("api/validate")]  // Base URL for all methods in this controller
public class ValidationController : ControllerBase
{
    [HttpGet("account")]  // Handles GET /api/validate/account
    public ActionResult<ValidationResponse> Account(
        [FromQuery] string accountNo,   // Read from query string
        [FromQuery] string branchCode)
    {
        // Your logic here
        return Ok(new ValidationResponse { Code = "1001" });
    }
}
```

The full URL for the above method:

```
GET https://[your-app].azurewebsites.net/api/validate/account?accountNo=XXXX&branchCode=XXXX
```

`[FromQuery]` means the parameter comes from the URL query string.
`Ok(...)` returns HTTP 200 with the object serialised as JSON.

### Configuration: appsettings.json and Azure App Service Settings

In development, configuration lives in `appsettings.json`:

```json
{
  "ConnectionStrings": {
    "CustomerDatabase": "Server=..."
  }
}
```

In Azure, you never put real credentials in `appsettings.json`. You put them in **Azure App Service → Configuration → Application settings**:

```
ConnectionStrings__CustomerDatabase = Server=tcp:...
```

The double underscore `__` is the Azure App Service way of representing nested JSON keys. `ConnectionStrings__CustomerDatabase` maps to `ConnectionStrings.CustomerDatabase` in the configuration system.

Your code reads it the same way regardless of where it lives:

```csharp
var conStr = _config.GetConnectionString("CustomerDatabase");
```

ASP.NET Core's configuration system automatically merges `appsettings.json` and Azure App Service Application Settings. Azure App Service settings override `appsettings.json` values - so local development uses the file, production uses the portal settings.

---

## Part 4 - How a .NET Application Consumes a REST API

### Using HttpClient

`HttpClient` is the standard .NET class for making HTTP calls. It is in `System.Net.Http` - no NuGet package needed.

```csharp
using System.Net.Http;
using System.Text.Json;

// Create client
var client = new HttpClient();

// Build the URL
var url = "https://captivavalidationsl210.azurewebsites.net/api/validate/account"
        + "?accountNo=XXXXXXXXXX"
        + "&transactionCode=AO"
        + "&branchCode=XXXXXX"
        + "&productCode=XX"
        + "&holderNo=1"
        + "&populate=true";

// Make the call
HttpResponseMessage response = await client.GetAsync(url);

// Read the response body as a string
string json = await response.Content.ReadAsStringAsync();

// Deserialise JSON into a typed object
var result = JsonSerializer.Deserialize<ValidationResponse>(json);

// Use the result
Console.WriteLine(result.Code);       // "1001"
Console.WriteLine(result.OutputTxt1); // "CUSTOMER NAME"
```

### The Response Model on the Client Side

The client needs the same model class to deserialise the JSON response into:

```csharp
public class ValidationResponse
{
    public string Code       { get; set; } = string.Empty;
    public string OutputTxt1 { get; set; } = string.Empty;
    public string OutputTxt2 { get; set; } = string.Empty;
    public string Message    { get; set; } = string.Empty;
}
```

This does not need to be in the same project or assembly as the API. The client defines its own copy. The JSON property names are the contract.

### HttpClient: The Right Way to Use It

Do not create a new `HttpClient` for every call. `HttpClient` is designed to be reused. In ASP.NET Core, register it as a singleton or use `IHttpClientFactory`:

```csharp
// In Program.cs
builder.Services.AddHttpClient();

// In your class
public class MyService
{
    private readonly HttpClient _client;

    public MyService(IHttpClientFactory factory)
    {
        _client = factory.CreateClient();
        _client.BaseAddress = new Uri("https://captivavalidationsl210.azurewebsites.net");
    }

    public async Task<ValidationResponse> ValidateAccount(string accountNo, ...)
    {
        var url = $"/api/validate/account?accountNo={Uri.EscapeDataString(accountNo)}&...";
        var response = await _client.GetAsync(url);
        var json     = await response.Content.ReadAsStringAsync();
        return JsonSerializer.Deserialize<ValidationResponse>(json)!;
    }
}
```

### Handling HTTP Status Codes

Always check the HTTP status code before reading the response body:

```csharp
var response = await client.GetAsync(url);

if (response.IsSuccessStatusCode)
{
    var json   = await response.Content.ReadAsStringAsync();
    var result = JsonSerializer.Deserialize<ValidationResponse>(json);
    // Use result
}
else
{
    // Handle error - log, retry, throw
    Console.WriteLine($"HTTP error: {response.StatusCode}");
}
```

Common status codes you will see:

| Code | Meaning |
|---|---|
| `200 OK` | Success - response body contains data |
| `400 Bad Request` | Your request was malformed |
| `401 Unauthorized` | Missing or invalid authentication |
| `404 Not Found` | The endpoint URL does not exist |
| `500 Internal Server Error` | Something went wrong on the server |

### URL Encoding

Always encode query string parameter values with `Uri.EscapeDataString()`. Without this, special characters in field values will break the URL:

```csharp
// Wrong - breaks if accountNo contains & or = characters
var url = $"/api/validate/account?accountNo={accountNo}";

// Correct
var url = $"/api/validate/account?accountNo={Uri.EscapeDataString(accountNo)}";
```

### Synchronous vs Asynchronous

`HttpClient` methods are asynchronous - they return `Task`. In a console app or a script context (like the Captiva Completion module), you may need to call `.Result` to block synchronously:

```csharp
// Async - preferred in ASP.NET Core
var response = await client.GetAsync(url);

// Synchronous - use when async is not available
var response = client.GetAsync(url).Result;
```

> **Note:** `.Result` blocks the calling thread. In a UI or server context this can cause deadlocks. Use `await` wherever possible. In the Captiva Completion module scripting context, `.Result` is typically acceptable.

---

## Part 5 - The Full Picture

```
CAPTIVA COMPLETION MODULE SCRIPT
(Document Type script on Captiva server)
        |
        | HttpClient.GetAsync()
        | GET /api/validate/account?accountNo=...
        | HTTPS
        ↓
AZURE APP SERVICE
(captivavalidationsl210.azurewebsites.net)
        |
        | ASP.NET Core Web API
        | ValidationController.Account()
        | Reads config from App Service Application Settings
        ↓
AZURE SQL
(CALLSL210PQ stored procedure)
        |
        | T-SQL
        | Returns: output_txt1, output_txt2, err_cd, err_txt
        ↑
        |
AZURE APP SERVICE
        |
        | Builds ValidationResponse object
        | Serialises to JSON
        | Returns HTTP 200
        ↑
        |
CAPTIVA COMPLETION MODULE SCRIPT
        |
        | Deserialises JSON
        | Reads result.Code
        | 1000 → new account, pass
        | 1001 → valid, write customer name to CL0_eICVPrOTxt, pass
        | 1002 → invalid, show message to operator, fail
        | 1003 → exception, show message to operator, fail
```

---

## Summary

Note: SOAP is not dead yet. It used to be a choice when security was important - WS-Security is still relevant in some scenarios. But for new development, REST is the clear winner. It is simpler, more efficient, and better supported on modern platforms like Azure App Service.

| Concept | SOAP | REST |
|---|---|---|
| What it is | Protocol | Architectural style |
| Message format | XML in an envelope | JSON over plain HTTP |
| Contract | WSDL - formal | OpenAPI - recommended |
| .NET client | `wsdl.exe` generates proxy | `HttpClient` |
| Azure hosting | IIS on a VM (legacy) | Azure App Service |
| Configuration | `web.config` | `appsettings.json` + App Service settings |
| When to use | Legacy enterprise, existing SOAP systems | Everything new |

---

*Dwaipayan Das*