---
title: "C# - JWT Configuration"
tags: [tech, csharp, dotnet, security, jwt, auth]
type: atomic-pattern
ai-context: "Setup for JWT Bearer Authentication in ASP.NET Core API"
usage-pattern: "Add to Program.cs before app.Build()"
created: 2026-04-16
---

# C# - JWT Configuration

## JSON Configuration (`appsettings.json`)
```json
{
  "Jwt": {
    "Key": "your-secret-key-here",
    "Issuer": "your-issuer",
    "Audience": "your-audience"
  }
}
```

## Service Registration (`Program.cs`)
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
        ClockSkew = TimeSpan.Zero,
        ValidAlgorithms = new[] { SecurityAlgorithms.HmacSha256 }
    };
});
```

## บทเรียนจากการใช้จริง (STS)

- `ClockSkew = TimeSpan.Zero` สำคัญมาก — ถ้าไม่ set ค่า default คือ 5 นาที ทำให้ token "หมดอายุแล้วแต่ยังใช้ได้" ซึ่ง confuse มากตอน debug
- `ValidAlgorithms` ต้อง explicit ระบุ `HmacSha256` ไม่งั้น library ยอมรับ algorithm อื่นด้วย = ช่องโหว่
- ลำดับใน `Program.cs` ต้อง `UseAuthentication()` ก่อน `UseAuthorization()` เสมอ — ถ้าสลับกันจะ authorize ไม่ผ่านโดยไม่มี error ชัดเจน

## Related
- C# - Swagger JWT Setup
- [[Tech/CSharp .NET API Patterns|C# .NET API Patterns]] (Master)
