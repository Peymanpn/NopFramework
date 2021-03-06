# ASP.net Core Development Framework
###### Derived from NopCommerce

By fast growing of NopCommerce usage and numerous developers who are familiar with this e-commerce solution and based on the fact that the design and architecture of this platform is well-formed and very easy to understand and develop, I've derived this fundamental development framework from [V4.1 of NopCommerce](https://github.com/nopSolutions/nopCommerce).
This solution is a boilerplate for creating any .net Core web application.
I'll be adding some other capabilities and components to this primitive solution in future versions.


Feel free to ask any question: alirezakhosravi [at] live.com

### To create searchable entity follow this code:
Your entity must implement ISearchable interface in Nop.Core solution.
```
public partial class Country : BaseEntity, ILocalizedEntity, ISearchable
{
    public Country()
    {
       Route = new RouteModel()
       {
           RouteUrl = "your route url when you want show detail of entity",
           RouteName = "if you predefine route, enter route name",
           Parameters = new List<ParameterType> { ParameterType.Id, ParameterType.Description, ParameterType.Name}
       };
    }
    
    [NotMapped]
    public RouteModel Route { get; }
        ...
}
```
You should use one of RoutUrl and RouteName each time.
Parameter field must be used in route parameter, when you want to pass parameter to route.

```
Route = new RouteModel()
{
   RouteUrl = "/Country/Detail?name={0}&isocode={1}",
   Parameters = new List<ParameterType> { ParameterType.Name, ParameterType.Description}
};
```
On the code above the 'Name value' is replaced with {0} and the 'Description value' is replaced with {1}.

If you want to change some properties to default parameter type, you can use SearchableAttribute property and set UseFor parameter for this. In this case, ThreeLetterIsoCode is used for Description on search structure.
```
[Searchable(UseFor = ParameterType.Description)]
public string ThreeLetterIsoCode { get; set; }
```

If you want to ignore property in search, yet you want to use this property in search structure, you can set ignore parameter as true. 
```
[Searchable(UseFor = ParameterType.Id, Ignore = true)]
public string TwoLetterIsoCode { get; set; }
```

If you use SearchableAttribute property without setting parameter, this attribute is only used to query-string for search.
```
[Searchable]
public string ThreeLetterIsoCode { get; set; }
```

### To create listener to get notification, follow this code:
Your class must implement INotificationObserver interface in Nop.Services solution
```
using Nop.Services.Notifications;
....

 public class WebNotificationObserver : INotificationObserver
 {
    private readonly INotificationHandler _notificationHandler;
    private readonly IQueuedNotificationService _queuedNotificationService;
    
    public WebNotificationObserver(
        INotificationHandler notificationHandler, 
        IQueuedNotificationService queuedNotificationService
    )
    {
         _notificationHandler = notificationHandler;
         _queuedNotificationService = queuedNotificationService;
         
         _notificationHandler.Register(this);
    }
    
    public string Identifier => "WebNotificationObserver";
    
    public INotificationHandler Handler
    {
        get => _notificationHandler;
        set => throw new NotImplementedException();
    }
     
    public void Notify(QueuedNotification message)
    {
        if (!message.IsCheckMessage(this.Identifier))
        {
            ...
            message.AddObserver(this.Identifier);
            _queuedNotificationService.UpdateQueuedNotification(message);
        }
    } 
 }
```

Identifier is very important beacause when your listener observed the message,it puts it's own identifier on the message so if ,later, the observer got this message, do not process it.

### Enable Ignite Cache
To enable ignite cache, edit ``appsettings.json`` in root directory of Nop.Web.
```
    "IgniteCachingConnectionString": ["Address1", "Address2", "Address3", ...],
```
For enable persisted change `` "PersistDataProtectionKeysToIgnite" `` to `` true``.
```
    "PersistDataProtectionKeysToIgnite": true,
```

### Set Ignite to default caching
For Enable Ignite to default cachng set ``IgniteCachingEnabled`` attribute in ``appsettings.json`` file in root directory of Nop.Web to ``true``.
```
    "IgniteCachingEnabled": true,
    "IgniteCachingConnectionString": ["Address1", "Address2", "Address3", ...],
```

### Temporal Table
For enabling temporal feauture, your entity must be inherited from ``ITemporal`` interface.
```
    public class User : BaseEntity, ITemporal
    {
        ...
    }
```

If you want to retrieve data at specific time, you must add `` using Nop.Data.Extensions; ``.
```
    public class UserServices : IUserServices
    {
        private readonly IRepository<User> _user;
        
        public UserServices(IRepository<User> user)
        {
            this._user = user;
        }
        
        public List<User> GetUsers()
        {
            return _user.TemporalTable(DateTime.Now).ToList(); // or add specific time
        }
        
        public User GetUser(int id)
        {
            return _user.GetTemporalById(id, DateTime.Now).ToList(); // or add specific time
        }
    }
```

### Enable Change Traking for entity
For Enable Temporal Table, You must set value of ``EnableChangeTracking`` on root of Nop.Web is true, and your entity must implemented ``IChangeTraking`` interface.
```
 public partial class User : BaseEntity, IChangeTracking
 {
    ...
    [NotMapped]
    public string SYS_CHANGE_VERSION { get; set; }
    [NotMapped]
    public string SYS_CHANGE_CREATION_VERSION { get; set; }
    [NotMapped]
    public string SYS_CHANGE_OPERATION { get; set; }
    [NotMapped]
    public string SYS_CHANGE_COLUMNS { get; set; }
    [NotMapped]
    public string SYS_CHANGE_CONTEXT { get; set; }
 }
```
If you want to retrieve history of data, you must add `` using Nop.Data.Extensions; ``.
```
   public class UserServices : IUserServices
    {
        private readonly IRepository<User> _user;
        
        public UserServices(IRepository<User> user)
        {
            this._user = user;
        }
        
        public List<User> GetHistory()
        {
            return _user.ChangeTraking().ToList();
        }
    } 
```
By default, remove history after 10 days, if you want changing this time, you must changed the ``ChangeRetention`` attribute in ``appsettings.json`` on root directory of ``Nop.Web``.
