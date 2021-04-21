# Azure App Service Client IP

## Obtaining the client ip in App Services

## Application Insights

- [Disable IP masking](https://docs.microsoft.com/es-mx/azure/aks/concepts-scale)
- "DisableIpMasking": true

## Manual using a Handler

### Code

```c#
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.HttpsPolicy;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;
using Microsoft.OpenApi.Models;
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace SampleApi
{
    public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {

            services.AddControllers();
            services.AddSwaggerGen(c =>
            {
                c.SwaggerDoc("v1", new OpenApiInfo { Title = "SampleApi", Version = "v1" });
            });
            services.AddApplicationInsightsTelemetry(Configuration["APPINSIGHTS_CONNECTIONSTRING"]);
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IWebHostEnvironment env, ILogger<Startup> logger)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
                app.UseSwagger();
                app.UseSwaggerUI(c => c.SwaggerEndpoint("/swagger/v1/swagger.json", "SampleApi v1"));
            }

            //app.UseHttpsRedirection();

            app.UseRouting();

            app.UseAuthorization();

            app.Use((req, next) =>
            {
                var sb = new StringBuilder();
                foreach(var header in req.Request.Headers)
                {
                    sb.AppendLine($"{header.Key}-{header.Value}");
                }
                var requestInformation = $"Path: {req.Request.Path} Method: {req.Request.Method} Client IP: {req.Request.HttpContext.Connection.RemoteIpAddress} Headers: {sb}\n";
                logger.LogInformation(requestInformation);
                return next();
            });

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllers();
            });
        }
    }
}
```

### Results

```text
Path: /weatherforecast Method: GET Client IP: ::ffff:172.16.0.1 Headers: Cache-Control-max-age=0
Connection-Keep-Alive
Accept-text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding-gzip, deflate, br
Accept-Language-en-US,en;q=0.9,es-MX;q=0.8,es;q=0.7
Host-alemorclientip.azurewebsites.net
Max-Forwards-10
User-Agent-Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.128 Safari/537.36 Edg/89.0.774.77
Upgrade-Insecure-Requests-1
X-Client-IP-76.203.170.106
X-Client-Port-60718
Sec-Fetch-Site-none
Sec-Fetch-Mode-navigate
Sec-Fetch-User-?1
Sec-Fetch-Dest-document
X-WAWS-Unencoded-URL-/weatherforecast
CLIENT-IP-76.203.170.106:60718
X-ARR-LOG-ID-bef66f2f-b1e2-4632-9531-7266984aa116
DISGUISED-HOST-alemorclientip.azurewebsites.net
X-SITE-DEPLOYMENT-ID-alemorclientip
WAS-DEFAULT-HOSTNAME-alemorclientip.azurewebsites.net
X-Original-URL-/weatherforecast
X-Forwarded-For-76.203.170.106:60718
X-ARR-SSL-2048|256|C=US, O=Microsoft Corporation, CN=Microsoft RSA TLS CA 01|CN=*.azurewebsites.net
X-Forwarded-Proto-https
X-AppService-Proto-https
X-Forwarded-TlsVersion-1.2
```
