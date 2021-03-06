---
title: Xamarin.Forms 로컬 데이터베이스
description: Xamarin.Forms는 SQLite 데이터베이스 엔진을 사용하여 데이터베이스 기반 애플리케이션을 지원하기 때문에 공유 코드로 개체를 로드하고 저장할 수 있습니다. 이 문서에서는 Xamarin.Forms 애플리케이션이 SQLite.Net을 사용하여 데이터를 읽고 로컬 SQLite 데이터베이스에 데이터를 기록하는 방법을 설명합니다.
ms.prod: xamarin
ms.assetid: F687B24B-7DF0-4F8E-A21A-A9BB507480EB
ms.technology: xamarin-forms
author: profexorgeek
ms.author: jusjohns
ms.date: 12/05/2019
ms.openlocfilehash: e86d69bb9dc751f70d03a556f65c11efd0b34d10
ms.sourcegitcommit: db422e33438f1b5c55852e6942c3d1d75dc025c4
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/24/2020
ms.locfileid: "76725584"
---
# <a name="xamarinforms-local-databases"></a>Xamarin.Forms 로컬 데이터베이스

[![샘플 다운로드](~/media/shared/download.png) 샘플 다운로드](https://docs.microsoft.com/samples/xamarin/xamarin-forms-samples/todo)

SQLite 数据库引擎允许 Xamarin 应用程序加载和保存共享代码中的数据对象。 示例应用程序使用 SQLite 数据库表存储 todo 项。 本文介绍如何使用共享代码中的 SQLite.Net 在本地数据库中存储和检索信息。

[在 iOS 和 Android 上 ![Todolist 应用程序的屏幕截图](databases-images/todo-list-sml.png)](databases-images/todo-list.png#lightbox "IOS 和 Android 上的 Todolist 应用")

按照以下步骤将 SQLite.NET 集成到移动应用：

1. [安装 NuGet 包](#install-the-sqlite-nuget-package)。
1. [配置常量](#configure-app-constants)。
1. [创建数据库访问类](#create-a-database-access-class)。
1. [访问 Xamarin 中的数据](#access-data-in-xamarinforms)。
1. [高级配置](#advanced-configuration)。

## <a name="install-the-sqlite-nuget-package"></a>安装 SQLite NuGet 包

使用 NuGet 包管理器搜索**sqlite 网络 pcl** ，并将最新版本添加到共享代码项目。

이름이 유사한 NuGet 패키지가 여러 개 있습니다. 올바른 패키지에는 이러한 특성이 있습니다.

- **만든 사람:** Frank A. Krueger
- **ID：** sqlite 网络-pcl
- **NuGet 链接：** [sqlite 网络-pcl](https://www.nuget.org/packages/sqlite-net-pcl/)

> [!NOTE]
> 패키지 이름에도 불구하고 .NET 표준 프로젝트에서도 **sqlite-net-pcl** NuGet 패키지를 사용합니다.

## <a name="configure-app-constants"></a>配置应用常数

该示例项目包含一个**Constants.cs**文件，该文件提供了常见的配置数据：

```csharp
public static class Constants
{
    public const string DatabaseFilename = "TodoSQLite.db3";

    public const SQLite.SQLiteOpenFlags Flags =
        // open the database in read/write mode
        SQLite.SQLiteOpenFlags.ReadWrite |
        // create the database if it doesn't exist
        SQLite.SQLiteOpenFlags.Create |
        // enable multi-threaded database access
        SQLite.SQLiteOpenFlags.SharedCache;

    public static string DatabasePath
    {
        get
        {
            var basePath = Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData);
            return Path.Combine(basePath, DatabaseFilename);
        }
    }
}
```

常量文件指定用于初始化数据库连接的默认 `SQLiteOpenFlag` 枚举值。 `SQLiteOpenFlag` 枚举支持以下值：

- `Create`：连接将自动创建数据库文件（如果该文件不存在）。
- `FullMutex`：连接在序列化线程模式下打开。
- `NoMutex`：连接在多线程模式下打开。
- `PrivateCache`：连接将不会参与共享缓存（即使已启用）。
- `ReadWrite`：连接可以读取和写入数据。
- `SharedCache`：如果已启用共享缓存，则连接将参与共享缓存。
- `ProtectionComplete`：当设备被锁定时，文件已加密且不可访问。
- `ProtectionCompleteUnlessOpen`：在打开文件之前对其进行加密，但即使用户锁定设备，也可对其进行访问。
- `ProtectionCompleteUntilFirstUserAuthentication`：在用户已启动并解锁设备之前，对文件进行加密。
- `ProtectionNone`：数据库文件未加密。

根据数据库的使用方式，可能需要指定不同的标志。 有关 `SQLiteOpenFlags`的详细信息，请参阅 sqlite.org 上[的打开新的数据库连接](https://www.sqlite.org/c3ref/open.html)。

## <a name="create-a-database-access-class"></a>创建数据库访问类

数据库包装类从应用程序的其余部分对数据访问层进行抽象化。 此类集中了查询逻辑，并简化了数据库初始化的管理，从而更容易在应用程序增长时重构或扩展数据操作。 Todo 应用出于此目的定义了一个 `TodoItemDatabase` 类。

### <a name="lazy-initialization"></a>초기화 지연

`TodoItemDatabase` 使用 .NET `Lazy` 类在第一次访问数据库之前延迟数据库的初始化。 使用延迟初始化可防止数据库加载进程延迟应用程序启动。 有关详细信息，请参阅[懒惰&lt;t&gt; 类](xref:System.Lazy`1)。

```csharp
public class TodoItemDatabase
{
    static readonly Lazy<SQLiteAsyncConnection> lazyInitializer = new Lazy<SQLiteAsyncConnection>(() =>
    {
        return new SQLiteAsyncConnection(Constants.DatabasePath, Constants.Flags);
    });

    static SQLiteAsyncConnection Database => lazyInitializer.Value;
    static bool initialized = false;

    public TodoItemDatabase()
    {
        InitializeAsync().SafeFireAndForget(false);
    }

    async Task InitializeAsync()
    {
        if (!initialized)
        {
            if (!Database.TableMappings.Any(m => m.MappedType.Name == typeof(TodoItem).Name))
            {
                await Database.CreateTablesAsync(CreateFlags.None, typeof(TodoItem)).ConfigureAwait(false);
                initialized = true;
            }
        }
    }

    //...
}
```

数据库连接是一个静态字段，可确保在应用程序的生存期内使用单个数据库连接。 使用持久性静态连接比在单个应用会话期间多次打开和关闭连接提供了更好的性能。

`InitializeAsync` 方法负责检查是否存在用于存储 `TodoItem` 对象的表。 如果表不存在，此方法会自动创建该表。

### <a name="the-safefireandforget-extension-method"></a>SafeFireAndForget 扩展方法

实例化 `TodoItemDatabase` 类时，它必须初始化数据库连接，这是一个异步过程。 그러나

- 类构造函数不能是异步的。
- 未等待的异步方法不会引发异常。
- 使用 `Wait` 方法会阻止线程_并_吞并异常。

为了启动异步初始化，请避免阻止执行，并有机会捕获异常，示例应用程序使用名为 `SafeFireAndForget`的扩展方法。 `SafeFireAndForget` 扩展方法向 `Task` 类提供附加功能：

```csharp
public static class TaskExtensions
{
    // NOTE: Async void is intentional here. This provides a way
    // to call an async method from the constructor while
    // communicating intent to fire and forget, and allow
    // handling of exceptions
    public static async void SafeFireAndForget(this Task task,
        bool returnToCallingContext,
        Action<Exception> onException = null)
    {
        try
        {
            await task.ConfigureAwait(returnToCallingContext);
        }

        // if the provided action is not null, catch and
        // pass the thrown exception
        catch (Exception ex) when (onException != null)
        {
            onException(ex);
        }
    }
}
```

`SafeFireAndForget` 方法等待提供的 `Task` 对象的异步执行，并允许附加在引发异常时调用的 `Action`。

有关详细信息，请参阅[基于任务的异步模式（单击）](https://docs.microsoft.com/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap)。

### <a name="data-manipulation-methods"></a>数据操作方法

`TodoItemDatabase` 类包括用于四种类型的数据操作的方法：创建、读取、编辑和删除。 SQLite.NET 库提供了一个简单的对象关系映射（ORM），可用于存储和检索对象，而无需编写 SQL 语句。

```csharp
public static class TodoItemDatabase {

    // ...

    public Task<List<TodoItem>> GetItemsAsync()
    {
        return Database.Table<TodoItem>().ToListAsync();
    }

    public Task<List<TodoItem>> GetItemsNotDoneAsync()
    {
        // SQL queries are also possible
        return Database.QueryAsync<TodoItem>("SELECT * FROM [TodoItem] WHERE [Done] = 0");
    }

    public Task<TodoItem> GetItemAsync(int id)
    {
        return Database.Table<TodoItem>().Where(i => i.ID == id).FirstOrDefaultAsync();
    }

    public Task<int> SaveItemAsync(TodoItem item)
    {
        if (item.ID != 0)
        {
            return Database.UpdateAsync(item);
        }
        else
        {
            return Database.InsertAsync(item);
        }
    }

    public Task<int> DeleteItemAsync(TodoItem item)
    {
        return Database.DeleteAsync(item);
    }
}
```

## <a name="access-data-in-xamarinforms"></a>访问 Xamarin 中的数据

Xamarin `App` 类公开 `TodoItemDatabase` 类的实例：

```csharp
public static TodoItemDatabase Database
{
    get
    {
        if (database == null)
        {
            database = new TodoItemDatabase();
        }
        return database;
    }
}
```

此属性允许 Xamarin 组件在 `Database` 实例上调用数据检索和操作方法以响应用户交互。 예를 들면 다음과 같습니다.:

```csharp
var saveButton = new Button { Text = "Save" };
saveButton.Clicked += async (sender, e) =>
{
    var todoItem = (TodoItem)BindingContext;
    await App.Database.SaveItemAsync(todoItem);
    await Navigation.PopAsync();
};
```

## <a name="advanced-configuration"></a>고급 구성

SQLite 提供了一个功能强大的 API，该 API 的功能多于本文和示例应用中介绍的功能。 以下各节介绍了对于可伸缩性很重要的功能。

有关详细信息，请参阅 sqlite.org 上的[SQLite 文档](https://www.sqlite.org/docs.html)。

### <a name="write-ahead-logging"></a>预写日志记录

默认情况下，SQLite 使用传统的 rollback 日志。 未更改的数据库内容的副本将写入单独的回滚文件中，然后将这些更改直接写入数据库文件。 删除回滚日志时进行提交。

预写日志记录（WAL）首先将更改写入单独的 WAL 文件中。 在 WAL 模式下，COMMIT 是一个特殊记录，附加到 WAL 文件，这允许在单个 WAL 文件中发生多个事务。 在名为 "_检查点_" 的特殊操作中，WAL 文件将合并回数据库文件。

对于本地数据库，WAL 可能会更快，因为读取器和编写器彼此之间不会相互阻止，这允许进行读写操作。 但是，WAL 模式不允许更改_页面大小_、向数据库添加其他文件关联以及添加额外的_检查点_操作。

若要在 SQLite.NET 中启用 WAL，请对 `SQLiteAsyncConnection` 实例调用 `EnableWriteAheadLoggingAsync` 方法：

```csharp
await Database.EnableWriteAheadLoggingAsync();
```

有关详细信息，请参阅 sqlite.org 上的[SQLite 预写日志记录](https://www.sqlite.org/wal.html)。

### <a name="copying-a-database"></a>复制数据库

在以下几种情况下，可能需要复制 SQLite 数据库：

- 数据库附带了你的应用程序，但必须将其复制或移动到移动设备上的可写存储中。
- 你需要创建数据库的备份或副本。
- 需要对数据库文件进行版本、移动或重命名。

通常，移动、重命名或复制数据库文件与其他任何文件类型的过程相同，但有一些其他注意事项：

- 在尝试移动数据库文件之前，应关闭所有数据库连接。
- 如果使用[预写日志记录](#write-ahead-logging)，SQLite 将创建共享内存访问（. 具有 shm）文件和（写入日志）（wal）文件。 请确保也将任何更改应用于这些文件。

有关详细信息，请参阅[Xamarin 中的文件处理](~/xamarin-forms/data-cloud/data/files.md)。

## <a name="related-links"></a>관련 링크

- [Todo 示例应用程序](https://docs.microsoft.com/samples/xamarin/xamarin-forms-samples/todo)
- [SQLite.NET NuGet 包](https://www.nuget.org/packages/sqlite-net-pcl/)
- [SQLite 文档](https://www.sqlite.org/docs.html)
- [将 SQLite 用于 Android](~/android/data-cloud/data-access/using-sqlite-orm.md)
- [将 SQLite 与 iOS 配合使用](~/ios/data-cloud/data/using-sqlite-orm.md)
- [基于任务的异步模式（点击）](https://docs.microsoft.com/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap)
- [迟缓&lt;T&gt; 类](xref:System.Lazy`1)
