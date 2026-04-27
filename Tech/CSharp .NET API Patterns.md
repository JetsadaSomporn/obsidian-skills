---
tags: [tech, csharp, dotnet, jwt, api, patterns, aspnetcore]
created: 2026-04-11
updated: 2026-04-11
used-in: [[STS/STS - Project Overview]]
---

# C# .NET API Patterns

> Pattern ที่ copy ได้เลยเวลาสร้าง API service ใหม่
> ดึงจาก STS-ADM และ STS-NOC (verified ใช้งานจริง)

---

## 1. Program.cs — Boilerplate ครบชุด

```csharp
var builder = WebApplication.CreateBuilder(args);

// Controllers + Swagger
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new OpenApiInfo { Title = "API Name", Version = "v1" });
    // JWT support ใน Swagger UI
    c.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme {
        In = ParameterLocation.Header,
        Description = "Please enter a valid token",
        Name = "Authorization",
        Type = SecuritySchemeType.Http,
        BearerFormat = "JWT",
        Scheme = "Bearer"
    });
    c.AddSecurityRequirement(new OpenApiSecurityRequirement {{
        new OpenApiSecurityScheme {
            Reference = new OpenApiReference {
                Type = ReferenceType.SecurityScheme, Id = "Bearer"
            }
        },
        new string[] {}
    }});
});
```

---

## 2. JWT Authentication Setup

```csharp
builder.Services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
})
.AddJwtBearer(options =>
{
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuerSigningKey = true,
        IssuerSigningKey = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"])),
        ValidateIssuer = true,
        ValidIssuer = builder.Configuration["Jwt:Issuer"],
        ValidateAudience = true,
        ValidAudience = builder.Configuration["Jwt:Audience"],
        ValidateLifetime = true,
        ClockSkew = TimeSpan.Zero,                          // ไม่ยืดเวลา
        ValidAlgorithms = new[] { SecurityAlgorithms.HmacSha256 }
    };
});
```

```json
// appsettings.json
{
  "Jwt": {
    "Key": "your-secret-key-here",
    "Issuer": "your-issuer",
    "Audience": "your-audience"
  }
}
```

---

## 3. CORS Setup (STS config)

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("_myAllowSpecificOrigins", policy =>
    {
        policy.WithOrigins(
            "http://localhost:3000",
            "http://10.0.60.10",
            "https://scs.samtel.com"
        );
        policy.AllowAnyHeader();
        policy.AllowAnyMethod();
    });
});

// ใน app pipeline:
app.UseCors("_myAllowSpecificOrigins");
```

---

## 4. Repository Pattern

### Interface

```csharp
// Interfaces/IYourRepository.cs
public interface IYourRepository
{
    dynamic Search(SearchParam p);
    string GetError();
}
```

### Implementation

```csharp
// Repositories/YourRepository.cs
public class YourRepository : IYourRepository
{
    private string _conn;

    public YourRepository(IConfiguration config)
    {
        _conn = config.GetConnectionString("ConnString");
    }

    public dynamic Search(SearchParam p)
    {
        using (NpgsqlConnection conn = new NpgsqlConnection(_conn))
        {
            conn.Open();
            var tran = conn.BeginTransaction();
            // ... Dapper query
        }
    }
}
```

### Register ใน Program.cs

```csharp
builder.Services.AddScoped<IYourRepository, YourRepository>();
```

> [!tip] ใช้ `AddScoped` เสมอสำหรับ web app — 1 request = 1 instance

---

## 5. EF Core Identity (ADM only)

```csharp
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("ConnString"))
           .EnableSensitiveDataLogging()
           .LogTo(Console.WriteLine, LogLevel.Information));

builder.Services.AddIdentity<ApplicationUser, IdentityRole>(options =>
{
    options.Password.RequireDigit = true;
    options.Password.RequiredLength = 6;
    options.Password.RequireNonAlphanumeric = false;
    options.Password.RequireUppercase = false;
    options.SignIn.RequireConfirmedEmail = false;
})
.AddEntityFrameworkStores<ApplicationDbContext>()
.AddDefaultTokenProviders();
```

---

## 6. Security Headers

```csharp
// ใส่ก่อน app.Run()
app.Use(async (context, next) =>
{
    context.Response.Headers.Add("X-Xss-Protection", "1; mode=block");
    context.Response.Headers.Add("X-Frame-Options", "sameorigin");
    context.Response.Headers.Add("X-Content-Type-Options", "nosniff");
    context.Response.Headers.Add("Strict-Transport-Security",
        "max-age=31536000; includeSubDomains");
    context.Response.Headers.Add("Content-Security-Policy",
        "default-src 'self'; img-src 'self' data:; frame-src 'self'; connect-src 'self';");
    await next();
});
```

---

## 7. Pipeline Order (สำคัญมาก — ผิดลำดับ = auth ไม่ทำงาน)

```csharp
app.UseSwagger();
app.UseSwaggerUI();
// app.UseHttpsRedirection();  // ปิดตาม ART:MERCIL request
app.UseAuthentication();       // ต้องก่อน Authorization
app.UseAuthorization();
app.UseCors("_myAllowSpecificOrigins");
app.MapControllers();
app.Use(/* security headers */);
app.Run();
```

---

## 8. Static Files (Upload Profile)

```csharp
var uploadPath = builder.Configuration["PathFile:UploadProfile"];
if (!string.IsNullOrEmpty(uploadPath) && Path.IsPathRooted(uploadPath))
{
    Directory.CreateDirectory(uploadPath); // สร้างถ้ายังไม่มี
    app.UseStaticFiles(new StaticFileOptions
    {
        FileProvider = new PhysicalFileProvider(uploadPath),
        RequestPath = "/UploadFile/UserProfile"
    });
}
```

---

## Used In

- [[STS/STS - Architecture]] — ทุก service ใช้ pattern นี้
- [[STS/STS - Project Overview]]
