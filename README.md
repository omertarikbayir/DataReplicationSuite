# DataReplicationSuite
> 🇹🇷 Türkçe | 🇬🇧 English

## 🇹🇷 Türkçe

**DataReplicationSuite**, çok veritabanlı (MySQL, PostgreSQL, MSSQL) ve çoklu dosya formatlı (Excel, PDF, XML) veri senkronizasyonu sağlayan, tamamen çift dilli (Türkçe + İngilizce) bir .NET 8.0 projesidir. Quartz.NET ile zamanlanmış senkronizasyon işleri, çakışma çözümü, gerçek zamanlı izleme (SignalR) ve bir WPF masaüstü istemcisi sunar.

---

## Özellikler

- **Çoklu Veritabanı Desteği** — MySQL, PostgreSQL ve Microsoft SQL Server arasında veri senkronizasyonu (Dapper + EF Core).
- **Dosya Kaynağı Desteği** — Excel (`.xlsx`, EPPlus ile) ve XML dosyalarından okuma/yazma.
- **Zamanlanmış Senkronizasyon İşleri** — **Quartz.NET 3.18.1** ile cron ifadeleriyle tekrarlayan senkronizasyon görevleri tanımlama.
- **Çakışma Tespiti & Çözümü** — Birincil anahtar bazlı satır seviyesi çakışmaları tespit etme; `SourceWins`, `TargetWins`, `LatestWins`, `Manual` çözüm stratejileri.
- **XSLT Dönüşümü** — Senkronizasyondan önce veya sonra XML verisine XSLT şablonu uygulama.
- **Gerçek Zamanlı İzleme** — **SignalR** hub'ı üzerinden canlı senkronizasyon olayları ve durum güncellemeleri.
- **WPF Masaüstü İstemcisi** — MVVM mimarisi, CommunityToolkit.Mvvm 8.3.2 ile tam Türkçe/İngilizce yerelleştirme.
- **Çift Dilli Arayüz** — Tüm arayüz metinleri Türkçe (`tr`) ve İngilizce (`en`) olarak mevcut; çalışma zamanında dil değiştirilebilir.
- **REST API** — Senkronizasyon işleri, veri kaynakları ve çakışma çözümü için CRUD ve yaşam döngüsü uç noktaları sunan ASP.NET Core Web API.

---

## Mimari

```
DataReplicationSuite/
├── DataReplicationSuite.Core              # Domain modelleri, enum'lar, arayüz sözleşmeleri (0 NuGet bağımlılığı)
├── DataReplicationSuite.Infrastructure    # EF Core DbContext, Dapper bağdaştırıcıları, hizmetler
├── DataReplicationSuite.Api               # ASP.NET Core Web API + SignalR hub
└── DataReplicationSuite.Client            # WPF MVVM masaüstü istemcisi
```

| Katman | Sorumluluk |
|---|---|
| **Core** | POCO modeller, enum'lar (`SyncDirection`, `DatabaseType`, `FileFormat`, `ConflictResolutionStrategy`, `SyncJobStatus`) ve arayüz sözleşmeleri. Hiçbir NuGet paketine bağımlı değildir. |
| **Infrastructure** | EF Core 8.0 DbContext; MySQL, PostgreSQL, MSSQL Dapper bağdaştırıcıları; EPPlus Excel işleyici; XML işleyici; `SyncEngine`, `SyncScheduler` (Quartz), `ConflictResolver`, `XsltTransformService`. |
| **Api** | REST denetleyicileri, SignalR `SyncNotificationHub` hub'ı, DTO'lar, CORS, Swagger, Quartz arka plan hizmeti kaydı. |
| **Client** | WPF `App`/`MainWindow`, ViewModels (`MainViewModel`, `SyncJobsViewModel`, `MonitorViewModel`, …), Görünümler, yerelleştirme dizeleri (TR/EN), `LocExtension` markup uzantısı. |

---

## Teknoloji Yığını

| Teknoloji | Sürüm | Amaç |
|---|---|---|
| .NET | 8.0 | Çalışma zamanı / SDK |
| ASP.NET Core | 8.0 | REST API + SignalR |
| EF Core | 8.0.26 | ORM / değişiklik izleme |
| MySql.EntityFrameworkCore | 9.0.0 | MySQL EF Core sağlayıcısı |
| Npgsql.EntityFrameworkCore.PostgreSQL | 8.0.4 | PostgreSQL EF Core sağlayıcısı |
| Microsoft.EntityFrameworkCore.SqlServer | 8.0.26 | MSSQL EF Core sağlayıcısı |
| Dapper | 2.1.66 | Mikro-ORM / ham sorgu katmanı |
| EPPlus | 8.5.4 | Excel `.xlsx` okuma/yazma |
| System.Xml | 8.0 | XML okuma/yazma + XSLT |
| Quartz.NET | 3.18.1 | Zamanlanmış iş zamanlayıcısı |
| SignalR | 1.2.10 | Gerçek zamanlı olay bildirimi |
| CommunityToolkit.Mvvm | 8.3.2 | WPF için MVVM yardımcıları |
| System.Text.Json | 9.0.0 | JSON serileştirme |

---

## Ön Koşullar

- [.NET 8.0 SDK](https://dotnet.microsoft.com/en-us/download/dotnet/8.0)
- En az bir adet desteklenen veritabanı örneği (MySQL 8+, PostgreSQL 14+, SQL Server 2019+)

---

## Hızlı Başlangıç

### 1. Klonlama

```bash
git clone https://github.com/your-org/DataReplicationSuite.git
cd DataReplicationSuite
```

### 2. Geri Yükleme & Derleme

```bash
dotnet restore
dotnet build DataReplicationSuite.sln
```

Beklenen sonuç: **Derleme başarılı. 0 Uyarı. 0 Hata.**

### 3. Veritabanı Bağlantılarını Yapılandırma

`src/DataReplicationSuite.Api/appsettings.json` dosyasını düzenleyin:

```json
{
  "ConnectionStrings": {
    "Default": "Host=localhost;Database=DataReplicationSuite;Username=postgres;Password=postgres"
  },
  "Sync": {
    "MaxBatchSize": 300,
    "MaxRetryCount": 3
  },
  "Quartz": {
    "SchedulerName": "SyncJobScheduler",
    "ThreadPoolMaxCount": 10
  }
}
```

---

## API'yi Çalıştırma

```bash
dotnet run --project src/DataReplicationSuite.Api
```

API varsayılan olarak `http://localhost:5109` adresinde başlar.

### API Uç Noktaları

| Metot | Uç Nokta | Açıklama |
|---|---|---|
| `POST` | `/api/syncjobs` | Yeni senkronizasyon işi oluştur ve zamanla |
| `GET` | `/api/syncjobs` | Tüm zamanlanmış senkronizasyon işlerini listele |
| `GET` | `/api/syncjobs/{id}` | Tek bir senkronizasyon işini getir |
| `PUT` | `/api/syncjobs/{id}` | Senkronizasyon işini güncelle |
| `DELETE` | `/api/syncjobs/{id}` | Senkronizasyon işini sil |
| `POST` | `/api/syncjobs/{id}/trigger` | İşi hemen çalıştır |
| `POST` | `/api/syncjobs/{id}/pause` | Çalışan işi duraklat |
| `POST` | `/api/syncjobs/{id}/resume` | Duraklatılmış işi devam ettir |

### SignalR Hub

Canlı senkronizasyon durum güncellemeleri almak için `http://localhost:5109/hubs/sync-notifications` adresine bağlanın.

### Swagger

`Development` ortamında Swagger UI `http://localhost:5109/swagger` adresinde kullanılabilir.

---

## WPF İstemcisini Çalıştırma

```bash
dotnet run --project src/DataReplicationSuite.Client
```

- Windows masaüstü uygulaması olarak çalışır.
- `Microsoft.Extensions.DependencyInjection` ile DI yapılandırılır; `MainWindow` hizmet konteynerinden çözülür.
- Üst bardaki **EN/TR** düğmesiyle çalışma zamanında dil değiştirilebilir.

---

## Proje Yapısı

```
DataReplicationSuite/
├── DataReplicationSuite.sln
├── .gitignore
│
├── src/
│   ├── DataReplicationSuite.Core/
│   │   ├── Enums/
│   │   │   ├── SyncDirection.cs
│   │   │   ├── DatabaseType.cs
│   │   │   ├── FileFormat.cs
│   │   │   ├── ConflictResolutionStrategy.cs
│   │   │   └── SyncJobStatus.cs
│   │   ├── Models/
│   │   │   ├── DataRecord.cs
│   │   │   ├── ColumnMapping.cs
│   │   │   ├── DataSource.cs
│   │   │   ├── SyncJob.cs
│   │   │   ├── SyncResult.cs
│   │   │   └── ConflictRecord.cs
│   │   ├── Interfaces/
│   │   │   ├── IDatabaseConnector.cs
│   │   │   ├── IFileHandler.cs
│   │   │   ├── ISyncEngine.cs
│   │   │   ├── ISyncJobScheduler.cs
│   │   │   ├── IConflictResolver.cs
│   │   │   └── IXsltTransformService.cs
│   │   ├── Exceptions/
│   │   └── Extensions/
│   │
│   ├── DataReplicationSuite.Infrastructure/
│   │   ├── Infrastructure.csproj
│   │   ├── Config/             (AppConfig, DataSourceConfig)
│   │   ├── DatabaseConnectors/ (MsSqlConnector, MySqlConnector, PostgreSqlConnector, DapperHelper)
│   │   ├── FileHandlers/       (ExcelHandler, XmlHandler)
│   │   └── Services/           (SyncEngine, SyncScheduler, ConflictResolver,
│   │                              XsltTransformService, DatabaseHealthCheck)
│   │
│   ├── DataReplicationSuite.Api/
│   │   ├── Program.cs
│   │   ├── appsettings.json
│   │   ├── Controllers/        (SyncJobsController, DataSourcesController,
│   │   │                          ConflictsController, ConnectionsController)
│   │   ├── DTOs/               (CreateSyncJobDto, UpdateSyncJobDto, SyncJobDto,
│   │   │                          ConflictRecordDto, DataSourceDto, ColumnMappingDto, …)
│   │   ├── Hubs/               (SyncNotificationHub.cs)
│   │   └── Models/
│   │
│   └── DataReplicationSuite.Client/
│       ├── App.xaml / App.xaml.cs
│       ├── MainWindow.xaml / MainWindow.cs
│       ├── Enums/              (Core enum'ları referans alınır)
│       ├── Models/             (SyncJobItemViewModel, ConflictItemViewModel,
│       │                        ConnectionStatusViewModel, BaseViewModel …)
│       ├── Services/           (IApiClient, ApiClient, ILocalizationService,
│       │                        LocalizationService, INotificationService …)
│       ├── ViewModels/         (MainViewModel, SyncJobsViewModel, DataSourcesViewModel,
│       │                        ConflictsViewModel, MonitorViewModel)
│       ├── Views/              (SyncJobsView, DataSourcesView, ConflictsView,
│       │                        MonitorView, ResourceDictionary, XAML denetimleri)
│       └── Resources/          (Strings.en.resx, Strings.tr.resx)
└── README_EN.md
```

---

## Temel Kavramlar

### SyncDirection

```csharp
public enum SyncDirection
{
    SourceToTarget,   // Tek yönlü: kaynak → hedef
    Bidirectional     // Çift yönlü: kaynak ↔ hedef
}
```

### ConflictResolutionStrategy

```csharp
public enum ConflictResolutionStrategy
{
    SourceWins,       // Hedefi her zaman kaynak verisiyle geçersiz kıl
    TargetWins,       // Mevcut hedef verisini koru
    LatestWins,       // En son zaman damgasına sahip kaydı sakla
    Manual            // Manuel inceleme için işaretle; otomatik çözüm yok
}
```

### SyncJobStatus

```csharp
public enum SyncJobStatus
{
    Idle,             // Şu anda çalışmıyor
    Running,          // Aktif olarak işleniyor
    Pending,          // Kuyrukta, başlama bekliyor
    Completed,        // Başarıyla tamamlandı
    Failed            // Hata ile sonlandı
}
```

---

## Katkıda Bulunma

1. Depoyu fork edin ve `main` dalından yeni bir özellik dalı açın.
2. Mevcut kod stiline uyun — örtük using'ler etkin, nullable tür referansları etkin.
3. **Core** katmanına hiçbir NuGet paketi eklemeyin; tüm harici bağımlılıklar Infrastructure, Api veya Client katmanlarına aittir.
4. `dotnet build DataReplicationSuite.sln` komutunu çalıştırın, hata ve uyarı olmadığından emin olduktan sonra bir Pull Request açın.

---

## Lisans

MIT Lisansı — detaylar için [LICENSE](LICENSE) dosyasına bakın.


---

## 🇬🇧 English

# DataReplicationSuite

**DataReplicationSuite** is a production-quality, fully bilingual (Turkish + English) .NET 8.0 solution for multi-source data synchronization across relational databases, spreadsheet, and document formats. It provides scheduled sync jobs, conflict resolution, real-time monitoring, and a WPF desktop client — all bilingual.

---

## Features

- **Multi-Database Support** — Synchronize data across MySQL, PostgreSQL, and Microsoft SQL Server using Dapper + EF Core.
- **File Source Support** — Read from/write to Excel (`.xlsx` via EPPlus) and XML files.
- **Scheduled Sync Jobs** — Define recurring synchronization jobs with cron expressions powered by **Quartz.NET 3.18.1**.
- **Conflict Detection & Resolution** — Detect row-level conflicts on primary key and apply configurable strategies: `SourceWins`, `TargetWins`, `LatestWins`, `Manual`.
- **XSLT Transformation** — Apply XSLT stylesheets to XML data before or after synchronization.
- **Real-Time Monitoring** — **SignalR** hub pushes live sync job events and status updates to connected clients.
- **WPF Desktop Client** — Modern, MVVM-based WPF application (CommunityToolkit.Mvvm 8.3.2) with full Turkish/English localization.
- **Bilingual UI** — All UI strings available in Turkish (`tr`) and English (`en`), switchable at runtime.
- **REST API** — ASP.NET Core Web API controllers exposing CRUD and lifecycle endpoints for sync jobs, data sources, and conflict resolution.

---

## Architecture

```
DataReplicationSuite/
├── DataReplicationSuite.Core          # Domain models, enums, interfaces (0 NuGet deps)
├── DataReplicationSuite.Infrastructure# EF Core DbContexts, Dapper connectors, services
├── DataReplicationSuite.Api           # ASP.NET Core Web API + SignalR hub
└── DataReplicationSuite.Client        # WPF MVVM desktop client
```

### Layer Responsibilities

| Layer | Responsibility |
|---|---|
| **Core** | POCO models, enums (`SyncDirection`, `DatabaseType`, `FileFormat`, `ConflictResolutionStrategy`, `SyncJobStatus`), and interface contracts. Zero NuGet dependencies. |
| **Infrastructure** | EF Core 8.0 DbContext, Dapper connectors (MySQL, PostgreSQL, MSSQL), EPPlus Excel handler, XML handler, `SyncEngine`, `SyncScheduler` (Quartz), `ConflictResolver`, `XsltTransformService`. |
| **Api** | REST controllers, SignalR `SyncNotificationHub`, DTOs, CORS, Swagger, Quartz hosted service registration. |
| **Client** | WPF `App`/`MainWindow`, ViewModels (`MainViewModel`, `SyncJobsViewModel`, `MonitorViewModel`, …), Views, localization strings (TR/EN), `LocExtension` markup extension. |

---

## Technology Stack

| Technology | Version | Purpose |
|---|---|---|
| .NET | 8.0 | Runtime / SDK |
| ASP.NET Core | 8.0 | REST API + SignalR |
| EF Core | 8.0.26 | ORM / change tracking |
| MySql.EntityFrameworkCore | 9.0.0 | MySQL EF Core provider |
| Npgsql.EntityFrameworkCore.PostgreSQL | 8.0.4 | PostgreSQL EF Core provider |
| Microsoft.EntityFrameworkCore.SqlServer | 8.0.26 | MSSQL EF Core provider |
| Dapper | 2.1.66 | Micro-ORM / raw query layer |
| EPPlus | 8.5.4 | Excel `.xlsx` read/write |
| System.Xml | 8.0 | XML read/write + XSLT |
| Quartz.NET | 3.18.1 | Scheduled job engine |
| SignalR | 1.2.10 | Real-time event push |
| CommunityToolkit.Mvvm | 8.3.2 | MVVM helpers for WPF |
| System.Text.Json | 9.0.0 | JSON serialization |

---

## Prerequisites

- [.NET 8.0 SDK](https://dotnet.microsoft.com/en-us/download/dotnet/8.0)
- A running instance of at least one supported database (MySQL 8+, PostgreSQL 14+, or SQL Server 2019+)

---

## Getting Started

### 1. Clone

```bash
git clone https://github.com/your-org/DataReplicationSuite.git
cd DataReplicationSuite
```

### 2. Restore & Build

```bash
dotnet restore
dotnet build DataReplicationSuite.sln
```

Expected result: **Build succeeded. 0 Warning(s). 0 Error(s).**

### 3. Configure Database Connections

Edit `src/DataReplicationSuite.Api/appsettings.json`:

```json
{
  "ConnectionStrings": {
    "Default": "Host=localhost;Database=DataReplicationSuite;Username=postgres;Password=postgres"
  },
  "Sync": {
    "MaxBatchSize": 300,
    "MaxRetryCount": 3
  },
  "Quartz": {
    "SchedulerName": "SyncJobScheduler",
    "ThreadPoolMaxCount": 10
  }
}
```

---

## Running the API

```bash
dotnet run --project src/DataReplicationSuite.Api
```

The API starts at `http://localhost:5109` (configurable in `appsettings.json`).

### API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/syncjobs` | Create and schedule a new sync job |
| `GET` | `/api/syncjobs` | List all scheduled sync jobs |
| `GET` | `/api/syncjobs/{id}` | Get a single sync job |
| `PUT` | `/api/syncjobs/{id}` | Update a sync job |
| `DELETE` | `/api/syncjobs/{id}` | Delete a sync job |
| `POST` | `/api/syncjobs/{id}/trigger` | Trigger immediate job execution |
| `POST` | `/api/syncjobs/{id}/pause` | Pause a running job |
| `POST` | `/api/syncjobs/{id}/resume` | Resume a paused job |

### SignalR Hub

Connect to `http://localhost:5109/hubs/sync-notifications` to receive real-time sync status events.

### Swagger

When running in `Development` environment, Swagger UI is available at `http://localhost:5109/swagger`.

---

## Running the WPF Client

```bash
dotnet run --project src/DataReplicationSuite.Client
```

- Runs as a WinExe Windows desktop application (requires Windows).
- Uses `Microsoft.Extensions.DependencyInjection` for DI; `MainWindow` is resolved from the service container.
- Switch language at runtime via the **EN/TR** button in the top bar.

---

## Project Structure

```
DataReplicationSuite/
├── DataReplicationSuite.sln
├── .gitignore
│
├── src/
│   ├── DataReplicationSuite.Core/
│   │   ├── Enums/
│   │   │   ├── SyncDirection.cs
│   │   │   ├── DatabaseType.cs
│   │   │   ├── FileFormat.cs
│   │   │   ├── ConflictResolutionStrategy.cs
│   │   │   └── SyncJobStatus.cs
│   │   ├── Models/
│   │   │   ├── DataRecord.cs
│   │   │   ├── ColumnMapping.cs
│   │   │   ├── DataSource.cs
│   │   │   ├── SyncJob.cs
│   │   │   ├── SyncResult.cs
│   │   │   └── ConflictRecord.cs
│   │   ├── Interfaces/
│   │   │   ├── IDatabaseConnector.cs
│   │   │   ├── IFileHandler.cs
│   │   │   ├── ISyncEngine.cs
│   │   │   ├── ISyncJobScheduler.cs
│   │   │   ├── IConflictResolver.cs
│   │   │   └── IXsltTransformService.cs
│   │   ├── Exceptions/
│   │   └── Extensions/
│   │
│   ├── DataReplicationSuite.Infrastructure/
│   │   ├── Infrastructure.csproj
│   │   ├── Config/             (AppConfig, DataSourceConfig)
│   │   ├── DatabaseConnectors/ (MsSqlConnector, MySqlConnector, PostgreSqlConnector, DapperHelper)
│   │   ├── FileHandlers/       (ExcelHandler, XmlHandler)
│   │   └── Services/           (SyncEngine, SyncScheduler, ConflictResolver,
│   │                              XsltTransformService, DatabaseHealthCheck)
│   │
│   ├── DataReplicationSuite.Api/
│   │   ├── Program.cs
│   │   ├── appsettings.json
│   │   ├── Controllers/        (SyncJobsController, DataSourcesController,
│   │   │                          ConflictsController, ConnectionsController)
│   │   ├── DTOs/               (CreateSyncJobDto, UpdateSyncJobDto, SyncJobDto,
│   │   │                          ConflictRecordDto, DataSourceDto, ColumnMappingDto, ...)
│   │   ├── Hubs/               (SyncNotificationHub.cs)
│   │   └── Models/
│   │
│   └── DataReplicationSuite.Client/
│       ├── App.xaml / App.xaml.cs
│       ├── MainWindow.xaml / .cs
│       ├── Enums/              (referenced from Core enums)
│       ├── Models/             (SyncJobItemViewModel, ConflictItemViewModel,
│       │                        ConnectionStatusViewModel, BaseViewModel …)
│       ├── Services/           (IApiClient, ApiClient, ILocalizationService,
│       │                        LocalizationService, INotificationService …)
│       ├── ViewModels/         (MainViewModel, SyncJobsViewModel, DataSourcesViewModel,
│       │                        ConflictsViewModel, MonitorViewModel)
│       ├── Views/              (SyncJobsView, DataSourcesView, ConflictsView,
│       │                        MonitorView, ResourceDictionary, XAML controls)
│       └── Resources/          (Strings.en.resx, Strings.tr.resx)
└── README_EN.md               ← this file
```

---

## Key Domain Concepts

### SyncDirection

```csharp
public enum SyncDirection
{
    SourceToTarget,   // unidirectional: source → target
    Bidirectional     // bidirectional: source ↔ target
}
```

### ConflictResolutionStrategy

```csharp
public enum ConflictResolutionStrategy
{
    SourceWins,       // always overwrite target with source data
    TargetWins,       // always keep existing target data
    LatestWins,       // keep the record with the most recent timestamp
    Manual            // flag for manual review; no automatic resolution
}
```

### SyncJobStatus

```csharp
public enum SyncJobStatus
{
    Idle,             // not currently running
    Running,          // actively processing
    Pending,          // queued, waiting to start
    Completed,        // finished successfully
    Failed            // terminated with errors
}
```

---

## Contributing

1. Fork the repository and create a feature branch from `main`.
2. Follow the existing code style — implicit usings enabled, nullable reference types enabled.
3. Do not add NuGet packages to the **Core** project; all external dependencies belong in Infrastructure, Api, or Client.
4. Run `dotnet build DataReplicationSuite.sln` and confirm zero warnings and zero errors before opening a PR.

---

## License

MIT License — see [LICENSE](LICENSE) for details.
