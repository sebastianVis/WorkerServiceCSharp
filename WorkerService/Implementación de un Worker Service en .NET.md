# Implementación de un Worker Service en .NET

Este documento explica paso a paso cómo crear e implementar un **Worker Service** usando .NET. Los Worker Services son ideales para tareas en segundo plano como procesamiento periódico, sincronización con APIs externas, limpieza de datos, etc.

---

## ¿Qué es un Worker Service?

Un **Worker Service** es una aplicación de consola basada en `BackgroundService` que se ejecuta continuamente como un servicio de Windows o un daemon en Linux. Utiliza `IHostedService` para integrarse con el `HostBuilder` de ASP.NET Core.



### ¿Qué es un background service?

Un **Background Service** en .NET es una clase que corre tareas **en segundo plano**, mientras tu aplicación principal sigue funcionando. Es perfecto para cosas que deben ejecutarse **periódicamente o de forma continua**, como:

- Procesar trabajos en cola

- Enviar correos

- Revisar el estado de una base de datos o API

- Mover archivos o limpiar datos automáticamente

  

------

### ¿Cómo funciona?

Un **background service** hereda de la clase `BackgroundService`, que a su vez implementa `IHostedService`. Solo necesitas sobrescribir el método:

```csharp
protected override async Task ExecuteAsync(CancellationToken stoppingToken)
```

---

## Crear un Worker Service desde cero

### Opción A: Usando .NET CLI

```bash
dotnet new worker -n MiWorkerService
cd MiWorkerService
```

Esto crea un proyecto con la siguiente estructura:

```
MiWorkerService/
├── Program.cs
├── Worker.cs
├── MiWorkerService.csproj
```

---

### Opción B: Usando Visual Studio

1. Crear nuevo proyecto
2. Buscar: `Worker Service`
3. Asignar nombre y ubicación
4. Click en **Crear**

---

## Estructura básica del Worker

### `Worker.cs`

```csharp
public class Worker : BackgroundService
{
    // ILogger es el sistema de logging oficial de .NET. En el contexto de un Worker o cualquier clase de ASP.NET Core, ILogger<T> sirve para escribir mensajes en consola, archivos, o servicios como Seq, Elasticsearch, etc.
    
    private readonly ILogger<Worker> _logger;

    public Worker(ILogger<Worker> logger)
    {
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            _logger.LogInformation("Worker running at: {time}", DateTimeOffset.Now);
            await Task.Delay(1000, stoppingToken);
        }
    }
}
```

```csharp
var cts = new CancellationTokenSource();

Task.Run(async () =>
{
    while (!cts.Token.IsCancellationRequested)
    {
        Console.WriteLine("Trabajando...");
        await Task.Delay(1000);
    }

    Console.WriteLine("Cancelado.");
});

// Cancelar después de 5 segundos
await Task.Delay(5000);
cts.Cancel();
```



### `Program.cs`

```csharp
IHost host = Host.CreateDefaultBuilder(args) 
    
    //Es el contenedor general que inicializa, configura y administra todo el ciclo de vida de tu app, ya sea una API, un Worker, una consola, o un microservicio.

    .ConfigureServices(services =>
    {
        services.AddHostedService<Worker>();
    })
    .Build();

await host.RunAsync();
```

---

## Ejecutar el Worker

Desde la terminal:

```bash
dotnet run
```

Verás:

```
Worker running at: 2025-06-06T06:30:00
Worker running at: 2025-06-06T06:30:01
...
```

---

## Extensiones comunes

- ✅ **Inyectar servicios personalizados** (bases de datos, APIs, correos).
- ✅ **Conexión a una base de datos** con EF Core.
- ✅ **Uso de appsettings.json** para configuración dinámica.
- ✅ **Publicación como servicio** en Windows/Linux.

---

## ¿Para qué usar un Worker con una base de datos?

Para tareas **automáticas** y **recurrentes**, por ejemplo:

| Ejemplo real                        | Qué haría el Worker                                   |
| ----------------------------------- | ----------------------------------------------------- |
| Encuestas sin completar             | Buscar encuestas pendientes y marcarlas como vencidas |
| Correos o notificaciones pendientes | Revisar tabla de notificaciones y enviarlas           |
| Limpieza de logs                    | Eliminar registros antiguos cada noche                |
| Generación de reportes              | Insertar reportes generados en la DB                  |

---

## ✅ Recomendaciones

- No pongas lógica de negocio en el `Worker` directamente. Usa servicios o puertos definidos en tu arquitectura (por ejemplo, inyecta `IUnitOfWork`).
- Siempre implementa control de errores y logs.
  - Usa `CancellationToken` para detener el proceso limpiamente.

