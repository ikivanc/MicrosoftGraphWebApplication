# Microsoft Graph 'a başlangıç
Bu lab örneğinde , Microsoft Graph ile ilk projenizi geliştireceksiniz.

## Gereksinimler
1. Bu lab'ı test edebilmeniz için Office 365 tenantına ihtiyacınız vardır. Eğer bir hesabınız yoksa [**Office 365 için geliştirme ortamı oluşturma linkini**](https://github.com/OfficeDev/TrainingContent/blob/master/O3651/O3651-7%20Setting%20up%20your%20Developer%20environment%20in%20Office%20365/Lab.md) kontrol edebilirsiniz.

## Adım 1:  Bir MVC Web Uygulaması Oluşturma ve Konfigüre etme
Bu adımda, bir MVC web uygulaması oluşturup onu Microsoft Graph ile bağlamayı göreceksiniz.

1. Visual Studio'dan, **File/New/Project** seçin.
1. **New Project** 'i seçin
  1. **Templates/Visual C#/Web** 'i seçin.
  1. **ASP.NET Web Application** 'ı seçin.

    ![](Screenshots/01.png)
    > **Not:** Projenizi olutşururken bu lab içerisindeki isimlerin aynısını vermeye dikkat edin. The Visual Studio Project adı projenin kodu içerisinde namespace olacaktır. Bu adımlardaki kodların tamamı namespace ile ilişkili olacağı için kodunuzun çalışmasına da etkili olacaktır. Eğer farklı bir proje adı verirseniz namespace'ler farklı olacağından buradaki örnek kodlar çalışmayacaktır.
  1. **OK** Tıklayın.
  
1.  **New ASP.NET Project** dialogu içerisinden
  1. **MVC** 'yi seçin.
  2. **Change Authentication** 'ı seçin.
  3. **Work And School Accounts** 'ı seçin.
  4. **Cloud - Single Organization** 'ı seçin
  5. **Domain** adınızı girin
  6. Directory Access Permissions altındaki **Read directory data** seçeneğini seçin.  
  7. **OK** 'i tıklayın.
  8. **Host in the cloud** seçeniğini kaldırın.
  9. **OK** 'İ tıklayın.

    ![](Screenshots/03.png)

    ![](Screenshots/02.png)

1.  SSL'i default olarak kullanmak için web projesini aşağıdaki şekilde update edin :
  1. **Solution Explorer** bölümündeki araçlar penceresinde, projeyi seçin ve **Properties** araç ekranına bakın. 
  1. **SSL Enabled** özelliğini **TRUE** olarak değiştirin.
  1. **SSL URL** değerini bir sonraki adımlarda kullanmak için kopyalayın.
  1. Değişikliklerinizi kaydedin.

    ![](Screenshots/SslEnabled.png)
    
    > Bu aşama önemli çünkü Azure AD üzerinde bir application oluşturduğunuzda ona HTTPS kullarak dönüş yapmanız gerekmektedir. Eğer bu değişiliği bu aşamada gerçekleştirmediyseniz, Visual Studio wizard kullanarak yapabilir ve sizin adınıza gerekli konfigürasyonları gerçekleştirebilir.
    
1. Debug ederken her zaman anasayfanıza gitmesini sağlamak için projenizi konfigure edebilirsiniz:
  1. **Solution Explorer** araç penceresi içerisinden **Properties** 'i seçin.
  1. Sol taraftaki **Web** 'i seçin.
  1. **Start Action** bölümüne gidin.
  1. **Start URL** olan radio buttonuna tıklayın ve daha önceki adımlarda kopyalamış olduğunuz web projenize ait olan SSL URL adresini yapıştırın.
  
1. Şu an ki noktada, uygulamanızın Authentication akışını test edebilirsiniz.
  1. Visual Studio içerisinde, **F5** 'e basın. Browser otomatik olarak çalışarak HTTPS başlangıç sayfanıza gidecektir.

   > **Not:** Eğer "ASP.NET could not connect to the SQL database" hatası alırsanız lütfen [SQL Server Database Connection Error Resolution document](https://github.com/OfficeDev/TrainingContent/blob/master/SQL-DB-Connection-Error-Resolution.md) dökümanını inceleyerek çözüm getirebilirsiniz.. 

  1. Sign-in olabilmek için, sağ üst köşedeki **Sign In** işaretini tıklayın.
  1. **Organizational Account** ile O365 kurum hesabınızı kullanarak login olabilirsiniz.
  1. Başarılı bir login gerçekleştirebilmek için, ilk defa sign-in olmaya çalışırken, Azure AD  sizlere tüm login sayfalarında ortak olarak görebileceğiniz aşağıdaki ekranı görebileceksiniz:
    
    ![](Screenshots/ConsentDialog.png)

  1. Office 365'deki bilgilerinize erişimi isteyen bu yetki diyalogunda **Accept** buttonunu tıklayarak gerekli yetki izinlerini kabul edebilirsiniz.
  1. Daha sonrasında web uygulamanıza geri yönlendirileceksiniz. Dikkat etmeniz gereken nokta ise sağ üst köşede e-mail adresiniz ve **Sign Out** linkinin belirdiğini göreceksiniz.

Tebrikler... şu ana kadar başarılı bir şekilde Azure AD ve OpenID Connect ve OWIN kullanarak login olmayı başardınız!



## Adım 2:Azure AD ve OWIN'i kullanabilmek için, Web Uygulamanızı konfigure edin.
Bu adımda bir önceki adımda Azure AD & OpenID Connect ile bağladığımız ASP.NET MVC web uygulamasını kullanıcılar ve uygulama authentication'ları konusunda bir sonraki aşamaya geçireceğiz. bunu OWIN framework'ü kullanarak gerçekleştireceğiz. Authenticate olduktan sonra, Azure AD'den dönen access token aracılığı ile the Microsoft Graph'a erişeceğiz.


1. Gerekli Uygulama Yetkilendirmelerinin Verilmesi 

  1. Tarayıcınız üzerinde [Azure Management Portal](https://manage.windowsazure.com) 'e açarak Office 365 **Organizasyon Hesabınızla** login olun.
  
	> **Not:** Eğer Azure Management Portal bir pop-up mesajı çıkarara sizi yeni portala yönlendirmek isterse bunu iptal edin. Örneğimiz eski portal üzerinden devam edecektir. 
	 
  1. Soldaki menüden **Active Directory** seçeneğini seçin.
  1. Office 356 hesabınızın bulunduğu directory'i seçin. 
  1. Üstteki başlık/ayarlar barındaki sekme menülerinden **Application** 'ı seçin.
  1. Bu lab örneği için oluşturduğunuz uygulamayı seçin.
  1. **Configure** sekmesini açın.
  1. Sayfa altına inerek **permissions to other applications** seçeneğini seçin. 
  1. **Add Application** butonuna tıklayın.
  1. **Permissions to other applications** dialog ekranında, **Microsoft Graph** seçeneğinin yanında bulunan **ARTI** ikonuna tıklayarak ekleyin
  1. Sağ alt bölümde bulunan **CHECK** ikonunu tıklayın.
  1. Yeni **Microsoft Graph** uygulama yetki girişi için , **Delegated Permissions** bölümündeki dropdown menüsünden aşağıdaki yetkileri seçin:
    * **Read user calendars**    
  1. Sayfanın altında yer alan **Save** butonuna tıklayın.

     ![](Screenshots/AzurePermission.png)
1. Yeni bir helper class ekleyerek `web.config` haricinde authentication için gerekli olan konfigürasyonları düzenleyelim:

  1. Projeyi sağ tık ile seçerek  **Add/New Folder** ile yeni bir klasör ekleiyn ve klasörün ismini **Utils** olarak değiştirin. 
  1. [Lab Dosyaları](https://github.com/OfficeDev/TrainingContent/tree/master/O3651/O3651-5%20Getting%20started%20with%20Office%20365%20APIs/Lab%20Files) klasörü içerisinde yer alan  [`SettingsHelper.cs`](https://github.com/OfficeDev/TrainingContent/tree/master/O3651/O3651-5%20Getting%20started%20with%20Office%20365%20APIs/Lab%20Files) dosyasını bulun. Buradaki [`SettingsHelper.cs`](https://github.com/OfficeDev/TrainingContent/tree/master/O3651/O3651-5%20Getting%20started%20with%20Office%20365%20APIs/Lab%20Files) dosyayı **Utils** klasörü içerisine alın.
      
1. **_Layout** dosyasınız **Calendar** linkini eklemek için düzenleyelim:
    1. **Views/Shared** klasörü içerisinde yer alan **_Layout.cshtml** dosyasını bulup aşağıdaki şekilde güncelleyelim..
      1. Sayfann en üst kısmını aşağıdaki şekilde update edelim... kod güncellemesi öncesi  aşağıdaki şekilde görünecektir:
      
        ````asp
        <div class="navbar-collapse collapse">
          <ul class="nav navbar-nav">
            <li>@Html.ActionLink("Home", "Index", "Home")</li>
            <li>@Html.ActionLink("About", "About", "Home")</li>
            <li>@Html.ActionLink("Contact", "Contact", "Home")</li>
          </ul>
        </div>
        ````

      1. Sayfa linklerini (**Calendar** linki aşağıdaki gibi ekleyerek) aşağıdaki gibi ekleyerek, yeni oluşturulan login kontrolüne de referans vererek aşağıdaki gibi update edelim:

        ````asp
        <div class="navbar-collapse collapse">
          <ul class="nav navbar-nav">
            <li>@Html.ActionLink("Home", "Index", "Home")</li>
            <li>@Html.ActionLink("About", "About", "Home")</li>
            <li>@Html.ActionLink("Contact", "Contact", "Home")</li>
            <li>@Html.ActionLink("Calendar", "Index", "Calendar")</li>
          </ul>
          @Html.Partial("_LoginPartial")
        </div>
        ````

        > **Calendar** linki henüz çalışmayacak... bir sonraki adımda bunun çalışması için gerkeli işlemleri gerçekleştireceğiz.

## Adım 3: Microsoft Graph ve SDK'nın kullanılması
Bu adımda Microsoft Graph ve SDK kullanımını gerçekleştirmek için controller ve view'lar ekleyerek devam edeceğiz. 

1. Authentication işleminin tamamlanmasından sonra, takviminizden bilgileri çekebilmesi için aşağıdaki gibi yeni bir controller ekleyeceğiz:
  1. **Models** klasörüne sağ tıklayarak **Add/Class** 'ı seçelim.
    1. Açılan **Add Class** dialogunda, Class ismi olarak **MyEvent** adını vererek **Add** 'e tıklayalım.
    1. Implement the new class **MyEvent** using the following class definition.
    
    ````c#
    public class MyEvent {
           
        [DisplayName("Subject")]
        public string Subject { get; set; }

        [DisplayName("Start Time")]
        [DisplayFormat(DataFormatString = "{0:MM/dd/yyyy}", ApplyFormatInEditMode = true)]
        public DateTimeOffset? Start { get; set; }

        [DisplayName("End Time")]
        [DisplayFormat(DataFormatString = "{0:MM/dd/yyyy}", ApplyFormatInEditMode = true)]
        public DateTimeOffset? End { get; set; }
    }
    ````
  1. **Controllers** klasörüne sağ tıklayarak **Add/Controller** 'u seçelim.
    1. **Add Scaffold** dialog penceresinde  **MVC 5 Controller - Empty** seçeneğini seçerek **Add** butonunua tıklayarak ekleyelim.
    1. **Add Controller** dialog penceresinde, controller'a **CalendarController**  adını verek **Add** butonunua tıklayarak ekleyelim.
  1.  **CalendarController.cs** dosyası içerisine aşağıdaki `using` statementlarını var olan `using` statementlarından sonra ekleyelim:

    ````c#
	using System.ComponentModel;
	using System.ComponentModel.DataAnnotations;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    using System.Security.Claims;
    using System.Threading.Tasks;
    using Exercise2.Utils;
    using Exercise2.Models;
    using System.Net.Http;
    using System.Net.Http.Headers;
    using Newtonsoft.Json.Linq;    
    ````

  1. Buradaki içeriği sadece Authenticate olmuş kullanıcılara göstermek için aşağıda gibi `[Authorize]` attribute'nu controller tanımından önce koyalım:

    ````c#
    namespace Exercise2.Controllers {
      [Authorize]
      public class CalendarController : Controller {}
    }
    ````

  1. `GetGraphAccessTokenAsync`  adında yeni bir metod oluşturarak, Graph API Authentication'ından gerekli erişim token'ının alınmasını sağlayalım:

    ````c#
    public async Task<string> GetGraphAccessTokenAsync()
    {
        var signInUserId = ClaimsPrincipal.Current.FindFirst(ClaimTypes.NameIdentifier).Value;
        var userObjectId = ClaimsPrincipal.Current.FindFirst(SettingsHelper.ClaimTypeObjectIdentifier).Value;

        var clientCredential = new ClientCredential(SettingsHelper.ClientId, SettingsHelper.ClientSecret);
        var userIdentifier = new UserIdentifier(userObjectId, UserIdentifierType.UniqueId);

        // create auth context
        AuthenticationContext authContext = new AuthenticationContext(SettingsHelper.AzureAdAuthority, new ADALTokenCache(signInUserId));
        var result = await authContext.AcquireTokenSilentAsync(SettingsHelper.AzureAdGraphResourceURL, clientCredential, userIdentifier);

        return result.AccessToken;
    }
    ````
  1.`Index()` metodunu düzenleyerek asenkron çalışabilmesi için başına `async` anahtar kelimesini ekleyelim ve dönüş tipini de düzenleyelim:

    ````c#
    public async Task<ActionResult> Index() {}
    ````
  1. `Index()` methodu içerisinde, `HttpClient` kullanarak Graph Rest API 'dan kullanıcının takviminde yer alan ilk 20 etkinliği çekelim:

    ````c#
    var eventsResults = new List<MyEvent>();
    var accessToken = await GetGraphAccessTokenAsync();
    var restURL = string.Format("{0}me/events?$top=20&$skip=0", SettingsHelper.GraphResourceUrl);

    try
    {
        using (HttpClient client = new HttpClient())
        {
            var accept = "application/json";

            client.DefaultRequestHeaders.Add("Accept", accept);
            client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);

            using (var response = await client.GetAsync(restURL))
            {
                if (response.IsSuccessStatusCode)
                {
                    var jsonresult = JObject.Parse(await response.Content.ReadAsStringAsync());

                    foreach (var item in jsonresult["value"])
                    {
                        eventsResults.Add(new MyEvent
                        {
                            Subject = !string.IsNullOrEmpty(item["subject"].ToString()) ? item["subject"].ToString() : string.Empty,
                            Start = !string.IsNullOrEmpty(item["start"]["dateTime"].ToString()) ? DateTime.Parse(item["start"]["dateTime"].ToString()) : new DateTime(),
                            End = !string.IsNullOrEmpty(item["end"]["dateTime"].ToString()) ? DateTime.Parse(item["end"]["dateTime"].ToString()) : new DateTime()

                        });
                    }
                }                
            }
        }
    }
    catch (Exception el)
    {
        el.ToString();
    }

    ViewBag.Events = eventsResults.OrderBy(c => c.Start);
    ````
  
  `Index()` methodunun son satırında, controller için varsayılan view 'ı döndüreceği için onu olduğu gibi bıraklım.

1. Değişiklikleri kaydedelim.
1. Sonunda view arayünüzü update derek sonuçları görebiliriz.
  1.  `CalendarController` class'ında, `Index()` methodunun  sonunda yer alan `View()` 'a sağ tıklayarak **Add View** diyelim.
  1. **Add View** dialog penceresinde, aşağıdaki değerleri girelim:
    1. View Name: **Index**.
    1. Template: **Empty (without model)**.

    > Diğer değerleri boş ve seçilmemiş olarak bırakalım.    
  1. **Add** butonuna tıklayalım.   
  1. **Views/Calendar/Index.cshtml** dosyasınıda bulunan tüm kodu silerek aşağıdaki kod ile değiştirelim:

    ````html
    @{
      ViewBag.Title = "Home Page";
    }
    <div>
      <table>
        <thead>
          <tr>
            <th>Subject</th>
            <th>Start</th>
            <th>End</th>
          </tr>
        </thead>
        <tbody>
          @foreach (var o365Event in ViewBag.Events) {
            <tr>
              <td>@o365Event.Subject</td>
              <td>@o365Event.Start</td>
              <td>@o365Event.End</td>
            </tr>
          }
        </tbody>
      </table>
    </div>
    ````

  1. Değişikliklerimizi kaydedelim.
1. **F5** 'e basarak uygulamamızı test edelim.

  > **Not:** Eğer "ASP.NET could not connect to the SQL database" hatası alırsanız lütfen [SQL Server Database Connection Error Resolution document](https://github.com/OfficeDev/TrainingContent/blob/master/SQL-DB-Connection-Error-Resolution.md) dökümanını inceleyerek çözüm getirebilirsiniz.. 

  1. Şimdi tekrar login olmanızı isteyecek (Eğer logged-in olarak gelmediyse). Eğer hemen login olmadıysanız hemen sayfanın sağ üst köşesindeki **Sign in** seçeneğini tıklayın.
  2. Login ekranı gelince, Office365 **Organizasyon Hesabınızı** kullanarak log-in olun.
  3. Eğer sorarsa, uygulamanız için gerekli tüm yetkileri verebilirsiniz.
  4. Uygulamada AnaSayfa'da üst menüde yer alan **Calendar** linkine tıklayın.
  5. Takviminizde yer alan güncel etkinliklerin yerlerinde olduğundan emin olun.
   

** Tebrikler! İlk Microsoft Graph uygulamanızı geliştirdiniz.**